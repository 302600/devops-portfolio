# Jenkins 本地 CI 流水线搭建实录（Gitea + Jenkins + Docker）

> 记录人：陈世豪　｜　记录日期：2026-09-13
> 用途：补齐项目一（DevOps 交付平台）中"Jenkins 流水线"环节，记录全过程与踩坑，供面试讲解。

---

## 一、项目背景

项目一（K8s + Harbor + Gitea + Jenkins + ArgoCD）的云服务器是**按量付费**的，用完已经释放，当时 Jenkins 只完成了部署，**没有真正跑通流水线**。本次在**本地 VMware 虚拟机**里用 Docker 重建 CI 环节，目标是跑通完整闭环：

```
提交代码 → Gitea 仓库 → Jenkins 自动发现变更 → docker build 打镜像 → docker run 部署 → 页面自动更新
```

一句话：**改完代码 push 之后全程零人工干预，页面自己更新**，这就是 CI 要解决的事。

## 二、环境信息

| 项目 | 内容 |
| --- | --- |
| 宿主机 | 本地 VMware 虚拟机（Rocky Linux，内网 IP 192.168.121.101） |
| 内存 | 可用约 2.7 GiB |
| Docker | 28.5.1（社区版） |
| Gitea | 容器运行，端口 **3000**，数据挂载 `/opt/gitea` |
| Jenkins | 容器运行（LTS），端口 **8081**（8080 被已有 nginx 占用）+ 50000 agent 端口，数据挂载 `/opt/jenkins/data` |
| demo 应用 | 静态页面，部署端口 **8090** |

**Jenkins 容器启动命令的关键参数说明**：

```bash
docker run -d --name jenkins \
  -u root \
  -p 8081:8080 -p 50000:50000 \
  -v /opt/jenkins/data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(which docker):/usr/bin/docker \
  --restart always \
  jenkins/jenkins:lts
```

- `-u root`：容器内用 root，否则没权限操作挂载进来的 docker。
- 挂载 `docker.sock` + docker 客户端：让 Jenkins 容器里的流水线命令实际调用**宿主机 Docker** 来构建和部署（Docker-out-of-Docker 方案），不用在容器里再装一套 Docker。

## 三、搭建步骤

1. **起 Gitea 容器**，浏览器访问 `:3000` 初始化（数据库用默认 SQLite3），注册管理员账号，建仓库 `demo-app`，推入示例代码（index.html + Dockerfile + Jenkinsfile）。
2. **起 Jenkins 容器**，`docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword` 拿初始密码，访问 `:8081` 完成初始化，安装必需插件：Git / Pipeline / Pipeline: Stage View / Credentials Binding。
3. **建 Pipeline 任务**，选 "Pipeline script from SCM"：仓库地址填 Gitea 的 `demo-app.git`，分支 `*/main`，脚本路径 `Jenkinsfile`。
4. **手动 Build Now 验证** Jenkinsfile 三阶段能跑通。
5. **配置自动触发**：任务配置 → Triggers → 勾选"轮询 SCM"，日程表 `* * * * *`（每分钟检测一次仓库变化）。
6. **闭环验证**：在 Gitea 网页上直接把 index.html 从 v1 改成 v2 提交，**不点 Jenkins 任何按钮**，约 1-2 分钟后 Jenkins 自动触发构建（Git 轮询日志显示 `Changes found`），构建完成后刷新 `:8090`，页面自动变成 v2。

## 四、Jenkinsfile（流水线即代码）

```groovy
pipeline {
    agent any
    stages {
        stage('拉取代码') {
            steps {
                checkout scm
            }
        }
        stage('构建镜像') {
            steps {
                sh 'docker build -t demo-app:${BUILD_NUMBER} .'
            }
        }
        stage('部署') {
            steps {
                sh '''
                    docker rm -f demo-app || true
                    docker run -d --name demo-app -p 8090:80 demo-app:${BUILD_NUMBER}
                '''
            }
        }
    }
}
```

- `agent any`：在任意可用节点（这里就是 master）上执行。
- `${BUILD_NUMBER}`：Jenkins 内置变量，用构建号做镜像 tag，每次构建的镜像可追溯、不冲突。
- `docker rm -f ... || true`：先删旧容器再部署，`|| true` 保证第一次构建（容器还不存在）时不报错。

## 五、踩坑记录（重点，面试排障素材）

| 坑 | 现象 | 排查与解决 |
| --- | --- | --- |
| 插件源网络不稳 | 初始化时推荐插件大面积安装失败 | 官方源在国外、网络丢包严重（ping 百度丢 27%）。策略：恢复官方源 + 逐个插件反复重试"磨"下来 |
| 清华镜像按 IP 封禁 | 换清华 update-center 源后 curl 返回 403 | 用 GET 请求测试（`curl -I` 的 HEAD 请求很多镜像站会误拦，需用 `-o /dev/null -w %{http_code}` 验证）；确认是封 IP 后放弃镜像站，回官方源 |
| 分支名不匹配 | 构建报 `couldn't find remote ref refs/heads/master` | Gitea 新版本默认分支是 **main**，任务里 Branches to build 要从 `*/master` 改成 `*/main` |
| 构建永远排队 | 一直 "Waiting for next available executor" | 首页"构建执行状态"发现 master 节点**未在线**；查配置 executors=2 无异常，最终定位是**磁盘用了 96%（剩 753M）**，触发 Jenkins 磁盘监控（默认 1G 阈值）自动把节点下线。`docker system prune -a -f` 清出 4.6G 后节点自动恢复 |
| 轮询不触发 | Git 轮询日志显示"还没有开始轮询" | 日程表填的 `H/1 * * * *` 被解析成**每小时一次**（页面提示下次运行在 1 小时后），改成显式的 `* * * * *` 后每分钟轮询生效 |

**排障方法论沉淀**：

- 网络类问题先分层测：宿主机 curl → 容器内 curl，区分是 VM 出口问题还是容器 DNS 问题。
- Jenkins 构建排队不要干等，先看"构建执行状态"里节点是否在线，再看 executors 数量。
- 页面提示（如 cron 日程表下方的"上次/下次运行时间"）是最好的验证工具，改完配置先看提示再保存。

## 六、验证证据

| 截图 | 说明 |
| --- | --- |
| Git 轮询日志 | 显示 `Changes found`，构建 #6 由轮询自动触发（非手动） |
| 构建历史 | 手动构建 #1-#5 + 自动触发 #6，绿色成功 |
| 应用页面 | `http://192.168.121.101:8090` 显示 "Demo App v2 - Jenkins 流水线部署成功"，证明提交后自动部署生效 |

## 七、面试讲解要点

- **为什么用轮询 SCM 而不是 Webhook**：Gitea Webhook 需要 Jenkins 装 Generic Webhook Trigger 插件并互配，轮询是 Jenkins 自带能力、零插件依赖，学习环境最省事；生产上 Webhook 实时性更好（秒级触发），轮询有分钟级延迟，两者都懂、按场景选。
- **流水线三阶段的对应关系**：拉代码（checkout scm）→ 构建（docker build）→ 部署（docker run），项目一里的完整链路是 Jenkins 构建后推 Harbor、再由 ArgoCD 部署到 K8s，本次是最小闭环，原理相同。
- **Jenkinsfile 放仓库里的意义**：流水线即代码（Pipeline as Code），构建流程版本化，换 Jenkins 实例也能一键恢复，这也是 GitOps 思想的一部分。
