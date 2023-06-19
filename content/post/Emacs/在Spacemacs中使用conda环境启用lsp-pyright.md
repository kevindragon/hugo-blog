---
title: "在Spacemacs中使用conda环境启用lsp-pyright"
date: "2023-06-19T22:15:08+08:00"
tags: ["Emacs", "Spacemacs", "Python"]
categories: [编辑器, 编程语言]
---


如果你是Emacs的用户，那么肯定听说过Spacemacs，以开箱即用而出名，界面也非常的漂亮。放弃了一段时间的Emacs，之前都是自己写的配置，没有时间维护，便尝试了一下Spacemacs。

在使用Spacemacs开发Python的时候，启动`lsp-pyright`对Python的支持，`lsp-pyright`可以识别virtualenv虚拟环境，暂时还没有对conda env的支持。好在还是有办法设置python程序的路径。

实现方式用到了几个关键词方法：

1. dotspacemacs-configuration-layers列表添加python、lsp以及conda
2. directory local variable
3. with-eval-after-load

下面解释每个点的配置

# Spacemacs layers

启动python和lsp，这两个layer不需要做过多的设置。

启动conda的时候需要设置conda-anaconda-home变量，如下：

```lisp
(conda :variables conda-anaconda-home "D:/path/to/miniconda3")
```

python解释器我使用的是miniconda，可以修改成自己对应的路径。

# Directory local variable

Emacs支持在项目目录创建一个 `.dir-locals.el` 文件，来创建针对项目文件的本地变量，而不会覆盖全局的变量。这样多个项目之间就不会相互影响了。这块可以参考[官方文档](https://www.gnu.org/software/emacs/manual/html_node/emacs/Directory-Variables.html)。

在我们的Python项目根目录下创建一个 `.dir-locals.el` 文件，写入以下内容：

```lisp
((python-mode . ((eval . (with-eval-after-load 'lsp-pyright
                           (progn
                             (lsp-register-custom-settings
                              `(("python.pythonPath" "D:/path/to/miniconda3/envs/stringle_pro/python.exe"))))
                           )))))
```

1. 首先`python-mode` 指定只有在python模式下后面的代码才会执行；
2. (eval . func_call)是一个cons pair，eval函数相当于是标记后面的fanc_call是一个可以执行的S表达式；

如果要设置指定模式下的变量值，则使用下面的代码：

```lisp
((python-mode . ((variable-name variable-value))))
```

# with-eval-after-load

`with-eval-after-load` 是Emacs内置的一个宏，定义在 `subr.el`。

这个地方是很需要注意代码运行的顺序与时机的。在Emacs打开python代码的时候会调用 `lsp` 启动lsp-mode，lsp会询问使用的lsp后端，python的lsp后端有非常的多，pyls, mspy等。我选择的是lsp-pyright。然后lsp根据设置的python lsp后端，加载lsp-pyright的代码。

这时上面的with-eval-after-load就会起作用了，在加载完lsp-pyright.el的代码后就会执行with-eval-after-load里面的代码。把 `python.pythonPath` 这个lsp变量设置完成后。

lsp就会调用lsp-pyright里面lsp相关函数，启动pyright后端。

# 总结

本文说明了如何在spacemacs下启用lsp来支持在conda env虚拟环境下开发python的配置工作，需要对Emacs和lsp的工作原理有一定的认识与了解。
