---
title: Racket开发环境Emacs配置
date: 2017-06-23 16:59:43
tags: [Scheme, Racket, Emacs]
---

首先到Racket官网下载安装包，然后你还需要Emacs。

在Emacs安装`rakcet-mode`，`M-x`输入`package-install`，再输入`racket-mode`。安装完后在 init.el 初始化文件添加下面配置：

```elisp
(when (package-installed-p 'racket-mode)
  (setq racket-racket-program "C:/Program Files/Racket/Racket.exe")
  (setq racket-raco-program "C:/Program Files/Racket/raco.exe"))
```

把 `C:/Program Files/Racket/` 改成对应的自己机器对应的路径即可。

如果是Linux或者maxOS系统，相应的程序应该是`racket`和`raco`。
