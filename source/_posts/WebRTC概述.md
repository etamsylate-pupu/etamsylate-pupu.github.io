---
title: WebRTC 概述
tags: [webrtc]
date: 2024-08-03 18:49:32
categories: technique
urlname: 47
---

WebRTC（Web Real-Time Communication）既是 API 也是协议。WebRTC 协议是两个 WebRTC Agent 协商双向安全实时通信的一组规则，其允许在浏览器和移动应用程序中进行实时音视频通信和数据共享，而无需借助中间服务器。



WebRTC 的主要功能
- 音视频通信：支持高质量的音视频通话，能够适应不同的网络环境。
- 数据通道：允许在对等端之间传输任意数据，适用于文件传输、游戏数据同步等场景。
- 网络穿透：通过 STUN（Session Traversal Utilities for NAT）和 TURN（Traversal Using Relays around NAT）服务器，WebRTC 可以穿透 NAT 和防火墙，实现对等连接。


WebRTC 通话建立流程如下图：
- Client A、Client B 都连接信令服务器；
- Client A 创建本地视频，并获取会话描述对象（offer sdp）信息；
- Client A 将 offer sdp 通过信令服务器发送给 Client B；
- Client B 收到信令后，创建本地视频，并获取会话描述对象（answer sdp）信息；
- Client B 将 answer sdp 通过信令服务器发送给 Client A；
- Client A 和 Client B 开始打洞（在两个位于不同私有网络（通常由 NAT设备隔离）的设备之间建立直接通信通道的过程），收集并通过信令服务器交换 ice 信息；
- 完成打洞后，Client A 和 Client B 开始为安全的媒体通信协商秘钥；
- 至此，Client A 和 Client B 可以进行音视频通话。

![WebRTC](https://github.com/etamsylate-pupu/Image-host/blob/main/blogImg/tech/webrtc_process.png)




## 关键步骤

### 信令

信令是 WebRTC 中用于交换控制信息的过程。信令的主要目的是在两个对等端（peer）之间交换必要的信息，以便建立、控制和终止 WebRTC 连接。当 WebRTC Agent 启动时，它不知道与谁通信以及他们将要通信的内容。信令用于解决这个问题，其用于引导呼叫，以便两个 WebRTC Agent 可以开始通信。


虽然把 WebRTC 称之为点对点的连接，但并不代表在实现过程中不需要服务器的参与。相反，在点对点的信道建立起来之前，二者之间是没有办法通信的。这也就意味着，在信令阶段，需要一个通信服务来帮助Web agent建立起这个连接。

信令消息的传递不在 WebRTC 的数据通道或媒体流中进行，而是通过独立的通信渠道进行。这些信令消息包括 SDP（Session Description Protocol，会话描述协议）和 ICE（交互式连接建立）候选，用于协商连接参数和网络信息。开发者可以为应用程序引擎选择任意的信息协议（如 SIP 或 XMPP），任意双向通信信道（如 WebSocket 或 XMLHttpRequest） 与持久连接服务器的 API（如Google Channel API）一起工作，而不受限于 WebRTC 的实现，可以直接复用现有的通信基础设施来传递信令消息，而不需要额外的信令服务器。


#### 会话描述协议（Session Description Protocol, SDP）

SDP 是一种格式化的文本，用于描述多媒体会话的参数，如格式、分辨率、编码和加密算法等。对等端通过信令通道交换 SDP 报文，以协商音视频编解码器、媒体格式等。从技术上讲，SDP 并不是一个真正的协议，而是一种数据格式，用于描述在设备之间共享媒体的连接。SDP 由一行或多行 UTF-8 文本组成，每个字段以一个字母开头，后跟一个等号和具体的值。SDP 的字段可以分为会话级字段和媒体级字段。会话级字段描述整个会话的信息，而媒体级字段描述具体的媒体流信息。

```
v=0
o=- 46117341 2 IN IP4 127.0.0.1
s=-
t=0 0
a=group:BUNDLE audio video
a=msid-semantic: WMS stream1

m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104
c=IN IP4 0.0.0.0
a=rtcp:9 IN IP4 0.0.0.0
a=ice-ufrag:abc123
a=ice-pwd:xyz456
a=ice-options:trickle
a=fingerprint:sha-256 12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF
a=setup:actpass
a=mid:audio
a=sendrecv
a=rtcp-mux
a=rtpmap:111 opus/48000/2
a=rtpmap:103 ISAC/16000
a=rtpmap:104 ISAC/32000
a=ssrc:123456789 cname:stream1

m=video 9 UDP/TLS/RTP/SAVPF 96 97 98
c=IN IP4 0.0.0.0
a=rtcp:9 IN IP4 0.0.0.0
a=ice-ufrag:abc123
a=ice-pwd:xyz456
a=ice-options:trickle
a=fingerprint:sha-256 12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF
a=setup:actpass
a=mid:video
a=sendrecv
a=rtcp-mux
a=rtpmap:96 VP8/90000
a=rtpmap:97 rtx/90000
a=fmtp:97 apt=96
a=rtpmap:98 VP9/90000
a=ssrc:987654321 cname:stream1
```

#### 交互式连接建立（Interactive Connectivity Establishment，ICE）

ICE 通过收集和交换候选地址（candidates），找到最佳的网络路径，以实现对等端之间的直接通信。ICE 主要解决 NAT（网络地址转换）和防火墙穿透问题，使得对等端能够在复杂的网络环境中建立连接。ICE可以分为以下几个步骤：
- 收集候选地址：每个对等端收集自身的候选地址，包括主机候选（Host Candidate）、反射候选（Server Reflexive Candidate）和中继候选（Relay Candidate）。
- 交换候选地址：对等端通过信令通道交换候选地址。
- 连接检查：对等端使用 STUN协议进行连接检查，测试候选地址的可达性。
- 选择最佳路径：根据连接检查的结果，选择最佳的候选地址对，建立连接。

> STUN（Session Traversal Utilities for NAT）服务器：用于获取对等端的公共 IP 地址和端口，帮助穿透 NAT。
> TURN（Traversal Using Relays around NAT）服务器：用于在复杂网络环境中进行中继通信，当直接连接无法建立时，TURN 服务器充当中继节点。

候选地址类型包括：
- 主机候选（Host Candidate）：本地网络接口的 IP 地址和端口。
- 反射候选（Server Reflexive Candidate）：通过 STUN 服务器获取的公共 IP 地址和端口，用于穿透 NAT。
- 中继候选（Relay Candidate）：通过 TURN 服务器获取的中继地址，用于在复杂网络环境中进行中继通信。



#### 信令过程示例（使用WebSocket进行信令）

> WebSocket是一种在单个TCP连接上进行全双工通信的协议。在 WebSocket 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。



**服务器端**

首先，需要一个 WebSocket 服务器来中继信令消息。


- 使用Node.js和ws库
```
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    // 广播消息给所有连接的客户端
    wss.clients.forEach((client) => {
      if (client !== ws && client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });
});

console.log('WebSocket server is running on ws://localhost:8080');

```

- 使用Go 和 gorilla/websocket 库
```
package main

import (
    "log"
    "net/http"
    "github.com/gorilla/websocket"
)

// 存储所有连接的客户端
var clients = make(map[*websocket.Conn]bool)

// 广播通道，用于将消息发送给所有客户端
var broadcast = make(chan Message)

// 升级器，用于将 HTTP 连接升级为 WebSocket 连接
var upgrader = websocket.Upgrader{}

// Message 结构体，用于表示从客户端接收到的消息
type Message struct {
    Data string `json:"data"`
}

func main() {
    // 设置 WebSocket 连接的处理函数
    http.HandleFunc("/ws", handleConnections)

    // 启动一个 goroutine 来处理广播消息
    go handleMessages()

    // 启动 HTTP 服务器，监听端口 8080
    log.Println("WebSocket server is running on ws://localhost:8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}

// 处理 WebSocket 连接的函数
func handleConnections(w http.ResponseWriter, r *http.Request) {
    // 将 HTTP 连接升级为 WebSocket 连接
    ws, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Fatal(err)
    }
    // 确保在函数结束时关闭 WebSocket 连接
    defer ws.Close()

    // 将新的 WebSocket 连接添加到客户端列表
    clients[ws] = true

    // 不断读取来自客户端的消息
    for {
        var msg Message
        // 读取 JSON 格式的消息
        err := ws.ReadJSON(&msg)
        if err != nil {
            log.Printf("error: %v", err)
            // 如果读取消息时出错，将客户端从列表中删除
            delete(clients, ws)
            break
        }
        // 将消息发送到广播通道
        broadcast <- msg
    }
}

// 处理广播消息的函数
func handleMessages() {
    for {
        // 从广播通道中读取消息
        msg := <-broadcast
        // 将消息发送给所有连接的客户端
        for client := range clients {
            err := client.WriteJSON(msg)
            if err != nil {
                log.Printf("error: %v", err)
                // 如果发送消息时出错，关闭连接并将客户端从列表中删除
                client.Close()
                delete(clients, client)
            }
        }
    }
}
```

**客户端**

在客户端，需要实现 WebRTC 的信令逻辑，包括创建 RTCPeerConnection、处理 ICE 候选、交换 SDP 等。







参考资料
[WebRTC协议](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API/Protocols)
[写给好奇者的WebRTC](https://webrtcforthecurious.com/zh/docs/)
[前端WebRTC实战](https://zhuanlan.zhihu.com/p/59520779)
[WebRTC Code Samples](https://github.com/webrtc/samples)
[WebRTC Samples](https://webrtc.github.io/samples/)




