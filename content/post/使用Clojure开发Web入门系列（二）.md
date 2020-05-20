---
title: 使用Clojure开发Web入门系列（二）
date: 2016-03-08 21:17:10
tags: [Clojure, Web]
---

开发web应用当然少不了web服务器了，咱们这里使用是的[`ring`](https://github.com/ring-clojure/ring)这个库。

# 零、添加依赖
打开根目录下的`project.clj`，找到`:dependencies`。

```clojure
:dependencies [[org.clojure/clojure "1.8.0"]]
```

有可能你看到的clojure版本跟我的不太一样，在里添加(`ring`)[https://github.com/ring-clojure/ring]，添加后像下面这样

```clojure
:dependencies [[org.clojure/clojure "1.8.0"]
               [ring "1.6.3"]]
```

然后在命令行运行：

```shell
lein deps
```

`lein`底层使用的是`maven`和(`clojars`)[https://clojars.org/]，下载下来的依赖包会在你的用户目录下的`.m2`目录，下载的所有文件可以在`~/.m2/repository/ring/ring/1.6.3/`找到。如果你身在"赵国"，那么你家里应该常备"楼梯"。

<!-- more -->

# 一、启动HTTP Server
打开`src/learnweb/core.clj`，这个是在`project.clj`里面的`:main`定义的入口点：

```clojure
(ns learnweb.core
  (:gen-class))

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (println "Hello, World!"))
```

下面来改造一下`core.clj`文件，把需要的函数包含进代码，然后把http server跑起来，最终的代码如下：

```clojure
(ns learnweb.core
  (:require [ring.adapter.jetty :refer [run-jetty]])  ;(1)
  (:gen-class))

(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/html"}
   :body "Hello World!"}) ;(2)

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (run-jetty handler {:port 3000})) ;(3)
```

改造了三个地方(1), (2), (3)

(1) 添加运行http server的函数`run-jetty`，`run-jetty`函数需要另外一个函数来处理http请求，同时会把请求的参数传递给那个函数；

(2) 处理http的函数，这里简单的返回状态码`200`，网页的内容类型`text/html`，网页的内容`"Hello World!"`；

(3) 通过`run-jetty`函数运行http server，启动的时候指定端口为`3000`。

然后在命令行运行如下命令：

```clojure
lein run
```

你将会看到如下的输出：

```text
2016-03-08 21:47:49.003:INFO::main: Logging initialized @1130ms
2016-03-08 21:47:49.073:INFO:oejs.Server:main: jetty-9.2.10.v20150310
2016-03-08 21:47:49.113:INFO:oejs.ServerConnector:main: Started ServerConnector@29a5f4e7{HTTP/1.1}{0.0.0.0:3000}
2016-03-08 21:47:49.114:INFO:oejs.Server:main: Started @1241ms
```

打开浏览器，访问[http://localhost:3000](http://localhost:3000)，你就会看到一个网页版的Hello World!

完整代码在我的github上面：[Learn Clojure web](https://github.com/kevindragon/learn-clojure-web)

```shell
git clone https://github.com/kevindragon/learn-clojure-web
git checkout 1.2
```

## 工作原理

`run-jetty`函数在指定的端口启动一个web server，接收http request，然后把请求交给`handler`函数进行处理，上面的`handler`函数返回一个状态码200，响应头指定Content-Type为html内容，响应的内容为Hello World!

# 二、显示更多信息

## 显示浏览器的User agent
把`"Hello World!"`改成

```clojure
(str "User agent: " (get-in request [:headers "user-agent"]))
```

handler函数变成如下：

```clojure
(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/html"}
   :body (str "User agent: " (get-in request [:headers "user-agent"]))})
```

然后在命令行按ctrl-c终止上次的运行，再次执行`lein run`，刷新浏览器便可以看到你自己浏览器的user agent，我的浏览器显示为：

```
User agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36
```

你的浏览器显示可能会不一样。`handler`函数的参数为http request的内容，包括请求头信息、url、请求参数等等。这里面直接从request中取出user-agent的信息。

## 根据url上面的参数显示不同的内容
接下来让咱们实现网页的内容显示url上面的信息。

比如`http://localhost:3000/hello/Kevin`，网页显示为`你好Kevin`

如果是`http://localhost:3000/hello/Kitty`则显示为`你好Kitty`

如果是`http://localhost:3000`显示为Index page，其他任何情况都显示`Page not found`。

完整代码如下：

```clojure
(ns learnweb.core
  (:require [ring.adapter.jetty :refer [run-jetty]]
            [clojure.string :refer [split]])
  (:gen-class))

(defn uri-split [str]
  (filter #(not-empty %) (split str #"/")))    ; (1)

(defn route [request]
  (let [uri (:uri request)
        seg (uri-split uri)]
    (cond
      (= uri "/") "Index page"
      (= (first seg) "hello") (str "你好" (nth seg 1 "Oops"))
      :else "Page not found")))    ; (2)

(defn handler [request]
  (let [uri (:uri request)
        seg (uri-split uri)]
    {:status 200
     :headers {"Content-Type" "text/html; charset=utf-8"}
     :body (route request)}))    ; (3)

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (run-jetty handler {:port 3000}))
```

(1) 添加uri-split函数，用于分割uri成列表。/hello/Kitty --> ["hello" "Kitty"]

(2) 添加路由处理函数`route`，也就是根据uri参数把请求分发到不同的处理函数，得到uri列表后，然后使用`cond`函数做分发处理

(3) 在返回的内容使用路由函数处理

完整的[代码](https://github.com/kevindragon/learn-clojure-web)请checkout到`1.2`分支。
