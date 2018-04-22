---
layout:     post
title:      "WebSocket"
subtitle:   "双工通信"
date:       2018-03-16 12:00:00
author:     "Lorry"
catalog: true
multilingual: false
header-img: "http://p799phkik.bkt.clouddn.com/night_sky.jpeg"
tags:
    - Node.js
---
现有的HTTP 1.1 连接都是单向连接,即服务器无法主动的向客户端发送信息,必须等待客户端的请求.那么如何使用长连接呢?如果用户需要即时刷新信息应该怎么做呢?

## 1. 轮询(polling)
### 客户端通过不停的请求,来获取信息.此时不管服务器有没有消息都会请求.利用setInterval和setTimeout来设置请求频率
setInterval 客户端代码
``` javascript
setInterval(function() {
    $.get("serverPath", function(data, status) {
        console.log(data);
    });
}, 10000);// 10秒询一次
```
以上的缺点是如果第一次的响应超过十秒,第二次快速响应会导致这两次的返回数据的顺序出错,无法辨别哪一个先返回数据,所以有了setTimeout方式,在回调中继续询
``` javascript
function poll() {
    setTimeout(function() {
        $.get("serverPath", function(data, status) {
            console.log(data);
            // 发起下一次请求
            poll();
        });
    }, 10000);
}
```

## 2. 长连接(也叫长轮询,Long-polling)
### 比如有著名的Comet,客户端与服务器建立连接后如果服务器未响应,则客户端挂起等待服务器推送.
### 服务器会设置一个timeout,在timeout内客户端和服务器处于持续连接的状态,如果这期间有任何msg发生(服务器会阻塞不断的进行判断是否有msg),那么服务器会响应客户端msg,并且关闭此次连接
### 如果timeout之后都没有msg响应,则会关闭此次连接并要求客户再次发送一次长连接请求.
客户端代码
``` javascript
$.ajax({
type: "POST",
contentType: "application/json",
url: "../ws/MyService.asmx/test",
data: '{"data":"'+someData+'"}',
timeout: 30000, //超时时间：30秒
dataType: 'json',
error: function(XMLHttpRequest, textStatus, errorThrown){
//TODO: 处理status， http status code，超时 408
// 注意：如果发生了错误，错误信息（第二个参数）除了得到null之外，还可能
//是"timeout", "error", "notmodified" 和 "parsererror"。
}, 
success: function(result) {
// TODO: check result
}
});
```

由此可见第二种长连接方式有很大的优势,

1 不必耗费大量的时间花在请求上,因为http是无状态的,每次请求需要携带很多与正文无关的内容给到服务器端进行判断(比如请求头),也会消耗带宽,而长连接只需要在一个timeout内连接一次.

2 短轮询也无法控制数据到达的先后顺序

长连接的缺点,
1. IE、Morzilla Firefox 下使用iframe方式建立的长连接,进度栏都会显示加载没有完成，而且 IE 上方的图标会不停的转动，表示加载正在进行
2. 服务器的hold需要消耗资源,对服务器的需求较高

# HTML5中引入了WebSocket

## 一句话总结: websocket可以说是基于HTTP但有有所进化的一个介于应用层和传输层的接口抽象,严格来说不是一种协议.

1. 同样需要基于HTTP进行3次握手,4次挥手(在握手期间建立websocket连接,不再通过HTTP协议传输),.

2. 握手期间发送协议切换的请求,接受101响应,通过web Socket Key进行验证.  后续传输过程不必再包含繁重的请求头信息

3. 双方响应,服务端也可以send massage到客户端,而不仅仅是只有客户端单向请求.

![](http://p799phkik.bkt.clouddn.com/websocket.png)

服务器端(基于nodejs)
``` JavaScript
var WebSocketServer = require('ws').Server;
var wss = new WebSocketServer({port: 8080});

wss.on("connection", function(socket) {
    socket.on("message", function(msg) {
        console.log(msg);
        socket.send("Nice to meet you!");
    });
});
```
客户端
```js
// WebSocket 为客户端JavaScript的原生对象
var ws = new WebSocket("ws://localhost:8080");
ws.onopen = function (event) {
    ws.send("Hello there!");
}
ws.onmessage = function (event) {
    console.log(event.data);
}
```
## 下面是对三种方式的总结:
<style>
table th:first-of-type {
    width: 100px;
}
</style>
|对比项|传统轮询|长轮询|WebSocket|
|:-|:-|:-|:-|
|浏览器支持|几乎所有现代浏览器|几乎所有现代浏览器|IE 10+ Edge Firefox 4+ Chrome 4+ Safari 5+ Opera 11.5+|
|服务器负载|较少的CPU资源,较多的内存资源和带宽资源|与传统轮询相似,但是占用带宽较少|无需循环等待(长轮询),CPU和内存资源不以客户端数量衡量,而是以客户端事件数衡量。四种方式里性能最佳|
|客户端负载|占用较多的内存资源与请求数|与传统轮询相似|浏览器中原生实现，占用资源很小|
|延迟|非实时,延迟取决于请求间隔|同传统轮询|实时|
|实现复杂度|非常简单|需要服务器配合,客户端实现非常简单|需要Socket程序实现和额外端口,客户端实现简单|


