# 运维实战档案（DevOps 交付平台 + WAF 安全防护）

应届运维工程师的项目实战记录，全部内容来自真实搭建过程留档。

## 项目一：云原生 DevOps 一体化交付平台

K8s + Calico + Harbor + Gitea + Jenkins + ArgoCD + Helm 完整 GitOps 链路。
集群为按量付费服务器搭建（已释放），以下为搭建时的真实截图与完整文档。

### 运行证据截图

| 截图 | 说明 |
|------|------|
| ![K8s节点](screenshots/01_K8s集群节点Ready.png) | 集群节点全部 Ready |
| ![Pod](screenshots/02_K8s全部Pod运行正常.png) | 全部 Pod 运行正常 |
| ![devops命名空间](screenshots/03_devops命名空间_Gitea_Jenkins.png) | Gitea + Jenkins 部署 |
| ![argocd](screenshots/04_argocd命名空间Pod.png) | ArgoCD 命名空间 |
| ![harbor](screenshots/05_harbor命名空间Pod.png) | Harbor 命名空间 |
| ![ArgoCD界面](screenshots/06_ArgoCD界面_Healthy_Synced.png) | ArgoCD Healthy/Synced |
| ![Harbor仓库](screenshots/07_Harbor镜像仓库_devops项目.png) | Harbor 镜像仓库 |
| ![demo](screenshots/08_demo应用页面_HelloDevOps_v2.png) | demo 应用发布效果 |

### 文档

- [搭建实录](docs/devops/01-搭建实录.md)
- [复盘讲解与自测](docs/devops/02-复盘讲解与自测.md)
- [项目完全掌握手册](docs/devops/03-项目完全掌握手册.md)
- [技术教程](docs/devops/04-技术教程.md)
- [防问倒手册](docs/devops/05-防问倒手册.md)
- [质疑点应对](docs/devops/06-质疑点应对.md)
- [Jenkins 本地 CI 流水线搭建实录](docs/devops/07-Jenkins本地CI流水线搭建实录.md)（服务器释放后在本地 VM 重建 CI 闭环：提交代码 → 自动构建 → 自动部署）
- [Jenkins 与 CI/CD 原理学习笔记](docs/devops/08-Jenkins与CICD原理学习笔记.md)（CI/CD 概念、Master/Agent 架构、Pipeline 原理、触发机制、排障复盘）

## 项目二：企业级 Web 应用架构与安全防护（雷池 WAF）

阿里云 ECS + 雷池 WAF + Nginx + Docker，含高可用验证与排障记录。

- [实操记录与原理笔记](docs/waf/01-实操记录与原理笔记.md)
- [雷池 WAF 部署教程](docs/waf/02-雷池WAF部署教程.md)

## 学习笔记

Docker / K8s / Nginx / MySQL / Redis 系统学习文档见 [docs/notes/](docs/notes/)。
