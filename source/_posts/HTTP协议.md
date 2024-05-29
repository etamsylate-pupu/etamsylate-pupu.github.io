---
title: HTTP协议
tags: [protocol]
date: 2024-05-07 10:21:18
categories: technique
urlname: 34
---

HTTP（HyperText Transfer Protocol）是万维网（World Wide Web）的基础协议。


HTTP 在应用的早期阶段非常简单，后来被称为 HTTP/0.9，有时也叫做单行（one-line）协议。

### HTTP/0.9 —— 单行协议

最初版本的 HTTP 协议并没有版本号，后来它的版本号被定位在 0.9 以与后来的版本进行区分。

HTTP/0.9 请求由单行指令构成，以唯一可用方法 GET 开头，其后跟目标资源的路径（一旦连接到服务器，协议、服务器、端口号这些都不是必须的）。


```
GET /mypage.html

```

响应也极其简单，只包含响应文档本身。由于其相应内容并不包含 HTTP 头，意味着只有 HTML 文件可以传送，无法传输其他类型的文件。也没有状态码或错误代码。一旦出现问题，一个特殊的包含问题描述信息的 HTML 文件将被发回，供人们查看。

```
<html>
    This is a simple example
</html>
```

### HTTP/1.0 —— 构建可扩展性

HTTP/1.0 相比 HTTP/0.9 ，扩展了以下内容：

- 协议版本信息现在会随着每个请求发送（HTTP/1.0 被追加到了 GET 行）。
- 状态码会在响应开始时发送，使浏览器能了解请求执行成功或失败，并相应调整行为（如更新或使用本地缓存）。
- 引入了 HTTP 标头的概念，无论是对于请求还是响应，允许传输元数据，使协议变得非常灵活，更具扩展性。
- 在新 HTTP 标头的帮助下，具备了传输除纯文本 HTML 文件以外其他类型文档的能力（凭借 Content-Type 标头）。

HTTP/1.0 请求的典型例子如下：

```
GET /mypage.html HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)

200 OK
Date: Tue, 15 Nov 1994 08:12:31 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/html
<HTML>
一个包含图片的页面
  <IMG SRC="/myimage.gif">
</HTML>

```

```
GET /myimage.gif HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)

200 OK
Date: Tue, 15 Nov 1994 08:12:32 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/gif
(这里是图片内容)

```

在 1991-1995 年，这些新扩展并没有被引入到标准中以促进协助工作，而仅仅作为一种尝试。服务器和浏览器添加这些新扩展功能，但出现了大量的互操作问题。直到 1996 年 11 月，为了解决这些问题，一份新文档（RFC 1945）被发表出来，用以描述如何操作实践这些新扩展功能。文档 [RFC 1945][1] 定义了 HTTP/1.0，但它是狭义的，并不是官方标准。


### HTTP/1.1 —— 标准化的协议

HTTP/1.0 多种不同的实现方式在实际运用中显得有些混乱。HTTP/1.1 消除了大量歧义内容并引入了多项改进（[RFC 2068][2],[RFC 2616][3]）：

- 连接可以复用，节省了多次打开 TCP 连接加载网页文档资源的时间。（长连接）
- 增加管道机制（pipelining），允许在第一个应答被完全发送之前就发送第二个请求，以降低通信延迟。
- 支持响应分块。
- 引入额外的缓存控制机制。
- 引入内容协商机制，包括语言、编码、类型等。并允许客户端和服务器之间约定以最合适的内容进行交换。
- 凭借 Host 标头，能够使不同域名配置在同一个 IP 地址的服务器上。


```
GET /zh-CN/docs/Glossary/Simple_header HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*\/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/zh-CN/docs/Glossary/Simple_header

200 OK
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Wed, 20 Jul 2016 10:55:30 GMT
Etag: "547fa7e369ef56031dd3bff2ace9fc0832eb251a"
Keep-Alive: timeout=5, max=1000
Last-Modified: Tue, 19 Jul 2016 00:59:33 GMT
Server: Apache
Transfer-Encoding: chunked
Vary: Cookie, Accept-Encoding

(content)


GET /static/img/header-background.png HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: *\/*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/zh-CN/docs/Glossary/Simple_header

200 OK
Age: 9578461
Cache-Control: public, max-age=315360000
Connection: keep-alive
Content-Length: 3077
Content-Type: image/png
Date: Thu, 31 Mar 2016 13:34:46 GMT
Last-Modified: Wed, 21 Oct 2015 18:27:50 GMT
Server: Apache

(image content of 3077 bytes)


```

[1]: https://datatracker.ietf.org/doc/html/rfc1945
[2]: https://datatracker.ietf.org/doc/html/rfc2068
[3]: https://datatracker.ietf.org/doc/html/rfc2616