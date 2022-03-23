---
title: spacemacs配置figwheel环境
date: 2016-09-01 09:04:01
tags: [ClojureScript, Emacs]
categories: [编辑器]
---

## 0. ...

`clojure`和`clojurescript`不像ruby on rails和django那样all in one的模式，只需要很少的配置就可以建立项目开始开发。clojure系的东西都是library，什么用着顺手就把什么加进来，然后稍做配置再使用。这跟emacs一样，完全私人定制，每个人都有自己的喜欢的extension，这应该是lisp系的习惯吧。所以在使用lisp系的东西的时候没有什么框架能一桶江湖。不过在`clojurescript`的开发当中`figwheel`还是少不了的，代码热加载功能太赞了，边写代码边看效果，不需要刷新浏览器，这就是传说的living coding。另外使用集成的IDE工具后，所有东西都有提示，敲两个字母就自动补全了，对于学习一门语言来说的确不是什么好事。闲话不多说，下面来配置一下livind coding的环境，使用`spacemacs`加上`cider`，`info-clojure`两个扩展。

了解更多`figwheel`请查看其[github](https://github.com/bhauman/lein-figwheel)

## 1. 配置emacs

在`spacemacs`的配置文件`.spacemacs`的`dotspacemacs-configuration-layers`添加`clojure`

    dotspacemacs-configuration-layers '(clojure)

另外还需要`inf-clojure`，由于这个扩展不在`spacemacs`的layer列表里面，需要另外添加，在`dotspacemacs-additional-packages`添加即可

    dotspacemacs-additional-packages '(inf-clojure)

重启`spacemacs`或者按`SPC-f-e-R`执行`dotspacemacs/sync-configuration-layers`

然后还需要添加一个个启动figwheel的函数，在`.spacemacs`后面添加下面的函数

    (defun figwheel-repl ()
      (interactive)
      (run-clojure "lein figwheel"))

在后面按`c-x c-e`执行上面函数的定义

## 2. 建立项目，修改project.clj

现在新建一个项目叫hello

    lein new app hello

修改`project.clj`为如下内容

    (defproject hello "0.1.0-SNAPSHOT"
      :description "FIXME: write description"
      :url "http://example.com/FIXME"
      :license {:name "Eclipse Public License"
                :url "http://www.eclipse.org/legal/epl-v10.html"}
      :dependencies [[org.clojure/clojure "1.8.0"]]
      :plugins [[lein-figwheel "0.5.6"]]  ;(1)
      :main ^:skip-aot hello.core
      :target-path "target/%s"

      ;;(2)
      :cljsbuild {:builds [{:id "dev"
                            :source-paths ["src/cljs"]
                            :figwheel true
                            :compiler {:main "hello.core"
                                       :asset-path "js/out"
                                       :output-to "resources/public/js/app.js"
                                       :output-dir "resources/public/js/out"}}]}

      :profiles {:uberjar {:aot :all}})

添加了两处：
(1) 添加`figwheel`的支持
(2) 添加`cljsbuild`配置，这里需要注意的就是`:figwheel true`这一行，这是在编译的js文件里面添加`figwheel`的支持。还有`:asset-path "js/out"`这一行，不然加载google closure library会找不到对应的路径。

## 3. 添加html

有了编译配置文件，还需要一个html页面来展示效果，添加一个`index.html`到`resources/public`目录

    <html>
      <body>
        <script src="js/app.js"></script>
      </body>
    </html>

## 4. core.cljs

添加cljs源代码，需要在`src`目录下面新建一个目录`cljs`，由于上面的`project.clj`指定的`:main`是`hello.core`，所以还需要在`src/cljs`目录下面建一个`hello`目录以跟包名匹配。然后在`src/cljs/hello`目录下建一个`core.cljs`文件，内容如下：

    (ns hello.core)

    (enable-console-print!)

    (println "Hello World!")

## 5. run

用emacs打开hello下面的clj或者cljs文件，然后`M-x figwheel-repl RET`，会出来一个`*inf-clojure*`的buffer，稍微等待一会就会看到下面的内容

    Figwheel: Cutting some fruit, just a sec ...
    Figwheel: Validating the configuration found in project.clj
    Figwheel: Configuration Valid :)
    Figwheel: Starting server at http://0.0.0.0:3449
    Figwheel: Watching build - dev
    Compiling "resources/public/js/app.js" from ["src/cljs"]...
    Successfully compiled "resources/public/js/app.js" in 6.043 seconds.
    Launching ClojureScript REPL for build: dev
    Figwheel Controls:
              (stop-autobuild)                ;; stops Figwheel autobuilder
              (start-autobuild [id ...])      ;; starts autobuilder focused on optional ids
              (switch-to-build id ...)        ;; switches autobuilder to different build
              (reset-autobuild)               ;; stops, cleans, and starts autobuilder
              (reload-config)                 ;; reloads build config and resets autobuild
              (build-once [id ...])           ;; builds source one time
              (clean-builds [id ..])          ;; deletes compiled cljs target files
              (print-config [id ...])         ;; prints out build configurations
              (fig-status)                    ;; displays current state of system
      Switch REPL build focus:
              :cljs/quit                      ;; allows you to switch REPL to another build
        Docs: (doc function-name-here)
        Exit: Control+C or :cljs/quit
     Results: Stored in vars *1, *2, *3, *e holds last exception object
    Prompt will show when Figwheel connects to your application

然后启动浏览器，打开`http://localhost:3499/index.html`，把浏览器的console打开就会看到有`Hello World!`的打印输出。`*inf-clojure*`的后面会输出：

    To quit, type: :cljs/quit
    cljs.user=>

输入一个`(js/alert "Hello World!")`试试看...

如果不是在`emacs`使用的话，在命令行使用下面的命令启动`figwheel`

    lein figwheel


QQ: 380800878, 微信: kittenll
