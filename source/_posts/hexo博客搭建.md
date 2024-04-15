---
title: hexo+github+material博客搭建
date: 2023-04-19 13:59:26
categories: technique
tags: [hexo]
urlname: 17
---

此前是使用typecho模板+阿里云服务器搭建的博客，由于某次忘记续费导致网站关闭，临时联系客服续费备份了网站和数据库文件。

考虑到后续的一些开销，于是转战github网站免费托管。而typecho是动态渲染的，不适用github的托管服务。因此网站框架也改用为hexo。
>   hexo 是一个静态博客生成工具，具备编译 Markdown、拼接主题模板、生成 HTML、上传 Git 或 FTP 等基本功能。

建站是主要参考了hexo官方文档[参考链接][1]和U2647's blog（[参考链接][2]）

## 环境准备
- Nodejs [下载地址][3]
  
  Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本

  检查是否安装成功
  ```
  PS C:\Windows\System32> node -v
  v16.14.2
  PS C:\Windows\System32> npm -v
  8.5.0
  ```

- Git [下载地址][4]
  检查是否安装成功
  ```
  PS C:\Windows\System32> git --version
  git version 2.35.1.windows.
  ```

## 博客搭建

### hexo部署
建议首先阅读hexo官方文档
- 安装hexo
  
  将hexo的功能封装为命令，提供给用户通过 hexo server / hexo deploy 等命令调用的模块，就是 hexo-cli（hexo-Command Line Interface）。

  终端输入：

  `npm install -g hexo-cli`
- 博客初始化
  
  新建一个用于存放hexo博客的空文件夹，进入该文件夹，终端输入：

  `hexo init`
  
  该命令执行相当于
  - Git clone hexo-starter 和 hexo-theme-landscape 主题到当前目录（指定文件夹时，则到指定目录）。
  - 下载依赖。

  此时，博客文件夹的文件结构如下：
  ```
  . 
  ├── _config.yml           网站的配置信息
  ├── package.json          应用程序的信息
  ├── scaffolds             文章模版
  ├── source                用户资源
  |     └── _posts          *
  ├── themes                主题     
  ├── node_modules   
  ├── .github
  ├── .gitignore
  ├── _config.landscape.yml  
  └──  yarn.lock    
  ```
  > *除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。

  接着，生成静态文件：
  
  `hexo generate`  缩写：`hexo g`

  然后，启动服务器（默认情况下，访问网址为： http://localhost:4000/ ）：

  `hexo server`    缩写：`hexo s` 

  访问网站查看博客部署情况，页面显示如下，则成功：
  ![hexo][5]


### 主题配置
hexo安装时默认使用landscape主题，我使用的是material主题，虽然主题最近的更新时间都是好几年前，但是还是能用用的！
- 下载material主题
  
  在博客目录，终端输入：
  ```
  cd themes
  git clone git@github.com:iblh/hexo-theme-material.git material
  ```
- 修改网站配置
  
  修改博客主目录下的网站配置文件_config.yml中的theme值，将值修改为安装的主题文件夹名称。（网站配置文件每项的含义可以仔细阅读注释或者查看官方文档）

  `theme: material`
- 主题配置
  
  首先将theme/material 目录下_config.template.yml 重命名为 _config.yml

  阅读material主题的文档[链接][6]，自定义主题配置项。

  这里分享常用的配置：
    - 侧边栏
      ```
      # Sidebar Customize
      sidebar:
          dropdown:
              Email Me:
              link: "mailto: 你的邮箱"
              icon: email
          homepage:
              use: true
              icon: home
              divider: false
          archives:
              use: true
              icon: inbox
              divider: false
          categories:
              use: true
              icon: chrome_reader_mode
              divider: true
          pages:  
              标签云: 
                  link: "/tags"
                  icon: cloud
                  divider: false
              时间轴:
                  link: "/timeline"
                  icon: timeline 
                  divider: true
              友链:
                  link: "/links"
                  icon: links
                  divider: false
              关于:
                  name: 关于
                  link: "/about"
                  icon: person
                  divider: true
          article_num:
              use: true
              divider: false
          footer:
              divider: true
              theme: false
              support: false
              feedback: false
              material: false
      ```
      
     
  


### 提交部署

- 安装部署插件
  - git类型
    终端输入：
    
    `npm install hexo-deployer-git --save`

    对应的网站配置项
    ```
    deploy:
      type: git
      repo: <repository url> 
      branch: [branch]
      message: [message]
    ```
- 仓库配置
  
  配置仓库的部署方式、仓库地址和分支名称，这里选择github和coding作为托管。

  ```
  deploy:
  - type: git
    repository: 
      github: git@github.com:xxx.git,main
      coding: git@e.coding.net:xxx,master
  ```
 - 网站部署

   在博客主目录下，终端输入：

   ```
   hexo d -g
   -g, --generate	部署之前预先生成静态文件
   ```
   执行 `hexo deploy` 时，Hexo 会将 public 目录中的文件和目录推送至 _config.yml 中指定的远端仓库和分支中，并且完全覆盖该分支下的已有内容 

## 一些问题
1. material主题，主页的分页不显示icon，显示源代码
   
   修复：在material主题文件夹的layout目录下，在index.ejs中的分页代码新增`escape:false`
   ```
    <%- paginator({
      prev_text: __('<button aria-label="Last page" class="mdl-button mdl-js-button mdl-js-ripple-effect mdl-button--icon"><i class="material-icons" role="presentation">arrow_back</i></button>'),
      next_text: __('<button aria-label="Next page" class="mdl-button mdl-js-button mdl-js-ripple-effect mdl-button--icon"><i class="material-icons" role="presentation">arrow_forward</i></button>'),
      space: '',
      escape: false,
    }) %>
   ```

  
[1]: https://hexo.io/zh-cn/docs/index.html
[2]: https://zdran.com/20180326.html
[3]: https://nodejs.org/zh-cn
[4]: https://git-scm.com/downloads
[5]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/20230420161436.png
[6]: https://neko-dev.github.io/material-theme-docs/#/