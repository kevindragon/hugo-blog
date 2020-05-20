---
title: Emacs下Python开发配置
date: 2018-03-20 13:56:32
tags: [Emacs, Python]
---

Python开发环境配置其实比较简单，而我一直没有配置好是参照了网上各种各样的配置，导致了非常混乱的配置，这次自己亲自测试安装自己需要的Emacs包来配置。

<!-- more -->

# 需要的Emacs包

* [Elpy](https://github.com/jorgenschaefer/elpy)
* [company-mode](https://github.com/company-mode/company-mode)
* [anaconda-mode](https://github.com/proofit404/anaconda-mode)

就这三个包就可以了。Elpy是一个很强大的Python集成开发环境了，后面两个其实就是做自动补全用的。如果启动Emacs后，执行`M-x elpy-config`没有办法出来配置的界面，可能是Emacs没有识别到python, pip, conda等命令，可以安装下面的包来解决：

* [exec-path-from-shell](https://github.com/purcell/exec-path-from-shell)

# 需要的Python包

[Elpy](https://github.com/jorgenschaefer/elpy)的github页面已经列出来了：

```
# Either of these
pip install rope
pip install jedi
# flake8 for code checks
pip install flake8
# and autopep8 for automatic PEP8 formatting
pip install autopep8
# and yapf for code formatting
pip install yapf
```

另外`anaconda-mode`需要安装`setuptools`

```
pip install setuptools
```

# Emacs配置

下面是最简单的配置，`~/.emacs.d/init.el`

```elisp
(require 'package)
(add-to-list 'package-archives
             '("melpa-stable" . "https://stable.melpa.org/packages/"))
(package-initialize)
(package-refresh-contents)

(defvar my-packages
  '(elpy
    company
    anaconda-mode
    exec-path-from-shell))

(mapc #'(lambda (package)
    (unless (package-installed-p package)
      (package-install package)))
      my-packages)

(exec-path-from-shell-initialize)

(elpy-enable)
(add-hook 'after-init-hook 'global-company-mode)
(add-hook 'python-mode-hook 'anaconda-mode)
(add-hook 'python-mode-hook 'anaconda-eldoc-mode)
```

# 常用按键

| Keybinding | Function                       | Description                                                                                            |
|------------|--------------------------------|--------------------------------------------------------------------------------------------------------|
| M-.        | anaconda-mode-find-definitions | 跳转到定义处。如果不使用anaconda-mode，则是绑定到elpy的elpy-goto-definition，elpy有时候工作得不是很好 |
| M-?        | anaconda-mode-show-doc         | 在另外一个window中显示光标当前所在位置符号的文档                                                       |
| M-,        | anaconda-mode-find-assignments | 跳转到变量赋值位置                                                                                     |
| M-r        | anaconda-mode-find-references  | 在另外一个window中显示光标当前所在位置变量的所有引用                                                   |
| M-*        | anaconda-mode-go-back          | 返回上一个位置                                                                                                       |

show一下我的[.emacs.d](https://github.com/kevindragon/.emacs.d)配置，欢迎交流。
