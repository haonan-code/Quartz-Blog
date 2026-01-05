> [【GeekHour】30分钟Docker入门教程\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV14s4y1i7Vf?vd_source=011520b0b897ef5713a0e46977614dd7&spm_id_from=333.788.videopod.sections)

## 1. 简介
### 1.1 Docker 简介
Docker 是一个用于**构建（build）**、**运行（run）**、**传送（share）** 应用程序的平台

### 1.2 为什么要使用 Docker
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250726162308942.png)
若使用传统的开发部署方式，需要配置开发环境、测试环境、生产环境等三个地方的环境，过程重复且繁琐。而如果使用 Docker 技术，即可将每个部分打包成单独的集装箱汇总起来，只要在开发环境中运行成功，即可在其他地方迁移使用。

### 1.3 Docker 和虚拟机的区别
#### 1.3.1 虚拟机
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250726162944020.png)

#### 1.3.2 Docker
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250726163022234.png)

## 2. 基本概念
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250726164409793.png)

**镜像**：是一个**只读的模板**，可以用来创建容器。
**容器**：是 docker 的**运行实例**，它提供了一个独立的可移植的环境，可以在这个环境中运行应用程序。
**仓库**：用来存储 docker 镜像的地方。常用的 dockerhub 就用来集中存储和管理 docker 镜像，从而实现镜像的共享和复用。

**Docker 使用 Client-Server 架构模式**：
Docker Client 和 Docker Daemon 之间通过 Socket 或者 RESTful API 进行通信。Docker Daemon 就是服务端的守护进程，它负责管理 Docker 的各种资源，Docker Client 负责向 Docker Daemon 发送请求，Docker Daemon 接收到请求之后进行处理，然后将结果返回给 Docker Client。Docker Daemon 为一个后台进程，用来接受并处理来自 Docker 客户端的请求，然后将结果返回给客户端。所以我们在终端中输入的各种 Docker 命令，实际上都是通过 Docker 客户端发送给 Docker Daemon 的，然后 Docker Daemon 在进行处理，最后再将结果返回给客户端，然后就可以在终端中看到执行结果。

## 3. 安装配置
直接官网下载安装

## 4. 常用命令
[[DockerCheatSheet-ByGeekHour.pdf]]

## 5. 构建镜像
### 5.1 容器化和 Dockerfile
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250727112358737.png)

Dockerfile：是一个文本文件，里面包含了一条条指令，用来告诉 docker 如何构建镜像。镜像中包含了应用程序执行的所有命令（各种依赖、配置环境和运行应用程序所需要的所有内容）。

主要包含以下内容：
- **精简版的操作系统**：Alpine
- **应用程序的运行时环境**：NodeJS、Java、Python
- **应用程序**：SpringBoot 打包好的 jar 包
- **应用程序的第三方依赖库或者包**：
- **应用程序的配置文件、环境变量**：

### 5.2 实践
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250727114831550.png)
示例：
**Dockerfile 文件编写**：
```Dockerfile
# 指定基础镜像
# 直接使用nodejs镜像（已基于Linux构建，alpine说明是基于alpine操作系统构建）
FROM node:14-alpine 

# 拷贝本地文件至容器中 
# COPY 源路径 目标路径
# 源路径：Dockerfile文件所在路径；
# 目标路径：相对于镜像的路径
COPY index.js /index.js

# 运行应用程序
# CMD ["node", "/index.js"]
CMD node /index.js
```
**构建**：
`docker build -t hello-docker .`
**运行**：
`docker run hello-docker`
**查看镜像**：
`docker images` 或
`docker image -ls`

## 6. 运行容器
`docker run hello-docker`

## 7. Docker Compose & Kubernetes
### 7.1 Docker Compose
- 用于定义和运行多容器 Docker 应用程序的工具
- 使用 YAML 文件来配置应用程序的服务
- 一条命令即可创建并启动所有服务

通过一个单独的 docker-compose.yaml 的配置文件，来将一组关联的容器组合在一起，形成一个项目，然后通过一条命令就可以启动、停止或者重建这些服务。

![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250728121613152.png)

命令：
`docker compose up`

### 7.2 Dockerfile & Docker Compose 区别

| 特性       | Dockerfile         | docker-compose.yml                         |
| -------- | ------------------ | ------------------------------------------ |
| **解决问题** | **构建单个** Docker 镜像 | **编排和运行多个**相互关联的 Docker 容器服务               |
| **作用范围** | 单个容器的构建指令          | 整个应用栈中所有服务的定义和关系                           |
| **文件格式** | 文本文件，一系列 Docker 命令 | YAML 格式                                    |
| **主要命令** | `docker build`     | `docker-compose up`, `docker-compose down` |
| **粒度**   | **容器镜像**级别         | **应用服务**级别                                 |
| **关注点**  | 如何将应用程序打包成可运行的容器   | 如何让多个服务协同工作，构成完整的应用程序                      |
| **生命周期** | 描述镜像的构建过程          | 描述如何启动、停止、链接和管理整个应用的服务集合                   |

