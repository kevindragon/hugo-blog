---
title: ClojureScript学习笔记一（Hello World与REPL）
date: 2016-08-22 17:40:54
tags: [ClojureScript, ClojureScript REPL]
---

学习一门编程语言就像新出来的婴儿一样，一个裸体可以看清全部。学习编程语言也是一样的，一开始不可以使用任何一个工具。

ClojureScript继承了Clojure简单和富有表现力的语法，编译后的代码可以运行在浏览器和Node.js上面。

编写hello world程序。新建一个目录`cljs_first_project`，然后还需要建立存放源代码的目录。下载[编译器](https://github.com/clojure/clojurescript/releases/latest){:target="_blank"}，目录结构如下：

    cljs_first_project
    |- build.clj
    |- cljs.jar
    |- src
        |- cljs_first_project
            |- core.cljs

在core.cljs写入如下内容：

```ClojureScript
(ns cljs-first-project.core)
(js/alert "Hello world!")
```

这段代码定义一个命名空间，然后是一个javascript里面的alert弹出框。另外还需要写一个clojure文件来对源代码进行编译，在根目录下新建一个build.clj文件，内容如下：

```ClojureScript
(require 'cljs.build.api)
(cljs.build.api/build "src" {:output-to "out/main.js"})
```

包含`cljs.build.api`命名空间，然后调整`build`函数，此函数接受两个参数，第一个是ClojureScript源代码目录，第二个是是一个map数据结果，这里指定了输出编译后的js的路径。现在使用前面下载的编译器进行编译：

Windows下编译

    java -cp "cljs.jar;src" clojure.main build.clj

*nix下编译

    java -cp cljs.jar:src clojure.main build.clj

编译完成后会在out目录生成main.js，同时还有其他一些目录。要运行main.js还需要一个greet.html，内容如下：

```html
<html>
  <body>
    <script type="text/javascript" src="out/goog/base.js"></script>
    <script type="text/javascript" src="out/main.js"></script>
    <script type="text/javascript">
      goog.require("cljs_first_project.core")
    </script>
  </body>
</html>
```

用浏览器打开greet.html就可以看到alert框了。第一个ClojureScript编写编译完成！

再来一点高级功能，把`goog/base.js`依赖从html去掉，把`goog.require("cljs_first_project")`也从html里面去掉，只需要包含main.js就可以让程序跑起来。只需要对build.clj修改一下，内容修改如下：

```ClojureScript
(require 'cljs.build.api)
(cljs.build.api/build
  "src"
  {:main 'cljs-first-project.core
  :output-to "out/main.js"})
```

greet.html的内容如下：

```html
<html>
  <body>
    <script type="text/javascript" src="out/main.js"></script>
  </body>
</html>
```

打开greet.html，效果跟之前一样。

另外一个高级功能是当编辑源代码的时候不需要手动多次编译，另外在根目录下再写一个watch.clj文件，内容如下：

```ClojureScript
(require 'cljs.build.api)
(cljs.build.api/watch
  "src"
  {:main 'cljs-first-project.core
  :output-to "out/main.js"})
```

只是把调用build的地方改成调用watch函数即可。运行`java -cp "cljs.jar;src" clojure.main watch.clj`（*nix运行`java -cp cljs.jar:src clojure.main watch.clj`），此时若修改源文件core.cljs，编译器会自动检测到文件的修改，然后自动进行编译，自动编译后在浏览器刷新即可看到效果。

既然是lisp系，怎么可以少得了REPL(Read-Eval-Print-Loop)。REPL可以让我们直接把代码片段推送到浏览器运行，只需要在REPL里面写上代码，然后立马可以在浏览器看到效果，这个Live Codiing的基础。

同样的要想运行repl也需要clojure的支持，在根目录下新建一个repl.clj，内容如下：

```ClojureScript
(require 'cljs.repl)
(require 'cljs.build.api)
(require 'cljs.repl.browser)
    
(cljs.build.api/build
  "src"
  {:main 'cljs-first-project.core
   :output-to "out/main.js"
   :verbose true})
    
(cljs.repl/repl
  (cljs.repl.browser/repl-env)
  :watch "src"
  :output-dir "out")
```

调整的build函数还是没有变，后面添加一个`cljs.repl/repl`函数调用。·cljs.repl.browser/repl-env·这个函数定义了一些默认配置，比如打开一个9000端口作为http的服务端口。`cljs.repl/repl`函数作了下面几件事情：

1. 监听源代码的修改
2. 编译修改的源代码
3. 监听9000端口提供http服务
4. 打开一个prompt等待输入

修改core.cljs内容如下：

```ClojureScript
(ns cljs-first-project.core
  (:require [clojure.browser.repl :as repl]))
    
(defonce conn
  (repl/connect "http://localhost:9000/repl"))
    
(js/alert "Hello world!")
```

可以看到这里初始化了一个connection，连接到9000端口的/repl，这里使用defonce是只初始化一次，之后的每次修改都不再连接服务端的REPL。

Ok再来运行repl.clj

    java -cp "cljs.jar;src" clojure.main repl.clj

*unix

    java -cp cljs.jar:src clojure.main repl.clj

第一编译需要等待一小会，因为需要编译源代码启动http服务，最后会输出等待浏览器连接的提示：

    Waiting for browser to connect ...

现在用浏览器打开`http://localhost:9000/greet.html`，浏览器会显示一个Hello world!的弹出框。这时候命令行会出现一个输入的提示符，如下：

    Watch compilation log available at: out\watch.log
    To quit, type: :cljs/quit
    cljs.user=> 

输入`(js/alert "Hello World From REPL!")`回车，浏览器就是看到这段代码的效果。

