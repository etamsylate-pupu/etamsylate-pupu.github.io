---
title: 项目开发-Jenkins篇
tags: [Jenkins]
date: 2023-10-18 11:46:52
categories: technique
urlname: 23
---


Jenkins是一款开源 CI&CD 软件，用于自动化各种任务，包括构建、测试和部署软件。其支持各种运行方式，可通过系统包、Docker 或者通过一个独立的 Java 程序。

### Jenkins节点

Jenkins中的节点是用于执行构建任务的计算机或计算机集群，可以通过配置界面、CLI或API进行配置和管理，可以分为主节点和代理节点，用于实现任务执行的并行化、负载均衡和资源隔离。

通常在安装Jenkins这台服务器默认就是一个主节点（俗称master），其他相对于这台安装Jenkins的机器都称为从节点或代理节点（俗称slaves, agent）。

- 主节点是Jenkins的核心节点，负责管理整个Jenkins系统的配置和任务分发。主节点可以执行一部分构建任务，但通常不建议在主节点上执行耗时较长或资源占用较高的任务，以免影响Jenkins的整体性能。

- 代理节点是由主节点管理的其他计算机或计算机集群。代理节点可以执行构建任务，并将结果返回给主节点。代理节点可以根据需要添加多个，以提供更多的计算资源和并行执行能力。

不同的Jenkins任务有不同的操作环境需求，比如部署基于IIS服务的需要windows操作系统，构建IOS应用需要MacOs，构建脚本是shell的需要Linux操作系统。

在 Jenkins 中添加一个 SSH 节点（称为 SSH Slave 或 SSH Agent），需要通过 SSH 连接到远程主机，并让 Jenkins 控制该节点上的构建任务。以下是具体的步骤：

#### 节点配置（Launch agents via SSH）-环境准备

- Jenkins 已安装并运行。
- 安装SSH Build Agents, Credentials插件。
- 远程主机的 SSH 访问权限，并且该主机上安装了 Java。

   ```
   #安装OpenJDK 11
   sudo apt update
   sudo apt install openjdk-11-jdk
   #检查是否安装成功
   java -version
   ```

- 远程主机的 IP 地址或主机名、SSH 端口号、用户名和密码（用户名和密码认证）或私钥（密钥认证）。


#### 节点配置（Launch agents via SSH）-添加节点

首先在节点管理处新增“固定节点”

![node entry][5]

![new node][6]

根据信息配置节点，填写远程主机信息以及SSH凭据。选项说明如下：

- Remote root directory: 输入 Jenkins 代理将运行工作的远程主机上的目录路径。
- Labels: 输入该节点的标签（可选，但有助于在构建时指定节点）。
- Usage: 通常选择 "Only build jobs with label expressions matching this node"。
- Launch method: 选择 "Launch agents via SSH"。
- Host: 输入远程主机的 IP 地址或主机名。
- Credentials: 点击 "Add" 按钮，然后在弹出的对话框中添加 SSH 凭据（用户名和密码或用户名和私钥）。
  - 如果使用用户名和私钥，点击 "Kind" 下拉菜单选择 "SSH Username with private key"。
  - 在 "ID" 栏输入一个识别名。
  - 在 "Username" 栏输入远程主机的用户名。
  - 在 "Private Key" 部分，输入私钥或者上传私钥文件。
- Host Key Verification Strategy: 根据安全需求选择一个选项，如果不介意安全风险，可以选择 "Non verifying Verification Strategy"。

![node configure][7]

![node configure][8]

![node configure][9]


#### 节点配置（Launch agents via SSH）-启动节点

通过点击节点旁边的 Launch agent 来启动。如果配置正确，Jenkins 将通过 SSH 连接到远程节点并启动代理。点击状态可查看启动信息，若启动失败，可根据日志中的报错信息修改。

![node launch][10]

### Jenkins任务

Jenkins中的任务（Job）是指要执行的特定操作或一系列操作的定义。任务通过配置和设置来定义其行为和执行方式，包括触发器、构建步骤、构建参数、构建环境等。通过任务的配置和管理，可以实现自动化的构建、测试和部署流程。

Jenkins Job中的一些概念如下：

- 任务类型：Jenkins支持多种任务类型，例如自由风格项目、流水线项目、多配置项目等。每种任务类型都有不同的配置选项和执行方式。

- 构建触发器：任务可以通过不同的触发器来触发构建操作。常见的触发器包括定时触发、版本控制系统的变更触发、其他任务的成功触发等。

- 构建步骤：任务可以定义一系列的构建步骤，每个步骤执行特定的操作。例如，可以包括代码拉取、编译、测试、部署等步骤。

- 构建参数：任务可以定义输入参数，允许用户在执行任务时提供参数值。参数可以是文本、下拉列表、布尔值等类型。

- 构建环境：任务可以定义构建环境，包括设置环境变量、配置工具路径、指定构建代理节点等。

- 构建历史和报告：Jenkins会记录每次任务的构建历史，包括构建状态、执行时间、控制台输出等。任务还可以生成构建报告，用于查看构建结果和分析构建过程。

- 插件扩展：Jenkins提供了丰富的插件生态系统，可以扩展任务的功能和特性。通过安装和配置插件，可以实现更多的自定义和集成。

#### 创建任务

以创建流水线任务为例，任务涉及通用、高级项目选项和流水线。通用配置包含上述提及的构建触发器，添加扩展后的扩展选项等。

针对推送至仓库后的构建，以gitlab webhook（[webhook介绍][15]）为例，需要先安装“Gitlab”插件，接着任务配置中，构建触发器选择“Build when a change is pushed to Gitlab”，之后在gitlab对应的项目中，选择设置中的webhooks，将前一步的GitLab webhook URL和高级中的Secret Token填入，出发事件根据自己的需求设置，例如，当代码推送至dev分支时触发，则勾选"Push events-wildcard pattern"，填写分支名称"dev"。最后，点击测试中的对应事件（push events），若返回200，则说明配置成功。

![Jenkins job][11]
![Job configure][12]
![Job configure][13]
![gitlab webhooks][14]

流水线是项目运行需要执行的一系列操作，具体见下小节。

### Jenkins Pipeline

Jenkins Pipeline（或简称为 "Pipeline"）是一套插件，将持续交付的实现和实施集成到 Jenkins 中。Pipelines 由多个步骤（step）组成，允许构建、测试和部署应用。当一个步骤运行成功时继续运行下一个步骤。 当任何一个步骤执行失败时，Pipeline 的执行结果也为失败。

在 Linux、BSD 和 Mac OS（类 Unix ）系统中的 shell 命令， 对应于 Pipeline 中的一个 sh 步骤（step）。

以下是简单的hello word脚本。

```Pipeline script
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo "Hello World"'
                sh '''
                    echo "Multiline shell steps works too"
                    ls -lah
                '''
            }
        }
    }
}

```

- pipeline 定义一个流水线脚本

- agent 指示 Jenkins 为整个流水线分配一个执行器（在 Jenkins 环境中的任何可用代理/节点上）和工作区。

- stages 全部的工作都在这里执行

- stage 每个工作开始

- steps jenkinsfile 声明式脚本往这里面写

- echo 写一个简单的字符串到控制台输出。


以构建项目docker镜像为例，用Pipleine script编写部署脚本。项目根据镜像文件Dockerfile构建，拉取代码等一系列命令都在Dockerfile中。因此，此时项目运行的步骤包含：切换至镜像目录→暂停镜像→构建镜像→启动镜像→删除多余的镜像（避免镜像堆积导致内存占用）。

此时命令执行的权限与节点认证时用户的权限有关，涉及sudo权限的，需要给当前用户赋予sudo权限，并需要设置为不需要输入密码（在/etc/sudoer文件中增加`Usename ALL=NOPASSWD: ALL`）。

```
pipeline {
    agent {
     node {
            label '代理节点标签'
        }
    }

    stages {
        stage('Deploy') {
            steps {
                script {

                    try {
                        sh 'cd 镜像文件所在目录 && sudo docker-compose down && sudo docker-compose build --no-cache  && sudo docker-compose up -d'
                        sh 'sudo docker image prune -f -a --filter \'until=24h\''
                        echo '部署完成'
                    } catch (Exception e) {
                        error "Command failed with exception: ${e}"
                    }
                }
            }
        }
    }
}


```


参考资料：

[Jenkins官方英文文档][1]

[Jenkins官方中文文档][2]

[Jenkins节点 node、凭据 credentials、任务 job][3]

[jenkins添加ssh节点（SSH Server）教程][4]







[1]: https://www.jenkins.io/doc/
[2]: https://www.jenkins.io/zh/doc/
[3]: https://blog.csdn.net/wlddhj/article/details/134990223
[4]: https://www.teambition.com/community/1917.html
[5]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/node.png
[6]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/new_node.png
[7]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/node_config1.png
[8]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/node_config2.png
[9]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/node_config3.png
[10]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/node_launch.png
[11]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/jenkins_job.png
[12]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/job_config.png
[13]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/job_config2.png
[14]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/develop/jenkins/gitlab_hook.png
[15]: https://zhuanlan.zhihu.com/p/133449879