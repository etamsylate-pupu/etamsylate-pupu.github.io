---
title: docker私有库搭建
tags: [docker]
date: 2024-03-19 14:57:26
categories: technique
urlname: 28
---



Docker官方维护了一个公共的镜像仓库Docker Hub，用户可以在其中找到各种镜像并进行下载使用。执行 docker pull 命令时，默认情况下会从 Docker Hub 上拉取镜像。除了 Docker Hub，用户也可以配置 Docker 从其他的镜像仓库拉取镜像，比如阿里云镜像仓库、腾讯云镜像仓库等。在执行 docker pull 命令时，可以通过在镜像名称前加上镜像仓库地址来指定从其他仓库拉取镜像。

Docker Hub虽然方便,但是有以下限制
- 需要internet连接,速度慢
- 所有人都可以访问
- 由于安全原因企业不允许将镜像放到外网

除了公开镜像仓库之外，Docker也提供名称为 registry 的镜像用于搭建本地私有仓库。在内部网络搭建的 Docker 私有仓库可以使内网人员下载、上传都非常快速，不受外网带宽等因素的影响，同时不在内网的人员也无法下载我们的镜像，并且私有仓库也支持配置仓库认证功能。因此，用户可以借此搭建自己的私有镜像仓库来存储和管理自己的镜像。


### 配置私有仓库(无认证)

#### 拉取私有仓库镜像

```
docker pull registry
```

#### 设置私有仓库参数

由于docker的私服库作了安全加固，通常不支持http的推送，因此需要配置支持本地私有库registry的http连接。

docker安装后默认没有daemon.json这个配置文件，需要进行手动创建，配置文件的默认路径：/etc/docker/daemon.json。

在daemon.json添加私有库地址信息，**让 Docker 信任私有仓库地址**。

```
vim /etc/docker/daemon.json
```

```json
{
       "insecure-registries":["主机IP:5000"]
}
```

修改参数后，重新加载配置信息及重启 Docker 服务。

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 运行registry容器

拉取了registry镜像后，需要运行该镜像，构建本地私有服务器容器，此后本地镜像将会推送至该容器。

仓库默认创建在容器的/var/lib/registry目录下，即推送至registry私有镜像仓库的文件会存储在/var/lib/registry目录下。考虑到权限管理问题，在实际应用中通常需要自定义容器卷映射（将容器目录挂载到宿主机目录），以便与宿主机联调。

```
docker run -id --name registry -p 5000:5000 -v /root/mydate/docker_registry:/var/lib/registry registry

#查看运行的容器
docker ps
```

- -v ：添加数据卷，格式为：-v /宿主机目录:/容器内目录；`-v /root/mydate/docker_registry:/var/lib/registry` 将容器内 /var/lib/registry 目录下的数据挂载至宿主机 /root/mydate/docker_registry目录下
- -p ：映射端口，指定容器端口绑定到主机相应主机端口；默认情况下Docker开放了5000端口，-p 5000:5000 将Docker内部默认端口映射到主机端口5000
- --name ：容器命名，可自定义任何名称
- -id : 以交互模式运行容器，并在后台运行

容器运行后，可以进入registry容器查看容器目录下是否创建成功

```
#查看运行的容器，查看registry的容器ID以及此时COMMAND，确认/bin/bash, bin/sh, bash, sh是否支持。
docker ps

#进入容器
docker exec -it 容器id /bin/sh
ls /var/lib/
```

也可以查看私有仓库镜像文件，在终端输入:

```
curl http://主机ip:5000/v2/_catalog
```

或者同一内网浏览器访问 http://主机ip:5000/v2/_catalog ，看到`{"repositories":[]}` 表示私有仓库搭建成功并且内容为空。

#### 本地镜像构建

准备需要推送至私有镜像仓库的镜像，以构建本地项目镜像为例。

根据Dockerfile构建镜像

```
docker build -t local-image:tag

# docker build -t lab_dev:1.0
```

#### 推送本地镜像至私有库

先给镜像设置标签，再将镜像推送至私有仓库

```
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname

#docker tag lab_dev:1.0 主机ip:5000/lab_dev:1.0
#docker push 主机ip:5000/lab_dev:1.0
```

若推送成功，则访问私有镜像仓库（如上终端输入命令或浏览器输入地址），可以查看到刚刚的镜像信息`{"repositories":["lab_dev"]} ` 






参考资料：

[docker私有镜像仓库的搭建及认证][1]

[Docker入门：私有库（Docker Registry）简介及使用方法（防踩坑）][2]

[1]: https://developer.aliyun.com/article/1416683
[2]: https://blog.csdn.net/weixin_37926734/article/details/123279987