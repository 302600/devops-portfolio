# Docker 学习详解（从零开始，白话讲解）

> 本文档目标：让完全不懂的人也能看懂 Docker，学会安装、操作、Dockerfile、数据卷、网络、Compose。
> 写作风格：生活比喻 + 逐行解释 + 对比表格 + 记忆口诀，不怕长，就怕你看不懂。

---

## 目录

- [第一章：Docker 是什么？](#第一章docker-是什么)
- [第二章：安装 Docker](#第二章安装-docker)
- [第三章：基本操作命令](#第三章基本操作命令)
- [第四章：部署常用服务](#第四章部署常用服务)
- [第五章：Dockerfile 构建自定义镜像](#第五章dockerfile-构建自定义镜像)
- [第六章：数据卷与网络](#第六章数据卷与网络)
- [第七章：Docker Compose 多容器编排](#第七章docker-compose-多容器编排)
- [第八章：命令速查表与面试考点](#第八章命令速查表与面试考点)

---

# 第一章：Docker 是什么？

## 1.1 一句话解释

Docker 是一个**打包和运行软件的工具**，把程序和它需要的所有环境打包成一个"镜像"，在任何装了 Docker 的机器上直接运行。

> **生活比喻**：
> 
> 你在家做了一道菜，端到别人家发现：锅不一样、灶不一样、调料不一样，味道全变了。
> 
> Docker 就像**外卖盒+说明书**：把菜连同锅、调料、菜谱全打包在一个盒子里。别人拿到盒子打开就能吃，不用管他家是什么灶。

## 1.2 Docker 解决什么问题？

| 问题 | 没有 Docker | 有 Docker |
|------|------------|----------|
| 部署环境 | 手动装依赖、配环境，容易出错 | 镜像打包好，一键运行 |
| 环境不一致 | "在我电脑上能跑啊" | 哪里都能跑 |
| 部署速度 | 几十分钟到几小时 | 几秒钟 |
| 资源占用 | 虚拟机每个几GB | 容器每个几十MB |

## 1.3 Docker vs 虚拟机（面试必问）

### 虚拟机

```
┌─────────────────────────────┐
│         物理机               │
│  ┌───────────────────────┐  │
│  │    虚拟机管理程序       │  │
│  │  ┌────────┐ ┌────────┐│  │
│  │  │完整系统  │ │完整系统  ││  │
│  │  │Windows  │ │CentOS  ││  │
│  │  │  应用    │ │  应用   ││  │
│  │  └────────┘ └────────┘│  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

虚拟机 = **完整的操作系统**，启动慢、占资源大。

### Docker

```
┌─────────────────────────────┐
│         物理机               │
│  ┌───────────────────────┐  │
│  │     Docker 引擎        │  │
│  │  ┌──────┐┌──────┐┌──────┐│
│  │  │Nginx ││MySQL ││Redis ││
│  │  └──────┘└──────┘└──────┘│
│  │   共用主机的Linux内核    │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

Docker 容器 = **只有应用和依赖**，不包含完整操作系统，直接共用主机的内核。

> **生活比喻**：
> - 虚拟机 = 买了一整套新房（地基、墙、屋顶、水电全有），住进去。贵、慢、占地方。
> - Docker = 租了个**精装公寓**（水电墙是楼里共用的，你只带行李入住）。便宜、快、省空间。

### 对比表

| 对比项 | 虚拟机 | Docker 容器 |
|--------|--------|------------|
| 有没有完整系统 | 有 | 没有（共用内核） |
| 启动速度 | 慢（几十秒到几分钟） | 极快（毫秒到几秒） |
| 占用资源 | 大（每个几GB） | 小（每个几十MB） |
| 隔离程度 | 强（完全隔离） | 较弱（共用内核） |
| 生活比喻 | 买新房 | 租公寓 |

## 1.4 Docker 三个核心概念

| 概念 | 英文 | 是什么 | 生活比喻 |
|------|------|--------|---------|
| **镜像** | Image | 只读模板，包含程序+依赖+配置 | 菜谱+食材包 |
| **容器** | Container | 镜像跑起来的实例 | 做好的那盘菜 |
| **仓库** | Registry | 存镜像的公共仓库 | 应用商店 |

三者关系：

```
  仓库（Docker Hub）          镜像（模板）           容器（实例）
  ┌──────────┐   docker pull  ┌──────────┐  docker run  ┌──────────┐
  │  nginx   │ ─────────────► │  nginx   │ ──────────► │  nginx   │
  │  mysql   │                │  镜像     │             │  容器     │
  │  redis   │                │（只读）   │             │（可读写） │
  └──────────┘                └──────────┘             └──────────┘
   应用商店                    菜谱+食材包               做好的菜
```

> 一句话记住：**从仓库拉镜像（pull），用镜像跑容器（run）**。

## 1.5 Docker 架构

Docker 是**客户端-服务端**模式：

```
┌──────────────────────────────────┐
│              你的机器              │
│                                  │
│  ┌──────────┐     ┌──────────┐   │
│  │ Docker   │命令 │ Docker   │   │
│  │ Client   │────►│ Daemon   │   │
│  │ (客户端)  │     │ (守护进程)│   │
│  │          │     │          │   │
│  │ docker   │     │ 管理镜像  │   │
│  │ 命令行   │     │ 管理容器  │   │
│  └──────────┘     │ 管理网络  │   │
│                    └────┬─────┘   │
│                    ┌────┴─────┐   │
│               ┌────┴──┐ ┌────┴──┐│
│               │容器1   │ │容器2   ││
│               │Nginx  │ │MySQL  ││
│               └───────┘ └───────┘│
└──────────────────────────────────┘
```

> **生活比喻**：
> 你（客户端）去饭店点菜 → 服务员传单给后厨（守护进程） → 后厨做好菜端给你

---

# 第二章：安装 Docker

## 2.1 环境说明

- 操作系统：CentOS 7
- 机器：hadoop101（192.168.121.101）

## 2.2 安装步骤

### 第 1 步：安装 yum 工具包

```bash
yum install -y yum-utils
```

- `yum install` —— 安装软件
- `-y` —— 自动回答yes
- `yum-utils` —— yum 的扩展工具包

### 第 2 步：添加 Docker 阿里云镜像源

```bash
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

> CentOS 默认源里没有 Docker，需要加阿里云的源（国内下载快）。

### 第 3 步：安装 Docker

```bash
yum install -y docker-ce docker-ce-cli containerd.io
```

三个包的含义：
- `docker-ce` —— Docker 社区版（ce = Community Edition，免费版）
- `docker-ce-cli` —— Docker 命令行工具
- `containerd.io` —— 容器运行时（Docker 的底层引擎）

### 第 4 步：启动 Docker 并设置开机自启

```bash
systemctl start docker
systemctl enable docker
```

### 第 5 步：验证

```bash
docker version
```

看到版本号就说明安装成功了。

## 2.3 配置国内镜像加速

```bash
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.m.daocloud.io"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

> 相当于给 Docker 配了一个"国内快递代理"，下载镜像速度快很多。
> 如果这个源不好用，可以换成 `https://docker.1panel.live`

## 2.4 运行第一个容器

```bash
docker run hello-world
```

看到 `Hello from Docker!` 就说明 Docker 完全正常。

---

# 第三章：基本操作命令

## 3.1 Docker 工作流程

```
docker pull 镜像名     ← 从仓库下载镜像（从应用商店下载APP）
      │
docker images          ← 查看本地有哪些镜像（看手机装了哪些APP）
      │
docker run 镜像名       ← 用镜像启动容器（打开APP开始用）
      │
docker ps              ← 查看哪些容器在运行（看哪些APP开着）
      │
docker stop 容器ID     ← 停止容器（关闭APP）
      │
docker rm 容器ID       ← 删除容器（删APP使用记录）
      │
docker rmi 镜像名      ← 删除镜像（卸载APP本身）
```

> **记忆口诀**：pull拉 → run跑 → stop停 → rm删容器 → rmi删镜像

## 3.2 镜像相关命令

### docker pull —— 下载镜像

```bash
docker pull nginx        # 下载最新版nginx镜像
docker pull redis        # 下载最新版redis镜像
docker pull mysql:5.7    # 下载5.7版本的mysql镜像
```

> 就像从应用商店下载APP。不加版本号默认下载 latest（最新版）。

### docker images —— 查看本地镜像

```bash
docker images
```

输出格式：
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    605c77e624dd   2 weeks ago    142MB
redis         latest    7614ae9453d1   3 weeks ago    117MB
```

各列含义：
- REPOSITORY —— 镜像名字
- TAG —— 版本标签（latest = 最新版）
- IMAGE ID —— 镜像唯一编号
- CREATED —— 创建时间
- SIZE —— 镜像大小

### docker rmi —— 删除镜像

```bash
docker rmi nginx          # 删除nginx镜像
docker rmi 镜像ID         # 用ID删除（不用写全，前几位就行）
```

> 注意：如果有容器正在使用这个镜像，会报错。需要先删容器再删镜像。

## 3.3 容器相关命令

### docker run —— 启动容器（最重要的命令）

```bash
docker run [参数] 镜像名
```

常用参数：

| 参数 | 作用 | 举例 | 记忆 |
|------|------|------|------|
| `-d` | 后台运行 | `docker run -d nginx` | d = daemon（后台） |
| `-p` | 端口映射 | `-p 8080:80` | p = port（端口） |
| `--name` | 给容器起名字 | `--name my-nginx` | name = 名字 |
| `-v` | 挂载目录 | `-v /data:/var/lib` | v = volume（卷） |
| `-e` | 传环境变量 | `-e MYSQL_ROOT_PASSWORD=123456` | e = environment（环境） |
| `--network` | 加入网络 | `--network mynet` | network = 网络 |
| `--restart=always` | 开机自启 | `--restart=always` | restart = 重启 |
| `-it` | 交互式进入 | `docker exec -it 容器 bash` | i=交互 t=终端 |

完整示例：

```bash
docker run -d \
  --name my-nginx \
  -p 8080:80 \
  -v /data/nginx:/usr/share/nginx/html \
  --restart=always \
  nginx
```

> 逐行解释：
> - `-d` —— 后台运行，不占住终端
> - `--name my-nginx` —— 给容器起名 my-nginx
> - `-p 8080:80` —— 虚拟机8080端口映射到容器80端口
> - `-v /data/nginx:/usr/share/nginx/html` —— 虚拟机目录映射到容器目录
> - `--restart=always` —— 虚拟机重启后容器自动启动
> - `nginx` —— 用哪个镜像

### docker ps —— 查看容器

```bash
docker ps          # 只看运行中的容器
docker ps -a       # 看所有容器（包括已停止的）
docker ps -q       # 只显示容器ID（常配合其他命令用）
```

> `docker ps` = 只看活着的
> `docker ps -a` = 连死的也看（a = all）
> `docker ps -q` = 只看ID号（q = quiet）

### docker stop —— 停止容器

```bash
docker stop 容器ID         # 停止一个容器
docker stop 容器名          # 也可以用名字
docker stop $(docker ps -q)  # 停止所有运行中的容器
```

### docker start —— 启动已停止的容器

```bash
docker start 容器ID
```

> stop 是"关机"，start 是"开机"，容器还在，没被删除。

### docker restart —— 重启容器

```bash
docker restart 容器ID
```

### docker rm —— 删除容器

```bash
docker rm 容器ID           # 删除已停止的容器
docker rm -f 容器ID        # 强制删除（正在运行的也能删）
docker rm $(docker ps -aq)  # 删除所有容器
```

> 先 stop 再 rm，就像先关机再搬电脑。
> 加 `-f` 可以不用先停就删（f = force，强制）。

### docker exec —— 进入容器执行命令

```bash
docker exec -it 容器名 bash         # 进入容器的bash终端
docker exec -it my-redis redis-cli  # 进入容器的redis-cli
docker exec -it my-mysql mysql -uroot -p123456  # 进入MySQL
```

> `docker exec -it` 就像"远程登录"到容器内部。
> - `-i` = 保持输入流打开
> - `-t` = 分配一个终端
> - 合起来 `-it` = 给你一个可以打字的终端窗口

### docker logs —— 查看容器日志

```bash
docker logs 容器名          # 查看全部日志
docker logs -f 容器名       # 实时跟踪日志（像tail -f）
docker logs --tail 20 容器名  # 只看最后20行
```

### docker inspect —— 查看容器详细信息

```bash
docker inspect 容器名
```

> 包括容器的IP地址、挂载信息、网络信息等，排查问题时很有用。

## 3.4 端口映射详解

```bash
docker run -d -p 8080:80 nginx
```

```
外部访问：http://虚拟机IP:8080
              │
              ▼
    ┌─────────────────┐
    │     虚拟机        │
    │  端口 8080 ───────┼──► 容器端口 80（Nginx）
    │                  │
    └─────────────────┘
```

> 容器是隔离的，外面访问不到容器内部的端口。
> `-p 8080:80` 就是开一个"隧道"：访问虚拟机的8080 → 隧道传到容器的80。

## 3.5 命令速查表

| 命令 | 作用 | 记忆 |
|------|------|------|
| `docker pull 镜像名` | 下载镜像 | pull = 拉 |
| `docker images` | 查看本地镜像 | images = 镜像列表 |
| `docker rmi 镜像名` | 删除镜像 | rmi = remove image |
| `docker run 镜像名` | 启动容器 | run = 跑 |
| `docker ps` | 看运行中的容器 | ps = process status |
| `docker ps -a` | 看所有容器 | a = all |
| `docker stop 容器` | 停止容器 | stop = 停 |
| `docker start 容器` | 启动已停的容器 | start = 开 |
| `docker restart 容器` | 重启容器 | restart = 重启 |
| `docker rm 容器` | 删除容器 | rm = remove |
| `docker exec -it 容器 bash` | 进入容器 | exec = 执行 |
| `docker logs 容器` | 查看日志 | logs = 日志 |
| `docker inspect 容器` | 查看详情 | inspect = 检查 |

---

# 第四章：部署常用服务

## 4.1 用 Docker 跑 Nginx

### 基础版

```bash
docker pull nginx
docker run -d --name my-nginx -p 8080:80 nginx
curl localhost:8080
```

### 挂载自定义网页

```bash
mkdir -p /data/nginx
echo '<h1>Hello Docker!</h1>' > /data/nginx/index.html

docker run -d \
  --name my-nginx \
  -p 8080:80 \
  -v /data/nginx:/usr/share/nginx/html \
  nginx
```

> `-v /data/nginx:/usr/share/nginx/html` 解释：
> - 左边 `/data/nginx` = 虚拟机上的目录（你的文件在这）
> - 右边 `/usr/share/nginx/html` = 容器内Nginx的网页目录
> - 冒号 `:` 连接两边，形成"通道"
> - 你在虚拟机改文件，容器内立刻生效，不用重启

### 修改网页不用重启容器

```bash
echo '<h1>Changed!</h1>' > /data/nginx/index.html
curl localhost:8080    # 内容立刻变了
```

## 4.2 用 Docker 跑 MySQL

```bash
docker run -d \
  --name my-mysql \
  -p 3307:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v /data/mysql:/var/lib/mysql \
  --restart=always \
  mysql:5.7
```

参数解释：
- `-p 3307:3306` —— 用3307端口（避免和宿主机MySQL的3306冲突）
- `-e MYSQL_ROOT_PASSWORD=123456` —— 设置root密码（`-e` = 环境变量）
- `-v /data/mysql:/var/lib/mysql` —— 数据存到宿主机，容器删了数据还在
- `mysql:5.7` —— 指定5.7版本

连接 MySQL：

```bash
docker exec -it my-mysql mysql -uroot -p123456
```

## 4.3 用 Docker 跑 Redis

```bash
docker run -d \
  --name my-redis \
  -p 6380:6379 \
  --restart=always \
  redis
```

连接 Redis：

```bash
docker exec -it my-redis redis-cli
```

## 4.4 -e 环境变量详解

`-e` 就是给容器传配置参数，不同镜像需要不同的环境变量：

| 镜像 | 常用环境变量 | 作用 |
|------|------------|------|
| mysql | `MYSQL_ROOT_PASSWORD=123456` | 设置root密码 |
| mysql | `MYSQL_DATABASE=mydb` | 自动创建数据库 |
| mysql | `MYSQL_USER=app` | 创建普通用户 |
| redis | `REDIS_PASSWORD=123456` | 设置密码（需指定配置） |
| nginx | 无常用环境变量 | — |

> **生活比喻**：`-e` 就像你安装软件时填的设置项。
> "你的密码是什么？" → `-e MYSQL_ROOT_PASSWORD=123456`

## 4.5 -v 数据卷详解

### 为什么需要 -v？

**没有 -v（数据会丢失）**：

```bash
docker run -d --name test redis                      # 启动
docker exec -it test redis-cli SET name "zhangsan"   # 写数据
docker stop test && docker rm test                    # 删容器
docker run -d --name test redis                      # 重新启动
docker exec -it test redis-cli GET name              # 数据没了！(nil)
```

**有 -v（数据永久保存）**：

```bash
mkdir -p /data/redis-data
docker run -d --name test -v /data/redis-data:/data redis  # 挂载
docker exec -it test redis-cli SET name "zhangsan"          # 写数据
docker stop test && docker rm test                          # 删容器
docker run -d --name test -v /data/redis-data:/data redis  # 重新启动
docker exec -it test redis-cli GET name                    # 数据还在！"zhangsan"
```

> **结论**：重要数据（MySQL数据、Redis数据、日志）**必须挂载 -v**。

### -v 语法

```bash
-v 宿主机目录:容器目录
```

| 常见服务 | 宿主机目录 | 容器目录 | 说明 |
|---------|-----------|---------|------|
| MySQL | /data/mysql | /var/lib/mysql | 数据文件 |
| Redis | /data/redis | /data | 数据文件 |
| Nginx | /data/nginx | /usr/share/nginx/html | 网页文件 |
| Nginx | /data/nginx.conf | /etc/nginx/nginx.conf | 配置文件 |

---

# 第五章：Dockerfile 构建自定义镜像

## 5.1 什么是 Dockerfile？

Dockerfile 就是一个**菜谱文件**，里面写了一步步怎么做菜（怎么构建镜像）。

> **生活比喻**：
> 之前你用的是别人做好的"方便面"（从 Docker Hub 下载现成镜像）。
> Dockerfile 就是让你**自己写菜谱**，按你的口味做菜。

## 5.2 Dockerfile 常用指令

| 指令 | 作用 | 生活比喻 | 举例 |
|------|------|---------|------|
| `FROM` | 基于哪个镜像 | 用什么锅底 | `FROM nginx` |
| `RUN` | 构建时执行命令 | 做菜过程中加调料 | `RUN apt-get install -y vim` |
| `COPY` | 复制文件到镜像 | 把食材放进锅里 | `COPY index.html /usr/share/nginx/html/` |
| `WORKDIR` | 设置工作目录 | 在哪个台面操作 | `WORKDIR /app` |
| `EXPOSE` | 声明端口 | 菜从哪个窗口端出 | `EXPOSE 80` |
| `CMD` | 启动时执行的命令 | 最后怎么上桌 | `CMD ["nginx", "-g", "daemon off;"]` |
| `ENV` | 设置环境变量 | 准备调料 | `ENV MYSQL_ROOT_PASSWORD=123456` |

### RUN vs CMD 的区别（面试常问）

| 对比项 | RUN | CMD |
|--------|-----|-----|
| 什么时候执行 | 构建镜像时 | 启动容器时 |
| 执行几次 | 只执行一次（打包进镜像） | 每次启动容器都执行 |
| 生活比喻 | 做菜过程中加调料 | 菜做好了端上桌 |

## 5.3 实操：制作自定义 Nginx 镜像

### 第 1 步：准备文件

```bash
mkdir -p /data/dockerfile
cd /data/dockerfile
echo '<h1>This is my custom Nginx image!</h1>' > index.html
```

> 注意：字符串里有 `!` 时必须用**单引号**，不能用双引号，否则 bash 会报错。

### 第 2 步：编写 Dockerfile

```bash
cat > Dockerfile <<'EOF'
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

逐行解释：
- `FROM nginx` —— 基于 Nginx 官方镜像
- `COPY index.html /usr/share/nginx/html/index.html` —— 把网页文件复制进镜像
- `EXPOSE 80` —— 声明用 80 端口

> 文件名必须叫 `Dockerfile`，首字母大写，没有后缀名。

### 第 3 步：构建镜像

```bash
docker build -t my-nginx:v1 .
```

- `docker build` —— 构建镜像命令
- `-t my-nginx:v1` —— 给镜像起名字:版本号
- `.` —— 当前目录（Docker 会自动找当前目录的 Dockerfile）

> 最后那个 `.` 不能漏！它代表"用当前目录的 Dockerfile"。

### 第 4 步：运行自定义镜像

```bash
docker run -d --name custom-nginx -p 8081:80 my-nginx:v1
curl localhost:8081
```

看到 `This is my custom Nginx image!` 就成功了！

## 5.4 Dockerfile 更完整的示例

```dockerfile
FROM nginx:1.20

# 安装额外工具
RUN apt-get update && apt-get install -y vim curl

# 设置工作目录
WORKDIR /usr/share/nginx/html

# 复制网页文件
COPY index.html .
COPY css/ ./css/
COPY js/ ./js/

# 声明端口
EXPOSE 80

# 启动命令
CMD ["nginx", "-g", "daemon off;"]
```

---

# 第六章：数据卷与网络

## 6.1 数据卷回顾

上一章已经讲了 `-v`，这里总结一下：

| 作用 | 没有数据卷 | 有数据卷 |
|------|-----------|---------|
| 容器删了 | 数据丢失 | 数据保留 |
| 修改配置 | 要进容器改 | 在宿主机改，立即生效 |
| 生活比喻 | 一次性纸杯 | 接了水管的杯子 |

语法：`-v 宿主机目录:容器目录`

## 6.2 Docker 网络

### 为什么需要网络？

默认情况下，容器和容器之间是**隔离的**，互相访问不到。

> **生活比喻**：每个容器就像一个独立的小房间，默认房门锁着，互相串不了门。
> Docker 网络就是给容器们开一个"公共走廊"，让它们能互相访问。

### 网络相关命令

```bash
docker network create mynet       # 创建一个网络
docker network ls                 # 查看所有网络
docker network inspect mynet      # 查看网络详情（看有哪些容器）
docker network rm mynet           # 删除网络
```

### 实操：让容器之间通信

```bash
# 第1步：创建网络
docker network create mynet

# 第2步：启动Redis，加入网络
docker run -d --name my-redis --network mynet redis

# 第3步：启动Nginx，加入网络
docker run -d --name my-nginx --network mynet -p 8080:80 nginx

# 第4步：验证连通性（用busybox测试ping）
docker run --rm --network mynet busybox ping my-redis -c 3
```

> **关键点**：同一个网络里的容器，可以用**容器名**互相访问，不用记IP！
> Docker 自动把容器名解析成 IP，就像 DNS 一样。

### 容器间通信原理

```
┌───────────────┐     ┌───────────────┐
│  my-nginx     │     │  my-redis     │
│  172.18.0.3   │     │  172.18.0.2   │
└───────┬───────┘     └───────┬───────┘
        │                      │
        │    mynet 网络         │
        └──────────┬───────────┘
                   │
          Docker 自动DNS解析
          my-redis → 172.18.0.2
          my-nginx → 172.18.0.3
```

> Nginx 容器想访问 Redis，直接用 `my-redis:6379` 就行，不用写 IP。

---

# 第七章：Docker Compose 多容器编排

## 7.1 什么是 Docker Compose？

Docker Compose 就是把多条 `docker run` 命令**写成一个文件**，一条命令全部启动。

> **生活比喻**：
> - `docker run` = 你一道一道菜单独点，每道报一堆要求
> - `docker compose` = 你直接说"来个套餐A"，后厨自动做好端上来

## 7.2 为什么要用 Compose？

假设你要部署一个网站，需要 Nginx + PHP + MySQL + Redis：

- 不用 Compose：敲 4 条很长的 `docker run` 命令，参数多，容易写错
- 用 Compose：写一个 `docker-compose.yml`，`docker compose up -d` 一键启动

## 7.3 docker-compose.yml 文件结构

```yaml
version: "3"                # 文件版本（新版可省略）

services:                    # 下面定义所有服务
  nginx:                     # 服务1：nginx
    image: nginx             # 用哪个镜像
    ports:                   # 端口映射
      - "8080:80"            # 虚拟机8080 → 容器80
    depends_on:              # 依赖关系
      - redis                # 先启动redis再启动nginx
    restart: always          # 开机自启

  redis:                     # 服务2：redis
    image: redis
    restart: always
```

### docker-compose.yml 里的参数对照

| yml参数 | 对应docker run参数 | 作用 |
|---------|-------------------|------|
| `image: nginx` | `nginx` | 用哪个镜像 |
| `ports: ["8080:80"]` | `-p 8080:80` | 端口映射 |
| `volumes: ["/data:/var"]` | `-v /data:/var` | 挂载目录 |
| `environment:` | `-e` | 环境变量 |
| `restart: always` | `--restart=always` | 开机自启 |
| `depends_on:` | 无 | 启动顺序依赖 |

## 7.4 Docker Compose 命令

> **注意**：新版 Docker 用 `docker compose`（空格），旧版用 `docker-compose`（横杠）。
> 先试 `docker compose version`，能用就用空格版。

| 命令 | 作用 | 记忆 |
|------|------|------|
| `docker compose up -d` | 一键启动所有服务 | up = 拉起来 |
| `docker compose down` | 一键停止并删除 | down = 放下去 |
| `docker compose ps` | 查看服务状态 | ps = 进程状态 |
| `docker compose logs` | 查看日志 | logs = 日志 |
| `docker compose restart` | 重启所有服务 | restart = 重启 |
| `docker compose stop` | 只停止不删除 | stop = 停 |
| `docker compose start` | 启动已停止的服务 | start = 开 |

## 7.5 实操：用 Compose 部署 Nginx + Redis

### 第 1 步：创建目录和文件

```bash
mkdir -p /data/compose
cd /data/compose

cat > docker-compose.yml <<'EOF'
version: "3"
services:
  nginx:
    image: nginx
    ports:
      - "8080:80"
    depends_on:
      - redis
    restart: always

  redis:
    image: redis
    restart: always
EOF
```

### 第 2 步：一键启动

```bash
docker compose up -d
```

> Compose 会自动：
> 1. 创建一个网络
> 2. 启动 redis 容器
> 3. 启动 nginx 容器
> 4. 两个容器自动在同一个网络里，能互相通信

### 第 3 步：验证

```bash
docker compose ps          # 查看状态
curl localhost:8080        # 测试Nginx
```

### 第 4 步：一键停止

```bash
docker compose down
```

> `down` = 停止 + 删除所有容器 + 删除网络，一条命令全部收拾干净。

## 7.6 更复杂的 Compose 示例

```yaml
services:
  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - /data/nginx/html:/usr/share/nginx/html
      - /data/nginx/conf:/etc/nginx/conf.d
    depends_on:
      - php
      - mysql
    restart: always

  php:
    image: php:7.4-fpm
    volumes:
      - /data/nginx/html:/var/www/html
    restart: always

  mysql:
    image: mysql:5.7
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      MYSQL_DATABASE: "mydb"
    volumes:
      - /data/mysql:/var/lib/mysql
    restart: always

  redis:
    image: redis
    restart: always
```

> 这就是用 Compose 部署完整的 LNMP + Redis，一个文件搞定四个服务！

---

# 第八章：命令速查表与面试考点

## 8.1 Docker 全部命令速查表

### 镜像命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `docker pull` | 下载镜像 | `docker pull nginx` |
| `docker images` | 查看本地镜像 | `docker images` |
| `docker rmi` | 删除镜像 | `docker rmi nginx` |
| `docker build` | 构建镜像 | `docker build -t myimg:v1 .` |
| `docker tag` | 给镜像打标签 | `docker tag myimg:v1 myimg:v2` |
| `docker save` | 导出镜像为文件 | `docker save nginx -o nginx.tar` |
| `docker load` | 从文件导入镜像 | `docker load -i nginx.tar` |

### 容器命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `docker run` | 创建并启动容器 | `docker run -d --name web -p 8080:80 nginx` |
| `docker ps` | 查看运行中的容器 | `docker ps` |
| `docker ps -a` | 查看所有容器 | `docker ps -a` |
| `docker stop` | 停止容器 | `docker stop web` |
| `docker start` | 启动已停的容器 | `docker start web` |
| `docker restart` | 重启容器 | `docker restart web` |
| `docker rm` | 删除容器 | `docker rm web` |
| `docker exec` | 进入容器执行命令 | `docker exec -it web bash` |
| `docker logs` | 查看日志 | `docker logs web` |
| `docker inspect` | 查看详情 | `docker inspect web` |
| `docker cp` | 容器与宿主机互拷文件 | `docker cp web:/etc/hosts /tmp/` |
| `docker stats` | 查看资源占用 | `docker stats` |
| `docker top` | 查看容器内进程 | `docker top web` |

### 网络命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `docker network create` | 创建网络 | `docker network create mynet` |
| `docker network ls` | 查看网络 | `docker network ls` |
| `docker network inspect` | 查看网络详情 | `docker network inspect mynet` |
| `docker network rm` | 删除网络 | `docker network rm mynet` |

### Compose 命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `docker compose up -d` | 启动所有服务 | `docker compose up -d` |
| `docker compose down` | 停止并删除 | `docker compose down` |
| `docker compose ps` | 查看状态 | `docker compose ps` |
| `docker compose logs` | 查看日志 | `docker compose logs` |
| `docker compose restart` | 重启服务 | `docker compose restart` |
| `docker compose stop` | 只停不删 | `docker compose stop` |

### 系统命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `docker system df` | 查看磁盘占用 | `docker system df` |
| `docker system prune` | 清理无用资源 | `docker system prune -a` |
| `docker info` | 查看Docker信息 | `docker info` |
| `docker version` | 查看版本 | `docker version` |

## 8.2 docker run 参数速查表

| 参数 | 作用 | 示例 |
|------|------|------|
| `-d` | 后台运行 | `docker run -d nginx` |
| `-p` | 端口映射 | `-p 8080:80` |
| `--name` | 容器名字 | `--name my-nginx` |
| `-v` | 挂载目录 | `-v /data:/var/lib` |
| `-e` | 环境变量 | `-e MYSQL_ROOT_PASSWORD=123456` |
| `--network` | 加入网络 | `--network mynet` |
| `--restart=always` | 开机自启 | `--restart=always` |
| `-it` | 交互式终端 | `docker exec -it web bash` |
| `--rm` | 用完即删 | `docker run --rm busybox ping a` |

## 8.3 面试高频考点

### Q1：Docker 和虚拟机的区别？

| 对比项 | 虚拟机 | Docker |
|--------|--------|--------|
| 原理 | 完整操作系统 | 共用宿主机内核 |
| 启动速度 | 慢（分钟级） | 快（秒级） |
| 资源占用 | 大（GB级） | 小（MB级） |
| 隔离性 | 强 | 较弱 |
| 生活比喻 | 买新房 | 租公寓 |

### Q2：镜像和容器的区别？

- **镜像（Image）** = 只读模板，相当于"菜谱+食材包"
- **容器（Container）** = 镜像运行起来的实例，相当于"做好的菜"
- 一个镜像可以创建多个容器

### Q3：Dockerfile 中 RUN 和 CMD 的区别？

- **RUN** = 构建镜像时执行，只执行一次，结果打包进镜像
- **CMD** = 启动容器时执行，每次启动都执行

### Q4：为什么需要数据卷（-v）？

容器是临时的，删除后内部数据会丢失。用 `-v` 把宿主机目录映射到容器内，数据存在宿主机上，容器删了数据还在。

### Q5：Docker 网络是怎么实现容器间通信的？

同一网络中的容器，Docker 会自动提供 DNS 解析，可以用容器名代替 IP 地址互相访问。

### Q6：Docker Compose 是什么？

一个用 YAML 文件定义和管理多个 Docker 容器的工具。用 `docker compose up -d` 一键启动所有服务，`docker compose down` 一键停止。

### Q7：如何让容器开机自启？

在 `docker run` 时加 `--restart=always` 参数。

### Q8：如何进入运行中的容器？

```bash
docker exec -it 容器名 bash
```

### Q9：如何查看容器日志？

```bash
docker logs 容器名           # 查看全部
docker logs -f 容器名        # 实时跟踪
docker logs --tail 20 容器名  # 最后20行
```

### Q10：docker pull 和 docker run 的关系？

- `docker pull` = 只下载镜像，不运行
- `docker run` = 如果本地没有镜像会自动下载，然后运行
- `docker run` = `docker pull` + 创建容器 + 启动容器

## 8.4 常见问题排查

### 问题1：端口被占用

```
Error: Bind for 0.0.0.0:8080 failed: port is already allocated
```

**解决**：8080端口被其他容器或进程占了。

```bash
docker ps                    # 看哪个容器占了端口
docker stop 占用端口的容器     # 停掉它
# 或者换个端口，比如改成8081
docker run -d -p 8081:80 nginx
```

### 问题2：镜像下载很慢或卡住

**解决**：配置国内镜像加速源。

```bash
cat > /etc/docker/daemon.json <<'EOF'
{
  "registry-mirrors": ["https://docker.1panel.live"]
}
EOF
systemctl restart docker
```

如果还是卡，`Ctrl+C` 取消重新拉，已下载的层会缓存。

### 问题3：容器名冲突

```
Error: Conflict. The container name "my-nginx" is already in use
```

**解决**：同名容器已存在，先删掉旧的。

```bash
docker stop my-nginx 2>/dev/null
docker rm my-nginx 2>/dev/null
# 然后重新 docker run
```

### 问题4：bash 中含 ! 的字符串报错

```
-bash: !!: event not found
```

**解决**：把双引号改成单引号。

```bash
# 错误
echo "<h1>Hello!</h1>" > index.html

# 正确
echo '<h1>Hello!</h1>' > index.html
```

### 问题5：docker-compose 命令找不到

**解决**：新版 Docker 自带 `docker compose`（空格），不用单独安装。

```bash
docker compose version    # 检查是否有空格版
docker compose up -d      # 用空格版代替横杠版
```

## 8.5 Docker 学习路线回顾

```
第1步：安装Docker              ✓ 已完成
  └─ yum install docker-ce

第2步：基本操作                 ✓ 已完成
  └─ pull / run / stop / rm / ps / images

第3步：部署常用服务              ✓ 已完成
  └─ Nginx / MySQL / Redis 一条命令启动

第4步：Dockerfile              ✓ 已完成
  └─ FROM / RUN / COPY / CMD 构建自定义镜像

第5步：数据卷与网络              ✓ 已完成
  └─ -v 数据持久化 / --network 容器互通

第6步：Docker Compose          ✓ 已完成
  └─ docker-compose.yml 一键管理多容器
```

---

> **文档总结**：
> Docker 核心就是6个模块：安装 → 基本操作 → 部署服务 → Dockerfile → 数据卷与网络 → Compose。
> 
> 记住核心口诀：
> - **镜像**是模板（菜谱），**容器**是实例（做好的菜）
> - **pull**拉镜像，**run**跑容器，**stop**停容器，**rm**删容器，**rmi**删镜像
> - **-p**映射端口，**-v**挂载目录，**-e**传环境变量，**--name**起名字
> - **Dockerfile** = 自己写菜谱做镜像
> - **Docker Compose** = 一键管理多个容器
> 
> 这就是 Docker 的完整知识体系！
