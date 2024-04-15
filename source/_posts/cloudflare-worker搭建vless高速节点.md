---
title: cloudflare+worker搭建vless高速节点
tags: [vless]
date: 2024-02-28 09:51:57
categories: technique
urlname: 27
---

## 准备
- Cloudflare账号
- v2rayN或其他代理软件
- 自己的域名（可选，在无优选IP时用）

## 搭建步骤

### worker创建

登录cloudflare，在Workers and Pages中创建Worker应用程序。
![cloudflare worker][1]

部署脚本，名称可以随意选取
![worker script][2]

部署后，选择刚刚部署的worker，快速编辑脚本，将[该脚本][3]复制，覆盖原hello world脚本。并修改其中的userID和ProxyIP。
![worker-vless.js][4]

userID就是UUID，UUID生成，有以下方法
- 按照脚本注释所说
  
  ```Press "Win + R", input cmd and run:  Powershell -NoExit -Command "[guid]::NewGuid()"```

  即在命令行输入对应命令
- V2rayN生成
  ![v2rayN uuid][5]
- 在线工具生成
  
  https://1024tools.com/uuid

  https://it.huluohu.com/uuid-generator

ProxyIP设置的IP或域名用于中转流量，而不是直接路由到使用Cloudflare的网站。如果没有设置ProxyIP，托管在Cloudflare的一些网站会无法访问。

[3Kmfi6HP][14]提供的CDN ProxyIP如下：
```
cdn-all.xn--b6gac.eu.org

cdn.xn--b6gac.eu.org

cdn-b100.xn--b6gac.eu.org

edgetunnel.anycast.eu.org

cdn.anycast.eu.org
```

UUID和ProxyIP设置后，保存并部署。

查看worker脚本603-626行的代码，vless和clash的节点配置信息如下。

![worker-js comment][6]

其中hostname是worker设置的反代地址（name+Zone），即路由地址。userID为刚刚设置的UUID。

![worker-host][7]

也可以在浏览器输入：https://你的hostname/你的UUID
查看赋值后的节点配置信息。（由于workers.dev域名已被墙，因此无法直接访问）

![worker setting][8]

### V2rayN添加节点

如上文提到的，workers.dev域名已无法直接访问，若也没有自己的域名，则只能使用http传输，无法使用443端口。需要使用**优选ip**进行反向代理。

以下是优选ip的测试工具，测试后选择一个可行IP即可：
```
http://ip.flares.cloud/
https://stock.hostmonit.com/CloudFlareYes
https://vfarid.github.io/cf-ip-scanner/?max=30
```

使用优选IP进行反向代理，则此时节点的配置信息需要修改，将服务器地址改为优选IP，端口改为80或2052，不勾选传输层安全TLS（为空）。

![vless-seeting][9]


若有自己的域名，则可以通过**自定义域名**进行代理。
在Cloudflare-websites添加自己的域名

![cloudflare website][10]

将域名服务器更改为Cloudflare的域名服务器。即将cloudflare提供的域名服务器信息覆盖自己注册域名所在平台DNS管理中的域名服务器信息(以下以阿里云→Cloudflare为例)。域名服务器更换成功，Cloudflare会发送邮件通知。

![cloudflare NS][11]
![aliyun DNS][12]

并将原解析记录迁移至Cloudflare（即在Cloudflare添加原域名解析记录）。

此时，在worker中选择triggers中的自定义域，添加自己的（子）域名，例如vless.example.com，添加成功后，DNS解析记录也会新增一条类型为worker的记录。
通过自定义域名，可以使用TLS和443端口。
此时节点的配置信息，可以开启传输层安全，服务器地址、tls servername填写自定义域名，端口可以选择443、2083、8443、2087、2053、2096.

![vless custom domain][13]


## 参考资料
https://www.huluohu.com/posts/913/
https://www.thinkhubx.com/network-cf-worker/
https://mailberry.com.cn/2023/07/use-cloudflare-workers-to-vless/

[1]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/cloudflare_worker.png
[2]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/worker_script.png
[3]: https://github.com/zizifn/edgetunnel/blob/main/src/worker-vless.js
[4]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/worker-vless.png
[5]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/v2rayN-uuid.png
[6]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/worker-vless-setting.png
[7]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/worker-host.png
[8]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/worker-vless-info.png
[9]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/vless-setting.png
[10]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/cloudflare-website.png
[11]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/cloudflare-NS.png
[12]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/aliyun-DNS.png
[13]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/vless-domain.png
[14]: https://github.com/3Kmfi6HP