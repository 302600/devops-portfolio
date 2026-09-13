# Jenkins 与 CI/CD 原理学习笔记

> 记录人：陈世豪　｜　记录日期：2026-09-13
> 用途：配套《07-Jenkins本地CI流水线搭建实录》，把实操背后的原理吃透，应对面试追问。
> 每节按"是什么 → 为什么 → 面试怎么答"组织。

---

## 一、CI / CD 到底是什么

### 1. 三个概念分清（面试高频）

| 概念 | 全称 | 干什么 | 关键词 |
| --- | --- | --- | --- |
| **CI 持续集成** | Continuous Integration | 开发人员频繁把代码合并到主干，每次合并**自动构建 + 自动测试**，尽早发现集成问题 | 提交即验证 |
| **CD 持续交付** | Continuous Delivery | 在 CI 之上，代码**随时可以发布**，但上线需要人工点一下按钮 | 随时可发，人工放行 |
| **CD 持续部署** | Continuous Deployment | 持续交付的更进一步，**连按钮都不用点**，通过验证后自动上线 | 全自动上线 |

一句话区分：**CI 解决"代码合进来能不能用"，持续交付解决"想发随时能发"，持续部署解决"发了不用管"**。我们本次做的（push 后自动构建镜像、自动替换容器上线）属于**持续部署**，因为没有人工审批环节。

### 2. 为什么需要 CI/CD（解决了什么痛点）

- 没有 CI 之前：开发各写各的，几周才合一次代码，合并时冲突爆炸、bug 难定位（"集成地狱"）。
- 没有 CD 之前：上线靠运维手动拉代码、编译、传包、重启，步骤多、易出错、半夜加班发版。
- 有了 CI/CD：**每次提交都自动验证，每次发版都走同一条流水线**，出错概率和人力成本都大幅下降。

### 3. 一条标准流水线的阶段

```
代码提交 → 拉取代码 → 编译/构建 → 单元测试 → 打包/打镜像 → 推镜像仓库 → 部署到环境 → （可选）自动化验收测试
```

我们 demo 里裁剪成了三段：拉代码 → docker build → docker run。项目一（云端完整版）里则是：Jenkins 构建 → 推 Harbor → ArgoCD 同步到 K8s。**阶段可多可少，思想不变：每一步自动衔接，产物可追溯到某次提交。**

---

## 二、Jenkins 核心概念

### 1. Jenkins 是什么

开源的自动化服务器（Java 写的），本质是一个**任务调度平台**：你告诉它"在什么时机、执行什么步骤"，它负责调度执行、记录日志、展示结果。它本身不内置"构建"能力，几乎所有能力都靠**插件**。

### 2. 架构：Master / Agent（控制器 / 执行节点）

```
┌─────────────────────┐
│   Master（控制器）    │  Web 界面、任务配置、调度分发、日志收集
│   （built-in node）  │
└─────────┬───────────┘
          │ 分发任务
   ┌──────┼──────┐
   ▼      ▼      ▼
 Agent1 Agent2 Agent3   真正干活的节点：编译、测试、打包
```

- **Master**：调度中心，负责界面、配置、把任务分给 Agent。**生产环境不建议在 master 上跑构建**（构建任务吃资源，会把 Web 界面拖垮；且有安全风险——构建脚本可以直接读到 master 上的密钥文件）。
- **Agent**：执行节点，可以是另一台机器、一个虚拟机、一个 K8s Pod。Jenkins 通过 SSH 或 JNLP（agent 端口，就是我们映射的 50000）连接 agent。
- **Executor（执行者）**：节点上的"工人数量"，1 个 executor 同时只能跑 1 个构建。executors=2 意味着这个节点最多同时跑 2 个任务，多出来的排队——这就是当时 "Waiting for next available executor" 的含义。
- 我们学习环境只有 master 一个节点，构建就跑在 master 上（`agent any` = 任意可用节点，此时只有 master）。

> 面试追问"Jenkins 怎么扩容"：构建多了 master 扛不住，就加 agent 节点横向扩展；K8s 环境下可以用 Kubernetes 插件，每个构建动态起一个 Pod 当 agent，跑完即销毁。

### 3. Workspace（工作目录）

每次构建，Jenkins 会把代码拉到这个节点上的一个专属目录（`$JENKINS_HOME/workspace/任务名`），构建的所有操作都在这个目录里进行。理解 workspace 能解释很多现象，比如"上一次构建留下的文件会影响下一次"（所以需要 Workspace Cleanup 插件或干净的 agent）。

### 4. 插件体系

Jenkins 核心非常精简，靠 1800+ 插件扩展：Git 插件负责拉代码、Pipeline 插件负责流水线、Credentials 插件管密钥……

- 插件从**更新中心（update center）**下载，默认是国外的 `updates.jenkins.io`，国内网络差时会大面积失败（本次踩坑）。
- 插件有**依赖链**：装 Pipeline 会自动带出一串依赖插件，一个失败可能导致整串失败。
- 国内方案：换清华/华为等镜像源，注意镜像的 json 里下载地址可能仍指回官网，需配合改 `default.json`；或直接下 `.hpi` 文件手动上传安装。

### 5. 凭据管理（Credentials）

代码仓库密码、镜像仓库账号这类敏感信息，**不能明文写在 Jenkinsfile 里**。Jenkins 有统一的凭据存储（加密保存在 master 上），任务里通过凭据 ID 引用，配合 Credentials Binding 插件注入为环境变量。本次 demo 仓库是公开的所以没用到，但这是面试必问点。

---

## 三、Pipeline 流水线原理

### 1. 为什么用 Pipeline（而不是老式自由风格任务）

老式任务（Freestyle）：在 Web 界面点选配置构建步骤。问题：**配置存在 Jenkins 里，换台 Jenkins 就没了；没法版本管理；步骤复杂后界面根本配不动**。

Pipeline：把构建流程写成代码（Jenkinsfile），和项目代码放同一个仓库。好处：

- **流水线即代码（Pipeline as Code）**：流程版本化，谁改了哪一步 git log 可查；
- **可移植**：换个 Jenkins 实例，指一下仓库地址整条流水线就回来了；
- **能表达复杂逻辑**：条件、循环、并行、审批，界面点不出来的都能写。

### 2. 声明式 vs 脚本式

- **声明式（Declarative）**：我们用的这种，`pipeline { agent ... stages { stage { steps } } }`，结构固定、易读，官方推荐。
- **脚本式（Scripted）**：`node { ... }` 开头，本质是 Groovy 脚本，更灵活但门槛高。

关键语法块：

| 块 | 作用 |
| --- | --- |
| `agent` | 这个流水线/这个阶段在哪个节点跑（any / none / label / docker / kubernetes） |
| `stages` → `stage` | 阶段划分，阶段视图里显示的一列列就是它 |
| `steps` | 阶段里具体执行的动作（sh、checkout、echo…） |
| `environment` | 定义环境变量 |
| `post` | 构建结束后的动作（always / success / failure，常用于清理和通知） |
| `when` | 阶段的执行条件（比如只有 main 分支才部署） |
| `input` | 人工审批（持续交付里"点按钮放行"就是它） |

### 3. 构建触发方式（面试高频：push 后 Jenkins 怎么知道的？）

| 方式 | 原理 | 优点 | 缺点 |
| --- | --- | --- | --- |
| **Webhook** | 仓库侧配置回调地址，有 push 时 Gitea/GitLab **主动** HTTP 通知 Jenkins | 秒级触发、零浪费 | 要配插件+两边网络互通，配置稍繁 |
| **轮询 SCM（本次用）** | Jenkins **定期**问仓库"有新提交吗"（cron 日程表控制频率） | 零插件、配置一行 | 有延迟（最长一个轮询周期）、频繁轮询有开销 |
| **定时构建（Build periodically）** | 不管有没有新代码，到点就构建 | 适合夜间全量测试 | 与提交无关 |

注意区分**轮询 SCM** 和**定时构建**：前者"有变化才构建"，后者"到点就构建"，面试里经常被混着问。

cron 语法：`分 时 日 月 周`，`* * * * *` = 每分钟。`H` 是 Jenkins 特有的哈希分散（避免所有任务同一时刻打爆仓库），但 `H/1` 的解析有坑——本次实测被当成每小时，**要每分钟就老老实实写 `* * * * *`**。

### 4. `${BUILD_NUMBER}` 与产物可追溯

`BUILD_NUMBER` 是 Jenkins 内置环境变量（还有 `BUILD_ID`、`JOB_NAME`、`GIT_COMMIT` 等）。用它做镜像 tag 的意义：**镜像和构建一一对应**，出了问题能精确回滚到"第 N 次构建的镜像"，而不是被 `latest` 覆盖得无影无踪。这是"制品不可变"思想的入门实践。

---

## 四、Docker-out-of-Docker（本次部署架构的关键原理）

面试必被追问："Jenkins 跑在容器里，它怎么执行 docker 命令？"

我们的做法：

```
Jenkins 容器 ──挂载──▶ /var/run/docker.sock（宿主机的 Docker 守护进程套接字）
              ──挂载──▶ docker 客户端二进制
```

- 容器里**没有**装 Docker 服务，只有 docker 客户端命令；
- 客户端通过挂载进来的 `docker.sock` 和**宿主机的 Docker daemon** 通信；
- 所以 `docker build` / `docker run` 实际都发生在**宿主机**上——这就是为什么部署完容器后 `docker ps` 在宿主机上能看到 demo-app。

对比另一种方案 **DinD（Docker-in-Docker，容器里再跑一个真 daemon）**：要特权模式、有存储和安全问题，一般 CI 场景优先选挂载 sock 的方式。

安全提醒（面试加分）：挂载 docker.sock ≈ 给了容器宿主机的 root 级权限，生产环境要配合权限控制（比如用 rootless 或 K8s 的 kaniko 构建）。

---

## 五、节点下线原理（本次最大坑的复盘）

**现象**：构建永远 "Waiting for next available executor"，首页 master 节点红点"未在线"。

**原理**：Jenkins 自带**节点监控（Node Monitors）**，持续检查各节点的磁盘空间、临时目录空间、时钟同步等。默认**剩余磁盘低于 1GB 就自动把节点标记离线**，防止构建把磁盘彻底写爆导致整个 Jenkins 崩溃。我们的 VM 磁盘用到 96%（剩 753M），触发监控 → master 下线 → 所有构建排队。

**排障路径**（可复用的套路）：

1. 构建排队 → 先看"构建执行状态"里节点在不在线；
2. 节点离线 → 去"系统管理 → 节点管理"看离线原因（监控会写明）；
3. 磁盘不足 → `df -h` 确认 → `docker system prune` 清理 → 节点自动恢复。

配置位置：系统管理 → 节点管理 → 节点配置里可以调整磁盘阈值。

---

## 六、Jenkins 与其他 CI/CD 工具的关系（面试横向对比）

| 工具 | 特点 | 适用场景 |
| --- | --- | --- |
| **Jenkins** | 老牌、插件生态最大、自建自托管、什么都能干但要自己维护 | 有专职运维、需要深度定制 |
| **GitLab CI** | 和 GitLab 一体，`.gitlab-ci.yml` 即流水线 | 代码托管在 GitLab |
| **GitHub Actions** | 和 GitHub 一体，市场现成 action 多 | 代码托管在 GitHub |
| **ArgoCD** | 专做 CD 的**部署**环节（GitOps）：监听 git 仓库声明的期望状态，自动同步到 K8s | K8s 环境的部署 |

关键理解：**Jenkins 和 ArgoCD 不冲突，是分工**——项目一里 Jenkins 负责 CI（构建镜像推 Harbor），ArgoCD 负责 CD（监听 Helm 仓库变化同步到 K8s）。业界常见组合就是"传统 CI 工具 + GitOps CD 工具"。

---

## 七、自测问答（背完算过关）

1. **CI、持续交付、持续部署的区别？**
   CI 是提交后自动构建测试；持续交付是代码随时处于可发布状态但需人工放行；持续部署是连放行都自动，验证通过直接上线。我们 demo 属于持续部署。

2. **push 代码后 Jenkins 怎么知道要构建？**
   两种方式：Webhook（仓库主动推送通知，秒级）和轮询 SCM（Jenkins 定期查，有延迟但零插件）。本项目用的轮询，cron 填 `* * * * *` 每分钟一次。

3. **Jenkinsfile 为什么要放在代码仓库里？**
   流水线即代码：流程版本化可追溯、可评审、换 Jenkins 实例可直接恢复，这是 GitOps 思想的组成部分。

4. **Jenkins 容器里为什么能执行 docker 命令？**
   挂载了宿主机的 `docker.sock` 和 docker 客户端，命令实际由宿主机 daemon 执行（Docker-out-of-Docker）。对比 DinD 更轻量，但要注意 sock 等于宿主机 root 权限。

5. **构建一直排队 "Waiting for next available executor" 怎么排查？**
   先看构建执行状态里节点是否在线 → 再看 executors 数量 → 节点离线就去节点管理看原因（常见是磁盘监控触发，默认剩余低于 1G 自动下线节点）。

6. **Master 和 Agent 的区别？为什么生产上不在 master 跑构建？**
   Master 负责调度和界面，Agent 负责执行。master 跑构建会拖垮 Web 服务，且构建脚本能直接读到 master 上的凭据文件，有安全隐患；扩容靠加 agent 或 K8s 动态 Pod agent。

7. **镜像 tag 为什么用 `${BUILD_NUMBER}` 而不是 latest？**
   构建号让镜像和构建一一对应，可追溯、可精确回滚；latest 会被不断覆盖，出问题无法定位是哪个版本。

8. **Jenkins 和 ArgoCD 是什么关系？**
   分工关系：Jenkins 做 CI（构建、测试、推镜像），ArgoCD 做 CD（监听 git 期望状态、同步到 K8s）。项目一就是"Jenkins 构建推 Harbor + ArgoCD 部署 K8s"的组合。

9. **轮询 SCM 和定时构建有什么区别？**
   轮询 SCM 检测到代码变化才构建；定时构建到点就构建不管有没有变化。前者适合 CI 触发，后者适合夜间全量回归。

10. **敏感信息（仓库密码、Harbor 账号）在流水线里怎么处理？**
    存进 Jenkins Credentials（加密保存），Jenkinsfile 里通过凭据 ID 引用，用 Credentials Binding 注入环境变量，绝不明文写进 Jenkinsfile 或代码仓库。
