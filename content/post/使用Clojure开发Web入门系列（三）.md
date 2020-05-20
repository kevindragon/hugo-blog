---
title: 使用Clojure开发Web入门系列（三）
date: 2016-03-14 19:12:49
tags: [Clojure, Web]
---

在[使用Clojure开发Web入门系列（二）](/2016/03/08/使用Clojure开发Web入门系列（二）)的最后做了一个动态输出hello+url参数的一个小demo，如果有很多这样的需求，不同的url调用不同的处理函数，那么就得在调用cond的地方写上很多条件，或者再写上另外一个函数专门处理这些条件，这就是web后端开发的路由组件。

Clojure方便的一点就在此，对于这个功能已经有一个非常强大而实用的类库来处理了，那就是[compojure](https://github.com/weavejester/compojure)，这就是一个强大的Ring/Clojure路由处理库。

# 零、添加依赖

打开项目根目录下面的`project.clj`，在`dependencies`里面添加`compojure`的依赖`[compojure "1.6.1"]`

然后运行`lein deps`下载依赖包。

# 一、修改代码
修改`core.clj`，最后看起来像下面这样

```clojure
(ns learnweb.core
  (:require [ring.adapter.jetty :refer [run-jetty]]
            [compojure.core :refer [defroutes GET]]    ; (1)
            [compojure.route :refer [not-found]])    ; (2)
  (:gen-class))

(defroutes
  app
  (GET "/" [] "<h1>Hello world!</h1>")
  (GET "/hello/:name" [name] (str "Hello " name))
  (not-found "<h1>Page not found</h1>"))    ; (3)

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (run-jetty app {:port 3000}))    ; (4)
```

<!-- more -->

相比之前的简洁不少。
(1) 引入宏`defroutes`和`GET`。`defroutes`定义路由列表。`GET`定义一条路由规则，说明此条规则只处理http的get请求（其他的有post, head, put, delete, option，详细内容请参考[rfc2616 Method Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)）。

(2) 引用函数`not-found`处理请求不在路由规则的情况

(3) 定义路由规则列表。第一个是`/`，也就是直接输入域名后访问的情况。第二个是`/hello/:name`，匹配所有以/hello/开头的，后面不含有/的所有请求，比如/hello/Kitty，像/hello/abc/defg这样的是不属于这个规则的；然后:name是一个匹配规则，可以匹配/hello/后面的内容，匹配后把匹配到的内容放入一个叫name的变量，以供后面的处理函数使用。最后一个是not-found，也就是不在前面的所有情况都被纳入到这里。

(4) 把run-jetty后面之前的`handle`改成这里定义的`app`

这里看不到{:status 200 :headers {"Content-Type" "text/html"} :body}这样的在代码里面写列的代码了（当然你也可以根据自己的需要做得更加灵活一些），`compojure`会调用`ring`的`response`去处理相应的status和header。打开compojure的not-found(~/.m2/repository/compojure/compojure/1.5.0/compojure-1.5.0.jar!/compojure/route.clj)函数就会看到调用了(status 404)。

修改完之后按`ctrl+c`停止之前启动的服务，然后两次运行`lein run`启动http server。

完整的代码请`git checkout 1.3`

这里用短短的几行定义就实现了非常强大的路由功能，在Clojure的世界里，还有很多强大而简洁的类库。

# 二、表单
## wrap-reload
在开始接下来的内容之前，先来看一下`wrap-reload`这个`ring`的middleware。`ring`提供了非常多的middleware来处理请求。可以把一个请求想像成一个管道，有很多个管道一个接一个，`ring`提供的middleware就是管道：


                   - - - - - - - + - - - - - - + - - - - - - + - - -
    -- request -->   middleware     middleware    middleware    ...   -- response -->
                   - - - - - - - + - - - - - - + - - - - - - + - - -

通过各种的middleware，一个request就转换成了一个response。那么`wrap-reload`这个middleware又是做什么的呢。回想一下之前的代码修改都是修改完成后都要重启服务器才能够看到修改的效果，使用了这个middleware就可以不需要重启服务器，所做的修改刷新页面就可以看到效果，那叫一个方便。在lisp的世界做开发就是这么的爽，还有一个`repl`可以做交互式的开发（Interactive development），比如有一个功能不知道该如何实现，可以边写边在`repl`里面调试，那叫一个爽啊。进入`repl`可以到项目的根目录运行`lein repl`。好了，回头再来看这个`wrap-reload`如何使用，在ns代码块里面把他require进来(1)：

```clojure
(ns learnweb.core
  (:require [ring.adapter.jetty :refer [run-jetty]]
            [ring.middleware.reload :refer [wrap-reload]]    ; (1)
            [ring.middleware.params :refer [wrap-params]]    ; (2)
            [compojure.core :refer [defroutes GET]]
            [compojure.route :refer [not-found]])
  (:gen-class))
```

然后套在app这个handler上面`(wrap-reload #'app)`，注意这里需要使用var quote`#'app`而不是直接使用`app`。来看下`wrap-reload`的说明：Reload namespaces of modified files before the request is passed to the supplied handler。如果使用`app`的话，那么在定义`app`的文件做了修改就不会reload，所以需要`#'app`这种[var-quote](http://clojuredocs.org/clojure.core/var)的方式。

另外在demo里面还使用了`wrap-params`这个middleware，他会把url上面的query string（也就是`http://localhost:3000/?a=1&b=c`里面的a=1&b=c这样的）和表单里面的数据提取出来放到request这个大map的:params里面，方便后面的middleware使用。修改run-jetty那一行，run-jetty的时候会是这样的，在app的前面添加了两个middleware：

```clojure
(run-jetty (wrap-reload (wrap-params #'app)) {:port 3000})
```

## 效果
现在咱们来做一个可以提交点东西的表单页面。有一个用户名输入框，还有一个写更多内容的大输入框，再加上一个提交按钮。页面看起来像下面这样：
![](/img/form.png)
图1

点击提交按钮后会像下面这样：
![](/img/message.png)
图2

## hiccup
实现上面效果之前，再来介绍一个非常强大而简洁的生成html的库，使用这个库我们不需要写一行html代码，可以全部使用clojure代码来生成整个html。

[`hiccup`](https://github.com/weavejester/hiccup)同样也是[`compojure`](https://github.com/weavejester/compojure)的作者[weavejester](https://github.com/weavejester)实现的。先来看下如何使用：

```clojure
(html [:span {:class "foo"} "bar"])
;; 生成的html代码为：<span class="foo">bar</span>
;; hiccup使用vector的方式来做一个html的标签，第一个元素为标签的名字；
;; 第二个参数是一个map，定义了这个标签的属性，这个参数可以省略；
;; 第三个参数是里面的内容，可以是一个字符串，也可以定义其他的元素，像下面这样
;; [:div {:class "outer"} [:div {:class "inner"} "Inner div"]]
```

代码里面使用了一个`html5`函数，这个函数是在`hiccup.page`包里面，用于生成html5标准的doctype头和html元素。`html`函数用于把函数的参数转换为html代码。

所以我们的demo添加一个表单的页面：

```clojure
(def form-html
  (html5
    (html
      [:body
       [:h1 "Hello form"]
       [:form {:action "/message" :method "post"}
        [:div
         [:label {:for "name"} "名字:"]
         [:input {:type "text"
                  :name "name"}]]
        [:div
         [:label {:for "message"} "消息:"]
         [:textarea {:cols 80 :rows 5 :name "message"}]]
        [:div
         [:input {:type "submit" :value "提交"}]]]])))
```

最终生成的html效果就是上面图1一样
![](/img/form_source.png)

## 显示提交的内容
添加一个函数用户显示提交的内容：

```clojure
(defn show-message [params]
  (let [{name "name" message "message"} params]
    (html5
      (html
        [:body
         [:h2 name]
         [:p message]]))))
```

这里同样使用了hiccup来生成html。在let语句里面把params里面保存的表单的值name和message取出来，填充到html内容里面。这个函数的参数params是从何而来呢？那就来看一下`app`的定义：

```clojure
(defroutes
  app
  (GET "/" [] form-html)
  (POST "/message" {params :params} (show-message params))    ; (1)
  (GET "/hello/:name" [name] (str "Hello " name))
  (not-found "<h1>Page not found</h1>"))
```

(1) 这里添加了一个post的处理规则，如果表单提交的页面是/message，那么就用这个路由去处理。第二个参数是一个解构表达式，如果不用解构表达式的话，而且又需要提交的数据应该会像这样`(POST "/message" request (show-message request))`，用一个变量`request`存储请求的所有内容。现在这里是{params :params}，其实就是把request里面的:params提取出来，然后传递给`show-message`函数。

到这里就已经把所有的代码修改完成了，完整的[代码](https://github.com/kevindragon/learn-clojure-web)请：`git checkout 1.3.1`。

*注意：* 别忘记引入`POST`这个`compojure.core`里面的宏
