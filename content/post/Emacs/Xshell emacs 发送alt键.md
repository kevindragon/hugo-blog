---
title: Xshell emacs 发送alt键
date: 2016-01-21 11:40:54
tags: [Xshell, Emacs]
---

我比较喜欢使用Xshell来连接服务器，原因有下面几个：

1. 对于个人用户是免费的；
2. 配色不需要过多的配置，有5、6种可选的比较好看的配色。在比较亮的工作环境可以选择白色背景，暗的工作环境可以选择黑色背景；
3. 可以使用透明的窗体，看起来比较酷；
4. 还有其他，balabalabala

默认在Xshell里面使用emacs的时候，Alt键是不怎么好用的，其实是没有发送到服务器端的，所以需要对其他做一个设置：

    File -> Properties -> Terminal -> Keyboard
    
把`Use Alt as Meta key`这个选项勾上
好了，现在可以使用`Alt+x`来在emacs里面使用command了。

以上备忘
