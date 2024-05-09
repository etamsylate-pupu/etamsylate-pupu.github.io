---
title: docker私有库搭建
tags: [docker]
date: 2024-03-19 14:57:26
categories: technique
urlname: 28
---



Docker官方维护了一个公共的镜像仓库Docker Hub，用户可以在其中找到各种镜像并进行下载使用。执行 docker pull 命令时，默认情况下会从 Docker Hub 上拉取镜像。除了 Docker Hub，用户也可以配置 Docker 从其他的镜像仓库拉取镜像，比如阿里云镜像仓库、腾讯云镜像仓库等。在执行 docker pull 命令时，可以通过在镜像名称前加上镜像仓库地址来指定从其他仓库拉取镜像。

除了公开镜像仓库之外，Docker也提供私有镜像仓库。用户可以搭建自己的私有镜像仓库来存储和管理自己的镜像


本地镜像构建

推送本地镜像至私有库（需要先配置registry，并让Docker信任当前私有仓库地址）















[1]: https://developer.aliyun.com/article/1416683