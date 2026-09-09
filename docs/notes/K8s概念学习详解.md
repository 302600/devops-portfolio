# K8s（Kubernetes）概念学习详解

> **学习目标**：面试时能说清K8s是什么、核心概念（Pod/Deployment/Service）、kubectl常用命令
> **学习方式**：只学概念，不实操（虚拟机跑不动K8s集群）
> **前置知识**：Docker ✅（K8s就是管理Docker容器的工具）

---

## 第一章：K8s是什么？为什么需要它？

### 1.1 一句话理解

**K8s就是Docker容器的"集群管家"**

- Docker：把应用打包成容器，一台机器上跑几个容器
- K8s：管理几百上千个容器，跨多台机器，自动调度、扩容、故障恢复

### 1.2 生活比喻

| 概念 | 比喻 |
|------|------|
| Docker | 一个厨师（在一个厨房里做菜） |
| Docker Compose | 一个厨房里的多个厨师协同（一台机器上管理多个容器） |
| **K8s** | **连锁餐厅总部**（管理多个厨房、几百个厨师、自动分配任务、厨师病了自动换人） |

### 1.3 为什么需要K8s？

假设你用Docker部署了一个Web应用，只有1台服务器：
- 服务器挂了 → 应用全挂了
- 流量突然增大 → 1台扛不住
- 手动在10台机器上各跑一个容器 → 累死

**K8s解决的就是这些问题**：
- 自动调度：决定容器跑在哪台机器上
- 自动扩缩容：流量大了自动多跑几个容器，流量小了自动减少
- 自愈能力：某个容器挂了，自动重新启动一个
- 滚动更新：升级应用时不中断服务

### 1.4 K8s vs Docker Compose

| 对比项 | Docker Compose | K8s |
|--------|---------------|-----|
| 管理范围 | 单台机器 | 多台机器（集群） |
| 容器数量 | 几个~十几个 | 几百~几千个 |
| 自动调度 | ❌ 手动指定 | ✅ 自动决定跑在哪 |
| 故障恢复 | ❌ 挂了不自动恢复 | ✅ 自动重启 |
| 自动扩缩容 | ❌ | ✅ |
| 滚动更新 | ❌ | ✅ |
| 学习难度 | 简单 | 复杂 |
| 适用场景 | 开发/测试 | 生产环境 |

> **记忆口诀**：Compose管单机，K8s管集群；Compose手动配，K8s自动调。

---

## 第二章：K8s架构（Master+Worker）

### 2.1 整体架构

K8s集群 = 1个Master节点（控制中心） + N个Worker节点（干活的）

```
┌─────────────────────────────────────────┐
│              Master节点（大脑）            │
│  ┌─────────┐ ┌──────────┐ ┌──────────┐  │
│  │ API Server│ │ Scheduler│ │Controller│  │
│  │ (前台)    │ │ (调度员)  │ │ Manager  │  │
│  └─────────┘ └──────────┘ └──────────┘  │
│  ┌──────────────────────────────────┐    │
│  │           etcd（记事本）           │    │
│  └──────────────────────────────────┘    │
└─────────────────────────────────────────┘
        │                    │
        │  下发任务           │
        ▼                    ▼
┌──────────────┐    ┌──────────────┐
│ Worker节点1    │    │ Worker节点2    │
│ ┌──────────┐  │    │ ┌──────────┐  │
│ │kubelet   │  │    │ │kubelet   │  │
│ │(车间主任) │  │    │ │(车间主任) │  │
│ ├──────────┤  │    │ ├──────────┤  │
│ │kube-proxy│  │    │ │kube-proxy│  │
│ │(网络管家) │  │    │ │(网络管家) │  │
│ ├──────────┤  │    │ ├──────────┤  │
│ │  Docker  │  │    │ │  Docker  │  │
│ │ (厨师)   │  │    │ │ (厨师)   │  │
│ └──────────┘  │    │ └──────────┘  │
└──────────────┘    └──────────────┘
```

### 2.2 Master节点组件（4个）

| 组件 | 比喻 | 作用 |
|------|------|------|
| **API Server** | 前台接待 | 所有操作都经过它，kubectl命令就是发给它的 |
| **Scheduler** | 调度员 | 决定新容器跑在哪个Worker节点上 |
| **Controller Manager** | 监工 | 盯着所有容器，挂了就重启，少了就补 |
| **etcd** | 记事本 | 存储所有配置信息（键值对数据库） |

> **记忆口诀**：前台(API)接命令，调度(Scheduler)分任务，监工(Controller)盯状态，记事本(etcd)存所有。

### 2.3 Worker节点组件（3个）

| 组件 | 比喻 | 作用 |
|------|------|------|
| **kubelet** | 车间主任 | 听Master的命令，在本机启动/停止容器 |
| **kube-proxy** | 网络管家 | 管理容器的网络通信、负载均衡 |
| **Docker/容器运行时** | 厨师 | 实际运行容器的地方 |

> **注意**：Worker节点上装了Docker，K8s通过kubelet来控制Docker。

### 2.4 面试问答

**Q：K8s集群由什么组成？**
A：由Master节点和多个Worker节点组成。Master负责管理和调度（包括API Server接收命令、Scheduler决定容器跑在哪、Controller Manager监控状态、etcd存储配置），Worker节点负责实际运行容器（kubelet管理容器、kube-proxy管网络、Docker跑容器）。

---

## 第三章：核心概念（面试必问）

### 3.1 Pod（豆荚）—— 最小单位

| 项目 | 说明 |
|------|------|
| 是什么 | K8s中最小的部署单位 |
| 比喻 | 一个Pod = 一个豆荚，里面可以有一个或多个豆子（容器） |
| 和Docker的关系 | Pod里面装的是Docker容器 |
| 特点 | Pod是临时的，挂了就没了，IP会变 |

**为什么要有Pod？**
- Docker容器太小了，有时候一个应用需要多个容器紧挨着（共享网络、存储）
- Pod把1个或多个容器打包在一起，作为一个整体管理

```
Pod（豆荚）
├── 容器1（主容器，比如Nginx）
└── 容器2（Sidecar边车容器，比如日志收集）
```

> **生活比喻**：Pod就像一个外卖盒，里面可以放一个菜，也可以放主食+汤+小菜。它们共享一个外卖盒（共享网络和存储）。

### 3.2 Deployment（部署）—— 管理Pod的

| 项目 | 说明 |
|------|------|
| 是什么 | 管理Pod的控制器，决定跑几个Pod、怎么升级 |
| 比喻 | Pod是士兵，Deployment是连长，管着几个士兵 |
| 作用 | 维持指定数量的Pod运行，挂了自动补，升级时滚动更新 |

**为什么需要Deployment？**
- Pod是临时的，挂了就没了
- Deployment盯着Pod数量，比如设置"我要3个Pod"，少了就自动补，多了就删

```
Deployment（连长）
├── Pod1（士兵1）
├── Pod2（士兵2）
└── Pod3（士兵3）
```

> **生活比喻**：Deployment就像一个包工头，你说"我要3个工人搬砖"，他就保证随时有3个工人在搬，有人请假了自动叫新人来顶。

### 3.3 Service（服务）—— 给Pod一个固定入口

| 项目 | 说明 |
|------|------|
| 是什么 | 为Pod提供固定的访问入口（固定IP和域名） |
| 比喻 | Pod是流动摊贩（位置老变），Service是固定店面（地址不变） |
| 作用 | Pod的IP会变，Service给一个固定IP，流量自动转发到后面的Pod |

**为什么需要Service？**
- Pod挂了重建后IP会变
- 外部访问需要固定地址
- Service做负载均衡，把流量分发给多个Pod

```
用户 → Service（固定IP 10.0.0.5） → Pod1（192.168.1.1）
                                  → Pod2（192.168.1.2）
                                  → Pod3（192.168.1.3）
```

> **生活比喻**：Pod是外卖骑手（人经常换），Service是外卖平台的客服电话（号码不变）。你打客服电话，平台自动分配一个骑手给你。

### 3.4 三者关系图

```
Deployment（管理者）
    │ 管理
    ▼
  Pod（运行容器） ← IP会变
    │ 被访问
    ▼
Service（固定入口） ← 提供固定IP + 负载均衡
```

> **记忆口诀**：Deployment管Pod数量，Pod跑容器，Service给固定入口。

### 3.5 Namespace（命名空间）—— 资源隔离

| 项目 | 说明 |
|------|------|
| 是什么 | 把集群资源划分成多个虚拟集群 |
| 比喻 | 一栋办公楼分成不同楼层（开发部在3楼，测试部在4楼） |
| 作用 | 资源隔离，不同团队/环境互不影响 |

> **默认命名空间**：default。生产环境一般分 dev（开发）/test（测试）/prod（生产）。

### 3.6 面试问答

**Q：说说K8s的核心概念？**
A：核心有三个：
1. **Pod**：最小部署单位，里面装Docker容器
2. **Deployment**：管理Pod的控制器，保证指定数量的Pod运行，支持滚动更新
3. **Service**：为Pod提供固定访问入口和负载均衡，解决Pod IP变化的问题

**Q：为什么不直接用Pod，还要套一层Deployment？**
A：因为Pod是临时的，挂了就没了，IP会变。Deployment能自动维持Pod数量（自愈），还能做滚动更新（零停机升级）。相当于给Pod加了一个"管家"。

**Q：Service和Pod什么关系？**
A：Service是Pod的"门面"。Pod的IP不稳定，Service提供固定IP，外部访问Service，Service做负载均衡把流量转发给后面的Pod。类似Nginx反向代理后面的Web服务器。

---

## 第四章：kubectl常用命令

kubectl就是K8s的命令行工具，类似Docker的docker命令。

### 4.1 常用命令速查表

| 命令 | 作用 | 对应Docker命令 |
|------|------|---------------|
| `kubectl get pods` | 查看所有Pod | docker ps |
| `kubectl get deployments` | 查看所有Deployment | - |
| `kubectl get services` | 查看所有Service | - |
| `kubectl get ns` | 查看所有命名空间 | - |
| `kubectl describe pod xxx` | 查看Pod详细信息 | docker inspect |
| `kubectl logs xxx` | 查看Pod日志 | docker logs |
| `kubectl exec -it xxx -- bash` | 进入Pod容器 | docker exec |
| `kubectl delete pod xxx` | 删除Pod | docker rm |
| `kubectl apply -f xxx.yaml` | 根据yaml文件部署 | docker-compose up |
| `kubectl scale deployment xxx --replicas=5` | 扩容到5个Pod | - |

> **记忆口诀**：get看列表，describe看详情，logs看日志，exec进容器，apply部署yaml。

### 4.2 和Docker命令对比

| 操作 | Docker | K8s(kubectl) |
|------|--------|-------------|
| 看运行中的容器 | docker ps | kubectl get pods |
| 看容器详情 | docker inspect | kubectl describe pod |
| 看容器日志 | docker logs | kubectl logs |
| 进入容器 | docker exec -it | kubectl exec -it |
| 删除容器 | docker rm | kubectl delete pod |
| 部署 | docker run / compose up | kubectl apply -f |

---

## 第五章：YAML部署文件

K8s用YAML文件来定义资源，类似Docker Compose的docker-compose.yml。

### 5.1 一个完整的Deployment YAML示例

```yaml
apiVersion: apps/v1          # API版本
kind: Deployment             # 资源类型：Deployment
metadata:
  name: nginx-deployment     # Deployment名称
  namespace: default         # 命名空间
spec:
  replicas: 3                # 要跑3个Pod
  selector:
    matchLabels:
      app: nginx             # 管理标签为app=nginx的Pod
  template:                  # Pod模板
    metadata:
      labels:
        app: nginx           # Pod的标签
    spec:
      containers:
      - name: nginx          # 容器名
        image: nginx:1.20    # 用的Docker镜像
        ports:
        - containerPort: 80  # 容器端口
```

### 5.2 对应的Service YAML示例

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort             # 服务类型
  selector:
    app: nginx               # 关联标签为app=nginx的Pod
  ports:
  - port: 80                  # Service端口
    targetPort: 80            # Pod端口
    nodePort: 30080           # 外部访问端口
```

### 5.3 YAML结构理解

```
Deployment YAML
├── apiVersion: 用哪个API版本
├── kind: 资源类型（Deployment/Service/Pod等）
├── metadata: 元数据（名字、命名空间、标签）
└── spec: 具体配置
    ├── replicas: 要跑几个副本
    ├── selector: 管理哪些Pod（通过标签匹配）
    └── template: Pod的模板
        └── containers: 容器配置（镜像、端口等）
```

> **和Docker Compose对比**：K8s YAML比docker-compose.yml更复杂，但原理类似——都是用声明式文件描述"我要什么状态"。

---

## 第六章：K8s和已学知识的联系

### 6.1 K8s中的概念对应已学知识

| K8s概念 | 对应已学知识 | 说明 |
|---------|------------|------|
| Pod | Docker容器 | Pod里面跑的就是Docker容器 |
| Deployment | docker-compose的replicas | 管理多个副本 |
| Service | Nginx反向代理+负载均衡 | Service就是K8s内部的负载均衡 |
| Node | 虚拟机/物理机 | Worker节点就是一台服务器 |
| Namespace | Linux用户隔离 | 不同命名空间资源隔离 |
| kubectl | docker命令 | 管理K8s的命令行工具 |
| YAML文件 | docker-compose.yml | 声明式配置文件 |

### 6.2 Service和Nginx负载均衡对比

```
Nginx负载均衡（已学）：
用户 → Nginx(192.168.121.101) → Web1 / Web2 / Web3

K8s Service（新学）：
用户 → Service(固定IP) → Pod1 / Pod2 / Pod3
```

**几乎一模一样！** Service本质就是K8s内部的Nginx负载均衡。

---

## 第七章：面试速答卡片

### Q1：K8s是什么？
A：K8s（Kubernetes）是容器编排工具，用于自动化部署、扩缩容和管理Docker容器集群。它解决了多台机器上管理大量容器的问题。

### Q2：K8s集群架构？
A：Master节点 + Worker节点。Master负责管理（API Server/Scheduler/Controller/etcd），Worker负责运行容器（kubelet/kube-proxy/Docker）。

### Q3：Pod是什么？
A：Pod是K8s最小的部署单位，里面包含一个或多个Docker容器。同一个Pod内的容器共享网络和存储。Pod是临时的，IP会变。

### Q4：Deployment是什么？
A：Deployment是Pod的控制器，保证指定数量的Pod运行。Pod挂了会自动重启，支持滚动更新和回滚。

### Q5：Service是什么？为什么需要它？
A：Service为Pod提供固定的访问入口。因为Pod的IP会变，Service提供固定IP和负载均衡，外部访问Service，它把流量转发给后面的Pod。类似Nginx反向代理。

### Q6：kubectl常用命令？
A：kubectl get pods（查看Pod）、kubectl describe pod（查看详情）、kubectl logs（查看日志）、kubectl exec（进入容器）、kubectl apply -f（部署yaml）。

### Q7：K8s和Docker什么关系？
A：Docker负责打包和运行单个容器，K8s负责在集群层面管理大量Docker容器。K8s的Pod里面跑的就是Docker容器。Docker是基础，K8s是上层管理。

### Q8：K8s和Docker Compose什么区别？
A：Docker Compose管理单机上的多个容器，K8s管理多台机器上的大量容器。Compose没有自动调度、故障恢复、自动扩缩容功能，K8s有。

---

## 第八章：K8s学习结论

### 面试怎么说

> "我了解K8s的核心概念。K8s是容器编排工具，集群由Master和Worker节点组成。核心概念有Pod（最小部署单位）、Deployment（管理Pod的控制器，保证Pod数量和滚动更新）、Service（为Pod提供固定访问入口和负载均衡）。我用过kubectl常用命令，能编写简单的YAML部署文件。"

### 不需要深入的

- ❌ K8s集群搭建（minikube/kubeadm）——虚拟机跑不动
- ❌ 网络插件（Calico/Flannel）——太底层
- ❌ 存储卷（PV/PVC/StorageClass）——太复杂
- ❌ Helm包管理 —— 暂时不需要
- ❌ Operator/CRD —— 高级内容
- ❌ Service Mesh（Istio）—— 太高级

> **一句话**：知道Pod/Deployment/Service是什么，会kubectl基本命令，面试能说出来就行。入职后公司用什么学什么。
