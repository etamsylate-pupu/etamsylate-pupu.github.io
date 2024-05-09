---
title: 项目开发-docker篇
tags: [docker]
date: 2023-08-09 17:46:52
categories: technique
urlname: 20
---

应用程序通过隔离实现在运行时相互独立互不干扰，目前实现隔离的方式包括：虚拟机、容器。
与虚拟机通过操作系统实现隔离不同，容器技术只隔离应用程序的运行时环境，而容器之间可以共享同一个操作系统。在这里，运行时环境指的是程序运行所需的各种库和配置。


Docker 是一个开源的容器化平台，用于构建、部署和运行应用程序。它可以让开发者打包应用以及依赖项到一个轻量级、可移植的容器中，该容器可以在任何支持Docker的环境中运行。（[docker官方文档地址][1]）

Docker 使用的架构是 C/S 架构，包括 Docker 客户端和 Docker 服务器。Docker 客户端通过 RESTful API 与 Docker 服务器通信，Docker 服务器负责管理 Docker 镜像、容器等资源。

Docker解决了以下问题：

- 环境不一致
- 隔离性
- 弹性伸缩

### Docker安装

官方地址：[docs.docker.com/get-docker][2] 

检查安装版本：`docker --version`

### Docker基本概念
- Dokerfile
  > Docker builds images by reading the instructions from a Dockerfile. A Dockerfile is a text file containing instructions for building your source code. 
  > 用来构建 Docker 镜像的文本文件，包含了一系列指令，用于描述如何构建镜像。通过 Dockerfile，用户可以定义镜像的基础操作系统、安装依赖、配置环境变量等步骤，以及容器启动时需要执行的命令
- Docker Images
  > Docker images consist of layers. Each layer is the result of a build instruction in the Dockerfile. Layers are stacked sequentially, and each one is a delta representing the changes applied to the previous layer.
  > 是一个只读的模板，包含了运行容器所需的文件系统、应用程序和配置等信息。可以通过 Dockerfile 来构建 Docker 镜像，也可以从 Docker 仓库中拉取现有的镜像。
- Docker Container
  > A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.
  > 是 Docker 镜像的可运行实例，类似于一个轻量级、独立的虚拟机。每个容器都运行在独立的环境中，包含自己的文件系统、进程空间等，但与宿主机共享内核。
- Docker Compose
  > Docker Compose is a tool for defining and running multi-container applications. It is the key to unlocking a streamlined and efficient development and deployment experience.
  > 用于定义和运行多容器 Docker 应用的工具。通过一个单独的 YAML 文件（通常命名为 docker-compose.yml），用户可以定义多个容器的配置，包括镜像、环境变量、网络设置等，然后使用一个命令就可以启动、停止或删除整个应用。
- Docker Registry
  > Docker registry stores Docker images. Docker Hub is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default. You can also run your own private registry.
  > 用于存储 Docker 镜像，可以分为公共仓库（如 Docker Hub）和私有仓库。用户可以将自己构建的 Docker 镜像推送到仓库中，也可以从仓库中拉取他人分享的镜像。
  


docker启动/停止命令：
```
systemctl stop docker
systemctl start docker
systemctl restart docker
```
docker状态查看：`systemctl status docker`
docker信息查看：`docker info`
docker镜像查看：`docker images`
docker镜像删除：`docker rmi 镜像标识`
docker容器查看：`docker ps -a`

Dockerfile编写
```
FROM：指定基础镜像名称和版本，将打包的项目在该基础镜像上运行
LABEL：为镜像添加元数据，可以用于标识镜像的作者、版本、描述等信息（可忽略）
ENV：设置环境变量，可以在容器内部使用
WORKDIR：设置工作目录，用于指定容器内部的工作目录，后续的命令都将在该目录下执行
COPY：从本地复制文件至创建的镜像文件
RUN: 对创建的镜像使用的命令
CMD: 容器被创建启动时执行的命令
```



[1]: https://docs.docker.com/
[2]: https://docs.docker.com/get-docker/