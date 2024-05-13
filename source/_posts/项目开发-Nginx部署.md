---
title: 项目开发-Nginx部署
tags: [nginx]
date: 2024-05-11 11:13:54
categories: technique
urlname: 35
---


Nginx是一个开源（BSD许可）的异步框架的 Web 服务器，可以用作反向代理，负载均衡器和 HTTP 缓存。






### 安装Nginx

环境：Ubuntu 22.04

通过apt工具包安装，终端输入：

```
sudo apt-get install nginx
```

可以在终端中输入以下命令来检查Nginx配置文件是否有语法错误，此时也能查看配置文件路径。（默认情况下，配置文件名为 nginx.conf，并放在目录 /usr/local/nginx/conf，/etc/nginx 或 /usr/local/etc/nginx 中。）

```
lab@bit-PowerEdge-R740xd:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

配置文件内容如下：

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

如果虚拟主机的配置也在该文件，不便于管理。因此，通常使用`include /etc/nginx/conf.d/*.conf;`引入，在`/etc/nginx/conf.d/`目录下创建对应的域名配置文件。





参考资料：
[Nginx Docs][1]





[1]: https://docs.nginx.com/