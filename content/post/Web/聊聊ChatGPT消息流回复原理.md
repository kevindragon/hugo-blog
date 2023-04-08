---
title: 聊聊ChatGPT消息流回复原理
date: 2023-04-08T21:36:12+08:00
tags: [Server-sent events, ChatGPT]
---

去年底以来ChatGPT已经火出天际，似乎奇点已经到来。前不久GPT-4发布的新闻更是把这人工智能推向了一个新的高潮。我们注意到，ChatGPT的回复是具有打字的效果，时不时还停顿一下，看起来你就像在跟一个具有智慧的人在聊天，时不时思索一下。

那么ChatGPT的回复效果是如何实现的呢，今天就来聊一聊其中的一种实现方式：Server-sent events。这是一种服务器向客户端主动推送消息的技术。

<!-- more -->

服务器向浏览器推送信息，最先想到的就是WebSocket了。除了 WebSocket，还有一种方法：Server-Sent Events（以下简称 SSE）。本文介绍它的用法以及一个可以运行的示例。

除了SSE和WebSocket以外，还有一种经常使用到的方法是轮询，客户端不断的向服务器端发起查询请求以获得实时数据信息。

下面是 WebSocket、轮询和 SSE 的功能对比：

1. SSE 和轮询使用 HTTP 协议，现有的服务器软件都支持。WebSocket 是一个独立协议
2. SSE 相比 WebSocket 更加轻量，使用简单；WebSocket 使用相对复杂；轮询使用简单
3. SSE 默认支持断线重连，WebSocket 需要自己实现断线重连
4. SSE 一般只用来传送文本，二进制数据需要编码后传送，WebSocket 默认支持传送二进制数据
5. SSE 支持自定义发送的消息类型
6. WebSocket 支持双向推送消息，SSE 是单向的
7. 轮询性能开销大、轮询时间久导致客户端及时更新数据

## 什么是**SSE**

SSE是HTML 5规范的一个组成部分，是一种服务器推送技术，使客户端可以通过HTTP连接从服务器自动接收更新。使用server-sent events，服务器可以在任何时刻向我们的Web页面推送数据和信息。这些被推送进来的信息可以在这个页面上作为Events + data的形式来处理。

严格地说，HTTP 协议无法做到服务器主动推送信息，但是，服务器会向客户端发送响应的内容里面会声明，接下来会发送流信息（streaming），然后服务端就会不断的向客户发送数据流，直到结束。客户端在收到响应后会保持连接，持续的接收服务器发送过来的数据。

SSE 就是利用这种机制，使用流信息向浏览器推送信息。它基于 HTTP 协议，目前除了 IE，其他浏览器都支持。

各种浏览器支持情况可以查看[Can I Use](https://caniuse.com/?search=Server-sent%20events)

![Can I Use Server-sent events](/img/Web/Server-sent_events/can-i-use-server-sent-event.png)

## 举个栗子

下面我们使用Python及FastAPI框架，编写服务端代码。

### 服务端代码

```jsx
import time
import random
from fastapi import FastAPI, Response
from fastapi.responses import StreamingResponse

app = FastAPI()

async def fake_number_streamer():
    for i in range(random.randint(10, 20)):
        yield "{}".format(i+1)
        time.sleep(0.1 * random.randint(1, 10))

@app.get("/")
async def index():
    # 读取html内容输出给客户端
    return Response(open("index.html", encoding="utf-8").read())

@app.get("/sse")
async def sse():
    return StreamingResponse(fake_number_streamer(), media_type="text/event-stream")
```

服务端代码比较简单，首先需要安装`fastapi` （`pip install fastapi`）。然后需要定义一下生成器函数，用来模拟持续输出信息的逻辑，这里直接用一个随机数，输出从1开始，到10，20之间的随机数列。

index函数是首页加载的html内容，在下面会看到具体的内容。

然后最重要的部分就是`StreamingResponse` 响应对象，fastapi框架会自动处理响应内容，`fake_number_streamer()` 函数是一个生成器函数，在StreamingResponse内部的处理逻辑如下：

```jsx
async def stream_response(self, send: Send) -> None:
    await send(
        {
            "type": "http.response.start",
            "status": self.status_code,
            "headers": self.raw_headers,
        }
    )
    async for chunk in self.body_iterator:
        if not isinstance(chunk, bytes):
            chunk = chunk.encode(self.charset)
        await send({"type": "http.response.body", "body": chunk, "more_body": True})
    await send({"type": "http.response.body", "body": b"", "more_body": False})
```

首先调用`send`函数，服务器向浏览器发送的 SSE 数据，必须是 UTF-8 编码的文本，HTTP 头信息必要的参数如下：

```
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```

在上面我们调用`StreamingResponse(fake_number_streamer(), media_type="text/event-stream")` 时，需要设置`media_type` 。

然后是一个循环 `async for chunk in self.body_iterator` 这里的self.body_iterator就是 `fake_number_streamer()` 返回的生成器。

### 前端代码

```jsx
async function getData() {
  const el = document.getElementById("message")
  const resp = await fetch("/sse")
  const reader = resp.body.getReader();
  let text = "";
  while (true) {
    const { value, done } = await reader.read()
    const chars = new TextDecoder().decode(value);
    text += " " + chars;
    el.innerHTML = text;
    if (done) {
      break;
    }
  }
}
```

前端代码的核心就是上面的函数，使用得是 `fetch` 函数，主流浏览器基本都是支持的。

![Can I Use Fetch](/img/Web/Server-sent_events/can-i-use-fetch.png)

fetch函数的返回值是一个Promise。SSE在前端响应是用一个[ReadableStream](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream)对象来表示的，然后就使用getReader函数，获取一个reader对象，接着在一个循环里面不停的读取服务器发送过来的内容，使用reader.read函数进行读取，返回一个对象，包含一个value，还有一个done，如果服务器还没有响应完成，那么done就是false，响应完成就是true。

最后，在接收到更新数据后就展示在html元素当中，就可以看到文章最开始提到的效果了。

#### EventSource

另外，在前端还有一种实现方式，那就是EventSource，这里不展示演示了，可以参考[MDN的文档](https://developer.mozilla.org/zh-CN/docs/Web/API/EventSource)。

## Nginx配置

如果你的服务端是通过nginx或者apache等web服务器来进行代理的，那么这里有一个非常重要的设置，如果配置不正确，那么你就看不到streaming方式的效果了。

这一点就量nginx等web服务器的缓存，因为nginx默认的proxy_buffering是on，所以在Python代码持续输出的时候，nginx会做一个缓存，当缓存大小超过默认的大小的时候才会输出。

对于nginx最重要的几个配置如下，否则streaming就不生效了，一定要注意：

```
proxy_set_header Connection '';
proxy_http_version 1.1;
chunked_transfer_encoding off;
proxy_buffering off;
proxy_cache off;
```

## 总结

本文介绍了ChatGPT回复消息的实现方式之一：Server-sent events。通过这种技术，服务器可以向客户端主动推送消息，实现了类似于聊天的效果。文章介绍了SSE的用法和示例，以及与WebSocket和轮询的比较。同时，还提供了Python及FastAPI框架的服务端代码和前端代码示例，以及Nginx的配置方法。文章最后给出了参考链接。
