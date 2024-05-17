---
title: 项目开发-Nginx部署
tags: [nginx]
date: 2024-05-11 11:13:54
categories: technique
urlname: 35
---


Nginx是一个开源（BSD许可）的异步框架的 Web 服务器，可以用作反向代理，负载均衡器和 HTTP 缓存。

### Nginx代理与反向代理

在Nginx代理中，Nginx服务器作为客户端和后端服务器之间的中间人，将客户端请求转发至后端服务器，并将后端服务器的响应返回给客户端。这种代理方式代替客户端发送请求，隐藏了真实的客户端，通常用于加速访问、缓存等目的。

在Nginx反向代理中，Nginx服务器作为后端服务器的代理，客户端请求先到达Nginx服务器，然后Nginx服务器再将请求转发至后端服务器。后端服务器将响应返回给Nginx服务器，再由Nginx服务器返回给客户端。这种代理方式通常用于隐藏后端服务器的真实IP地址、负载均衡、安全过滤等目的。


### 安装Nginx

在 Linux 系统上安装 Nginx 通常有以下几种方式：

1. 通过包管理工具安装：
   - Debian/Ubuntu 系统：可以使用 apt-get 命令安装 Nginx，命令为：`sudo apt-get install nginx`
   - CentOS/RHEL 系统：可以使用 yum 命令安装 Nginx，命令为：`sudo yum install nginx`

2. 从源码编译安装：
   - 首先需要从 Nginx 官网下载源码包，并解压缩到本地
   - 进入解压后的 Nginx 目录，执行 `./configure` 命令进行配置，可以指定一些编译参数
   - 然后执行 `make` 命令编译源码
   - 最后执行 `make install` 命令进行安装

3. 使用第三方工具安装：
   - 一些 Linux 发行版提供了一键安装脚本或工具来方便安装 Nginx，比如 OpenResty、Nginx Plus 等

安装完成后可以使用 `nginx -v` 命令来查看 Nginx 的版本信息，使用 `nginx -t` 命令来测试配置文件的语法是否正确，使用 `systemctl start nginx` 命令来启动 Nginx 服务。

在终端中使用`nginx -t` 命令检查Nginx配置文件是否有语法错误时，终端也会输出配置文件路径。默认情况下，配置文件地址为“程序目录/conf/nginx.conf”，即配置文件通常放在目录 /usr/local/nginx/conf（源码安装默认路径），/etc/nginx/conf （包管理工具安装默认路径）或 /usr/local/etc/nginx/conf 中。）

```
lab@bit-PowerEdge-R740xd:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Nginx 配置文件


#### 文件结构与内容

配置文件由一些指令控制模块组成，其决定了 nginx 及其模块的工作方式。

> 指令可分为简单指令和块指令。一个简单的指令是由空格分隔的名称和参数组成，并以分号 ; 结尾。块指令具有与简单指令相同的结构，但不是以分号结尾，而是以大括号{}包围的一组附加指令结尾。如果块指令的大括号内部可以有其它指令，则称这个块指令为上下文（例如：events，http，server 和 location）。
配置文件中被放置在任何上下文之外的指令都被认为是主上下文 main。events 和 http 指令在主 main 上下文中，server 在 http 中，location 又在 server 中

配置文件nginx.conf内容如下：

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
                                     

```

常见的配置项含义如下：

- user：指定 Nginx worker 进程运行的用户和用户组
- worker_processes：指定 Nginx 启动时创建的 worker 进程数量
- error_log：指定错误日志文件的路径
- pid：指定 Nginx 主进程的 PID 文件路径
- events：指定 Nginx 事件模块的配置，如 worker_connections（每个 worker 进程的最大连接数）
- http：指定 HTTP 模块的配置，包括 server（定义虚拟主机）、upstream（定义负载均衡）、location（定义 URL 匹配规则）等配置项
- server：定义一个虚拟主机，包括监听的端口、域名、SSL 配置等
- location：定义 URL 匹配规则，包括匹配的 URL 路径、反向代理配置、缓存配置等
- access_log：指定访问日志文件的路径
- root：指定网站根目录的路径
- index：指定默认的首页文件
- include：包含其他配置文件
- upstream：定义负载均衡的后端服务器
- proxy_pass：指定反向代理的目标地址
- ssl_certificate / ssl_certificate_key：指定 SSL 证书和私钥的路径
- gzip：开启或关闭 HTTP 响应的压缩
- server_name：指定虚拟主机的域名
- error_page：定义错误页面的处理方式
- rewrite：重定向 URL 请求
- limit_req_zone / limit_req：限制请求速率

如果虚拟主机的配置也在该文件，不便于管理。因此，通常使用`include /etc/nginx/conf.d/*.conf;`引入，并在`/etc/nginx/conf.d/`目录下创建对应的域名配置文件。

在外网使用域名访问网站服务时，请求过程为：外网域名 -> 外网服务器:80/443 -> 外网请求映射到内网 -> Nginx反向代理 -> 内网前端:xx（前端显示） -> （发出请求） -> 外网域名 -> 外网服务器:80 -> 内网前端:xx -> Nginx反向代理 -> 内网后端

具体地，当使用域名访问时，根据DNS记录，域名被解析为对应的外网IP。HTTP默认访问80端口，HTTPS访问443端口，因此客户端对http://xxx.com的请求会被映射到外网IP:80上，对https//xxx.com的请求则映射到外网IP:443上。

经过NAT转换将外网映射到内网服务器监听端口，内网服务器上的Nginx服务监听到请求后，将该端口的请求转发至内网前端服务运行端口。此时，可以通过域名访问前端页面，并查看页面的静态内容，之后，前端向内网后端发起请求加载动态内容。

#### 具体例子

将来自www.example.com域名的HTTP请求通过Nginx代理转发至本地的127.0.0.1:806地址，实现反向代理。Nginx配置文件如下：
```
#http
server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                # 代理地址
        proxy_pass http://127.0.0.1:806/;
    }
}


```


实现对前端和后端API服务的反向代理和HTTPS加密配置，同时设置访问日志和错误日志的记录路径等功能。例如：其中dev.redamancy.tech对应前端服务，api-dev.redamancy.tech对应后端API服务，内网前端服务在800端口，内网后端服务在8082端口，前端Nginx配置文件frontend.dev.redamancy.tech.conf内容如下：

```
upstream dev-redamancy.tech {
    server 127.0.0.1:800;
}


# https
server {
    listen 443  ssl;
    server_name  dev.redamancy.tech;

    include mime.types;

    gzip on;
    gzip_min_length 256;
    gzip_comp_level 4;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript text/x-js;

    # ssl configurations
    ssl_certificate /etc/nginx/cert/xxx.crt;
    ssl_certificate_key /etc/nginx/cert/xxx.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;


    access_log  /etc/nginx/logs/dev-redamancy.tech_access.log;
    error_log   /etc/nginx/logs/dev-redamancy.tech.log error;

    #static
    location / {
        proxy_pass http://dev-redamancy.tech;
    }

    error_page 405 =200 http://$host$request_uri;

    underscores_in_headers on;

}

server {
    listen 80;

    server_name dev.redamancy.tech;

    rewrite ^(.*) https://$server_name$1 permanent;
}

```

后端Nginx配置文件api.dev.redamancy.tech.conf内容如下：

```
upstream api.dev-redamancy.tech {
   server 127.0.0.1:8082;
}



server {
   listen 443 ssl;
   server_name api-dev.redamancy.tech;


   client_max_body_size 100M;

   gzip off;
   #gzip_min_length 256;
   #gzip_comp_level 4;
   #gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript text/x-js;

   # ssl configurations
   ssl_certificate /etc/nginx/cert/xxx.crt;
   ssl_certificate_key /etc/nginx/cert/xxx.key;
   ssl_protocols TLSv1.2 TLSv1.3;
   ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
   ssl_prefer_server_ciphers on;


   proxy_max_temp_file_size 0;
   proxy_set_header Host $host:$server_port;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header X-Forwarded-Host $host;
   proxy_set_header X-Forwarded-Server $host;
   proxy_set_header X-Forwarded-Proto  http;
   #proxy_set_header Host $http_host;
   #proxy_pass_request_headers on;
   #proxy_set_header X-NginX-Proxy true;
   #proxy_redirect off;

   access_log  /etc/nginx/logs/api.dev-redamancy.tech_access.log;
    error_log   /etc/nginx/logs/api.dev-redamancy.tech_error.log error;


   location / {
       proxy_pass http://api.dev-redamancy.tech;
   }
}




server {
    listen 80;

    server_name api-dev.redamancy.tech;

    rewrite ^(.*) https://$server_name$1 permanent;
}
                                                            

```



#### 重新加载配置

Nginx 有一个主进程（Master）和几个工作进程（Worker）。主进程的主要目的是读取和评估配置，并维护工作进程。工作进程对请求进行处理。

若配置文件发生更改，可以重新启动Nginx使得配置生效，也可以将重新加载配置信号发送到 Nginx 的主进程。

nginx 启动之后，可以通过调用可执行文件附带 -s 参数 来控制，命令为`nginx -s 信号`。要重新加载配置，执行的命令为：`nginx -s reload`。

一旦主进程收到要重新加载配置的信号，它将检查新配置文件的语法有效性，并尝试应用其中提供的配置。如果成功，主进程将启动新的工作进程，并向旧工作进程发送消息，请求它们关闭。否则，主进程回滚更改，并继续使用旧配置。旧工作进程接收到关闭命令后，停止接受新的请求连接，并继续维护当前请求，直到这些请求都被处理完成之后，旧工作进程将退出。

参考资料：
[Nginx Docs][1]
[Nginx 中文文档][2]
[nginx--正向代理、反向代理及负载均衡（图解+配置）][3]



[1]: https://docs.nginx.com/
[2]: https://docshome.gitbook.io/nginx-docs/
[3]: https://blog.csdn.net/justinqin/article/details/119519019