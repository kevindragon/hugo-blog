---
title: "使用WebSocket技术实现ChatGPT应用"
date: 2023-06-25T12:34:25+08:00
tags: ["Python", "ChatGPT", "WebSocket"]
categories: [编程语言]
keywords: ["Python", "WebSocket", "ChatGPT", "Python实现ChatGPT打字效果"]
---

在[之前的文章]({{< ref "聊聊ChatGPT消息流回复原理" >}})当中介绍了利用Server-sent events技术实现了ChatGPT消息流回复，也就是ChatGPT在回复的时候的打字效果。本文介绍如何使用WebSocket技术来实现类似的效果，同时还实现在实时聊天的过程当中可以停止生成的功能。

## WebSocket简介

WebSocket是一种网络通信协议，通常在web应用当中作用他。看这名字就知道他跟Socket有一些相似之处，但又没有太多的联系。

### 什么是Socket

Socket翻译为套接字，就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。从所处的地位来讲，套接字上联应用进程，下联网络协议栈，是应用程序通过网络协议进行通信的接口，是应用程序与网络协议栈进行交互的接口。

**Socket 通信示例**

![Socket Connection Example](/img/Python/Socket_connection_example.png)

主机 A 的应用程序必须通过 Socket 建立连接才能与主机B的应用程序通信，而建立 Socket 连接需要底层 TCP/IP 协议来建立 TCP 连接。而建立 TCP 连接需要底层 IP 协议来寻址网络中的主机。

![TCP/IP编程模型](/img/Python/tcp_ip-programming-model.png)

上图直接画出来了编程模型，如果用C语言或者Java语言编写过Socket应用，对上面的一些函数调用就非常的熟悉了。后面介绍的WebSocket编程模型与上面的有非常相似的地方，熟悉Socket编程模型对实现WebSocket是非常有帮助的。

### 什么WebSocket

WebSocket是一种与[HTTP](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)不同的协议。两者都位于[OSI模型](https://zh.wikipedia.org/wiki/OSI%E6%A8%A1%E5%9E%8B)的[应用层](https://zh.wikipedia.org/wiki/%E5%BA%94%E7%94%A8%E5%B1%82)，并且都依赖于[传输层](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E5%B1%82)的TCP协议。 虽然它们不同，但是RFC 6455中规定：`it is designed to work over HTTP ports 80 and 443 as well as to support HTTP proxies and intermediaries`（WebSocket通过HTTP端口80和443进行工作，并支持HTTP代理和中介），从而使其与HTTP协议兼容。 为了实现兼容性，WebSocket握手使用HTTP Upgrade头从HTTP协议更改为WebSocket协议。

WebSocket协议支持Web浏览器（或其他客户端应用程序）与Web服务器之间的交互，具有较低的开销，便于实现客户端与服务器的实时数据传输。 服务器可以通过标准化的方式来实现，而无需客户端首先请求内容，并允许消息在保持连接打开的同时来回传递。通过这种方式，可以在客户端和服务器之间进行双向持续对话。 通信通过TCP端口80或443完成，这在防火墙阻止非Web网络连接的环境下是有益的。另外，[Comet](https://zh.wikipedia.org/wiki/Comet_(web%E6%8A%80%E6%9C%AF))之类的技术以非标准化的方式实现了类似的双向通信。

大多数浏览器都支持该协议，包括Google Chrome、Firefox、Safari、Microsoft Edge、Internet Explorer和Opera。

与HTTP不同，WebSocket提供全双工通信。此外，WebSocket还可以在TCP之上实现消息流。TCP单独处理字节流，没有固有的消息概念。 在WebSocket之前，使用Comet可以实现全双工通信。但是Comet存在TCP握手和HTTP头的开销，因此对于小消息来说效率很低。WebSocket协议旨在解决这些问题。

早期，很多网站为了实现[推送技术](https://zh.wikipedia.org/wiki/%E6%8E%A8%E9%80%81%E6%8A%80%E6%9C%AF)，所用的技术都是[轮询](https://zh.wikipedia.org/wiki/%E8%BC%AA%E8%A9%A2)。轮询是指由浏览器每隔一段时间（如每秒）向服务器发出HTTP请求，然后服务器返回最新的数据给客户端。这种传统的模式带来很明显的缺点，即浏览器需要不断的向服务器发出请求，然而HTTP请求与回复可能会包含较长的[头部](https://zh.wikipedia.org/wiki/HTTP%E5%A4%B4%E5%AD%97%E6%AE%B5)，其中真正有效的数据可能只是很小的一部分，所以这样会消耗很多带宽资源。

比较新的轮询技术是[Comet](https://zh.wikipedia.org/wiki/Comet_(web%E6%8A%80%E6%9C%AF))。这种技术虽然可以实现双向通信，但仍然需要反复发出请求。而且在Comet中普遍采用的[HTTP长连接](https://zh.wikipedia.org/wiki/HTTP%E6%8C%81%E4%B9%85%E9%93%BE%E6%8E%A5)也会消耗服务器资源。

在这种情况下，[HTML5](https://zh.wikipedia.org/wiki/HTML5)定义了WebSocket协议，能更好的节省服务器资源和带宽，并且能够更实时地进行通讯。

Websocket使用`ws`或`wss`的[统一资源标志符](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6)（URI）。其中`wss`表示使用了[TLS](https://zh.wikipedia.org/wiki/TLS)的Websocket。如：

```
ws://example.com/wsapi
wss://secure.example.com/wsapi
```

Websocket与HTTP和HTTPS使用相同的TCP端口，可以绕过大多数防火墙的限制。默认情况下，Websocket协议使用80端口；运行在TLS之上时，默认使用443端口。

#### 优点

- 较少的控制开销。在连接创建后，服务器和客户端之间交换数据时，用于协议控制的数据包头部相对较小。在不包含扩展的情况下，对于服务器到客户端的内容，此头部大小只有2至10字节（和数据包长度有关）；对于客户端到服务器的内容，此头部还需要加上额外的4字节的[掩码](https://zh.wikipedia.org/wiki/%E6%8E%A9%E7%A0%81)。相对于HTTP请求每次都要携带完整的头部，此项开销显著减少了。
- 更强的实时性。由于协议是全双工的，所以服务器可以随时主动给客户端下发数据。相对于HTTP请求需要等待客户端发起请求服务端才能响应，延迟明显更少；即使是和Comet等类似的[长轮询](https://zh.wikipedia.org/w/index.php?title=%E9%95%BF%E8%BD%AE%E8%AF%A2&action=edit&redlink=1)比较，其也能在短时间内更多次地传递数据。
- 保持连接状态。与HTTP不同的是，Websocket需要先创建连接，这就使得其成为一种有状态的协议，之后通信时可以省略部分状态信息。而HTTP请求可能需要在每个请求都携带状态信息（如身份认证等）。
- 更好的二进制支持。Websocket定义了二进制帧，相对HTTP，可以更轻松地处理二进制内容。
- 可以支持扩展。Websocket定义了扩展，用户可以扩展协议、实现部分自定义的子协议。如部分浏览器支持压缩等。
- 更好的压缩效果。相对于[HTTP压缩](https://zh.wikipedia.org/wiki/HTTP%E5%8E%8B%E7%BC%A9)，Websocket在适当的扩展支持下，可以沿用之前内容的上下文，在传递类似的数据时，可以显著地提高压缩率。

[WebSocket的基本教程](https://www.ruanyifeng.com/blog/2017/05/websocket.html)推荐阮一峰的博客[https://www.ruanyifeng.com/blog/2017/05/websocket.html](https://www.ruanyifeng.com/blog/2017/05/websocket.html)

## 初始化项目

项目需要一个服务端Python程序和一个客户端JavaScript程序，项目目录结构如下：

```
|-- index.html
|-- main.py
`-- requirements.txt
```

## 服务端FastAPI

本文在服务端选择FastAPI框架来开发，FastAPI对WebSocket支持已经非常的完善了。

项目的依赖模块如下，本文使用一个`for i in range`表达式来代码ChatGPT调用，如果你想了解如何调用ChatGPT的API，请参考我之前写的文章。

```
fastapi==0.92.0
websockets==10.4
uvicorn[standard]==0.20.0
```

websockets模块需要单独安装，fastapi才会支持WebSocket连接。

下面我们来写服务端代码，只需要少量的代码即可实现WebSocket服务端。

首先引入必要的模块

```python
import time
from fastapi import FastAPI, WebSocket, Response
```

这里引入的`time`模块我们来模拟生成过程的停顿。

然后创建FastAPI的app对象

```python
app = FastAPI()
```

添加html静态页面的输出，我们的服务会在`8001`端口启动，当访问[http://localhost:8001](http://localhost:8001)会加载index.html静态页面内容，静态页面内容包含了JavaScript代码，在后面将会看到。

```python
@app.get("/")
async def index():
    # 读取html内容输出给客户端
    return Response(open("index.html", encoding="utf-8").read())
```

后来到WebSocket服务端核心代码了。请注意，我们的服务器可以接收非常多的用户进行连接与使用，而且WebSocket都是长连接，所以需要一个连接管理器来管理这些连接，我们创建一个类来实现连接管理的目的。

```python
class ConnectionManager:
    def __init__(self):
        self.active_connections: list[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

## 创建一个连接管理器
wsConnManager = ConnectionManager()
```

我们使用 `ConnectionManager` 来管理连接，包含了两个方法，`connect`和`disconnect` 方法，用于连接和断开WebSocket。最后创建一个实例对象在后面的代码当中调用。

最核心的代码来了

```python
@app.websocket("/ws")  # 1
async def chat_websocket(websocket: WebSocket):
    # 与客户端建立连接
    await wsConnManager.connect(websocket)  # 2
    try:
        while True:  # 3
            # 接收客户端请求
            text = await websocket.receive_text()  # 4
            for i in range(12):  # 5
                time.sleep(1)    # 5
                # 向客户端发送数据
                await websocket.send_text(str(i))  # 6
                # 等待客户端确认并接收客户端回传数据
                text = await websocket.receive_text()  # 7
                # 如果客户端发送了stop信号，则退出生成
                if text == 'stop':  # 8
                    break
            # 向客户端发送生成完成的信号
            await websocket.send_text("_end_")  # 9
    except Exception:
        wsConnManager.disconnect(websocket)
```

下面我们来详细解释每个步骤的作用：

- 注释1：在app对象上使用websocket协议监听一个endpoint `/ws` ，客户端请求的时候使用 `ws://localhost:8001/ws` 这样的url进行连接。
- 注释2：当客户端发起请求的时候，服务器主动进行连接，并把连接信息记录到连接管理器的`active_connections`对象当中，连接后就用占用系统资源。后续客户断开连接的时候，服务器可以释放连接，同时释放系统资源。
- 注释3：一个无限循环，一直监听用户的请求，一轮响应结束后，继续下一轮的监听
- 注释4：用户与程序的连接会在这里挂起，服务端线程一直等待客户端的请求，只要有数据发送到服务端，程序就会往下执行
- 注释5：这两行代码使用range和sleep模拟生成的内容，向客户端一个字符一个字符的发送数据，每发一个数据等待一会
- 注释6：向客户端发送字符
- 注释7：发送完字符后，等待客户端给一个反馈信号
- 注释8：当接收到客户端发送了stop信息后，中断信息的生成与发送
- 注释9：发送一个完成信号给到客户端，客户端可以做一些逻辑与交互的操作

服务端写完后使用命令 `uvicorn main:app --port 8001 --reload` 启动程序。咱们的html页面还没有写，暂时还访问不了，下面来写html代码。

## HTML与JavaScript客户端

现在我们来创建一个 `index.html` 文件

```html
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
  </head>
  <script>
  // TODO
  </script>
  <style>
  #message {
      font-size: 24px;
      font-weight: bold;
  }
  .cursor {
      margin-left: 5px;
      width: 2px;
      background-color: gray;
      height: 20px;
      display: inline-block;
  }
  </style>
  <body onload="onReady()">
    <p>
      <button onclick="sendData('客户端消息')">WebSocket生成</button>
      <span>&nbsp;</span>
      <button onclick="stop()">停止 WebSocket</button>
    </p>
    <p>
      <span id="message"></span>
      <i class="cursor"></i>
    </p>
    <p>
      <ul></ul>
    </p>
  </body>
</html>
```

我们有两个按钮 `WebSocket生成` 以及 `停止WebSocket` 按钮，两个按钮上分别绑定了`sendData`和`stop`监听函数。这是html框架内容，CSS的内容就略过解释了，每个项目都会有自己的look&feel，下面直接来写JavaScript代码。

先来初始化四个变量

```jsx
  let chars = "";
  let client = null;
  let el = null;
  let stopped = false;
```

`chars`用来存放服务器发送过来的字符，然后用于渲染到html的div#message元素当中；`client`变量存放WebSocket的实例对象；`el`是`div#message`元素的引用对象，用于渲染；`stopped`变量标识是否停止了一轮数据交互。

```jsx
  function render() {
      el.innerHTML = chars;
  }

  function reset() {
      chars = "";
      stopped = false;
      render();
  }
```

`render`函数用于把接收到的字符串信息渲染到`div#message`当中，`reset`函数用于重置数据和页面显示内容。

上面的html代码当中在body元素上有一个`onReady`的函数调用，当html加载完成后就会调用`onReady`函数

```jsx
  function onReady() {
      el = document.getElementById("message");
      client = new WebSocket("ws://localhost:8001/ws");
      client.onmessage = onMessage;
  }
```

初始化`el`对象，找到用于显示数据的div元素，如何WebSocket对象，存放到`client`变量，设置当WebSocket接收到消息的处理函数 `onMessage`

```jsx
  function onMessage(event) {
      const data = event.data;
      if (data == '_end_') {
          chars += "  已完成";
          stopped = true;
          render();
          return;
      }
      if (!stopped) {
          chars += " " + data;
          render();
          client.send('');  // 注释1
      }
  }
```

event.data就是服务端发送过来的字符，判断是否停止或者完成一轮数据会话，使用`_end_`作为标识。如果没有停止，把接收到数据添加到chars对象，然后渲染出来。

重点来了，上面我们服务端发送一个字符就会等客户端回送一个消息，上面的注释1处就是回复服务端一个空消息，这相当于是一个数据协议，服务端会继续后面的生成内容，并不断的发送到客户端。这个机制是为了客户端可以主动停止生成。

```jsx
  function sendData(msg) {
      reset();
      client.send(msg);
  }

  function stop() {
      client.send('stop');
  }
```

`sendData`函数是`WebSocket生成按钮`的调用函数，相当于开启了一轮会话，开启之前先调用 reset 函数重置数据与状态。然后发送一个消息给服务端，如果是后端调用得是ChatGPT的API的话，那么就会把前端的message发送到ChatGPT，ChatGPT会以流的形式不断的把消息返回到服务端，然后服务端不断的把消息发往客户端。

最后一个函数`stop`，当点击`停止WebSocket按钮`，就会中止这一轮的会话。

完整的前端代码如下

```jsx
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
  </head>
  <script>
  let chars = "";
  let client = null;
  let el = null;
  let stopped = false;

  function render() {
      el.innerHTML = chars;
  }

  function reset() {
      chars = "";
      stopped = false;
      render();
  }

  function onMessage(event) {
      const data = event.data;
      if (data == '_end_') {
          chars += "  已完成";
          stopped = true;
          render();
          return;
      }
      if (!stopped) {
          chars += " " + data;
          render();
          client.send('');
      }
  }

  function onReady() {
      el = document.getElementById("message");
      client = new WebSocket("ws://localhost:8001/ws");
      client.onmessage = onMessage;
  }

  function sendData(msg) {
      reset();
      client.send(msg);
  }

  function stop() {
      client.send('stop');
  }
  </script>
  <style>
  #message {
      font-size: 24px;
      font-weight: bold;
  }
  .cursor {
      margin-left: 5px;
      width: 2px;
      background-color: gray;
      height: 20px;
      display: inline-block;
  }
  </style>
  <body onload="onReady()">
    <p>
      <button onclick="sendData('客户端消息')">WebSocket生成</button>
      <span>&nbsp;</span>
      <button onclick="stop()">停止 WebSocket</button>
    </p>
    <p>
      <span id="message"></span>
      <i class="cursor"></i>
    </p>
    <p>
      <ul></ul>
    </p>
  </body>
</html>
```

## Demo动画

我们来看一下Demo，服务器会固定生成0到11的数字

{{< bilibili BV1Cu41187qj >}}


## Nginx设置

如果你使用了Nginx作了反向代码，一定要记得开启WebSocket支持，添加如下代码到`nginx.conf`主配置文件当中。

```
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
```

然后在对应的url开启

```
location /ws
{
    ......
    # 下面这两行是关键
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
}
```

## 总结

本文介绍了什么是WebSocket，跟Socket的关系与异同，以及WebSocket的标准与技术原理。然后我们使用了Python的FastAPI框架和JavaScript，使用WebSocket技术实现了类似ChatGPT的消息回复效果（打字效果），同时还支持了中止当中会话输出的功能。

## 公众号

欢迎关注我的公众号，同步更新

![木木小小孩](/img/qrcode_for_gh.jpg)
