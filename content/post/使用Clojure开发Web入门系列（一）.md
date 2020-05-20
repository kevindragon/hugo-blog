---
title: 使用Clojure开发Web入门系列（一）
date: 2016-03-07 22:47:33
tags: [Clojure, Web]
---

对于Clojure语言这里有一些很好的入门教程：

1. [Learn X in Y minutes](https://learnxinyminutes.com/docs/clojure/)
2. [Clojure - Functional Programming for the JVM](http://java.ociweb.com/mark/clojure/article.html)
3. [Clojure koans](https://github.com/functional-koans/clojure-koans)

### 零、需要了解的知识
1. Web
2. [http](https://www.w3.org/Protocols)
3. [Clojure](https://clojure.org)
4. [Leiningen](http://leiningen.org)
5. [HTML](https://www.w3.org/html)
6. Java

### 一、安装
首先Clojure是JVM上的Lisp方言，所以需要安装Java，可以去官方网站下载安装包，最好下载最新的JDK http://www.oracle.com/technetwork/java/javase/downloads/index.html
安装完成之后需要打开命令行窗口看看是否可以执行java命令。

然后安装Leiningen，这个是项目的管理工具，可以管理依赖，编译工程，打包项目等等。可以参照 http://leiningen.org/#install 介绍的方法。

<!-- more -->

### 二、创建项目

```shell
lein new app learnweb
cd learnweb
```

创建完成后目录结构应该像下面这样：

```text
├── CHANGELOG.md
├── LICENSE
├── README.md
├── doc
│   └── intro.md
├── project.clj
├── resources
├── src
│   └── learnweb
│       └── core.clj
└── test
    └── learnweb
        └── core_test.clj
```

首先打开`project.clj`看一眼，这个文件就是定义项目的文件，同样也是使用的`clojure`的语法

```clojure
(defproject learnweb "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.7.0"]]
  :main ^:skip-aot learnweb.core
  :target-path "target/%s"
  :profiles {:uberjar {:aot :all}})
```

第一行里定义了项目名字learnweb，版本0.1.0-SNAPSHOT，这两个在打包的时候分体现在文件名上面。
`:dependencies`包含了项目执行时的所有依赖，后面添加第三方库的时候需要修改这里。
`:main`指明了程序的入口，是包`learnweb.core`的`-main`函数。

`src`目录就是咱们的源代码目录了

现在咱们先来运行一下创建好的项目，使用lein（Leiningen命令）运行：

```clojure
lein run
```

你会得到输出`Hello, World!`。哈哈，入门必经hello world。

完整的代码请checkout到`1.1`分支。
