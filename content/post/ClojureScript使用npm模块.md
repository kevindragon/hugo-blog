---
title: ClojureScript使用npm模块
date: 2018-07-16 10:18:54
tags: [ClojureScript, npm]
---

Javascript现在的生态系统一点也不输Java的生态，而且还在以高速的发展事态占领了企业招聘的热门技术榜单。Javascript社区不仅充满激情、创新，而且非常具有活力，同时也是一个非常开放的社区，从其他的编程语言借鉴了很多优点。

Javascript社区如此丰富的模块与库，作为ClojureScript的使用者也不得不来使用npm的模块。下面简单介绍一下如何在ClojureScript里面使用npm的module，这里用react为例说明如何使用。

<!-- more -->

# Requirement

1. leiningen
2. figwheel
3. cljsbuild
4. React
5. ReactDom

# Create project

```
lein new figwheel hello-world
```

直接使用figwheel的templaate来创建项目

# Add dependencies

修改`project.clj`，在`cljsbuild`修改id为dev的`:compiler`内容，添加如下代码：

```clojure
:closure-defines {process.env/NODE_ENV "development"}
:install-deps true
:npm-deps {:react "16.4.1"
           :react-dom "16.4.1"}
```

id为min的`:compiler`内容，只是修改了`:closure-defines`：

```clojure
:closure-defines {process.env/NODE_ENV "production"}
:install-deps true
:npm-deps {:react "16.4.1"
           :react-dom "16.4.1"}
```

`:closure-defines`定义的内容，可以直接在cljs里面进行读取。

# Source code

ClojureScript代码如下：

```clojure
(ns hello-world.core
  (:require [react :refer [createElement]]
            [react-dom :refer [render]]))

(enable-console-print!)

(js/console.log "Node enviornment is" process.env/NODE_ENV)

(def appDiv (.getElementById js/document "app"))

(render
 (createElement "h1" nil "Hello, World!")
 appDiv)
```

require部分的使用跟cljs的包一样，可以直接从一个包require函数以及对象，剩下的部分就跟js使用react/react-dom的方式一样，调用render函数渲染元素到指定的html节点下面。

# Run

```
lein figwheel
```

使用此命令运行代码，编译完成后figwheel会自动打开浏览器访问：[`http://localhost:3449/index.html`](http://localhost:3449/index.html)，完成后你会看到下面的内容：

![](/img/cljs_npm_react_module.png)
