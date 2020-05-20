---
title: Make your own Emacs package
date: 2017-07-24 16:13:11
tags: [Emacs]
categories: [Emacs]
---

Make you own Emacs package.
创建自己的Emacs包。

在[Elpa](https://elpa.gnu.org/)、[MELPA](https://melpa.org/)上面有许多的包，可以做各种各样的事情，但有可能你找不到一个合适你的某个问题的包，这时候可以自己动手写一个package，分离到MEELPA。

其实写一个package是一个比较简单的事情，但是又比较麻烦，麻烦在于有很多的约定需要了解，下面我们来动手写一个简单的闹钟包，参考了[alarm.el](https://www.emacswiki.org/emacs/alarm.el)。恰好我的包也叫做[alarm.el](https://github.com/kevindragon/alerm.el)，如有疑问请提出。

<!-- more -->

## 准备

开发阶段使用`package-insstall-file`来安装并且测试包是否正确，这个命令可以安装单个文件`something.el`，也可以安装一个`tar`包，这里主要有两个文件，所以需要打成tar包安装。

## 目录结构

alarm-20170724.1502
|-alarm-pkg.el
|-alarm.el
|-alarm.wav

1. 目录名的命名规则是`name`-`version`
2. alarm-pkg.el 是给emacs的package提供信息的一个文件
3. alarm.el 程序主要内容在这个文件
4. alarm.wav 闹钟响的时候播放这个文件，还可以使用.au文件

## alarm-pkg.el

```elisp
(define-package
  "alarm"
  "20170724.1502"
  "A package to alarm yourself at some time.")
```

这个文件比较简单，只是调用了`define-package`来定义你的包信息，此函数的签名如下:

```elisp
(defun define-package (_name-string _version-string
                                    &optional _docstring _requirements
                                    &rest _extra-properties)
  (error "Don't call me!"))
```

第一个参数是包名，第二个参数是版本号，第三个参数可以是一个长字符串提供说明文档

## alarm.el

这个文件的约定比较多，先看一下这个文件的头部

```elisp
;;; alarm.el --- Alarm

;; Copyright (C) 2017, Emacs guys, all rights reserved.

;; Author: Kevim Jiang <wenlin1988@126.com>
;; Version: 20170724.1502
;; Keywords: alarm
;; Maintainer: Kevin Jaing <wenlin1988@126.com>
;; Created: 2017-07-24 12:00:00

;;; Commentary:

;; Set an alarm with `alarm-clock', and enter the time;
;; cancel the alarm with `alarm-clock-cancel'

;;; Code:
```

每一个段落以空白行隔开。第一行是文件名和一个简短的说明，而且开头以`;;;`三个分号注释。`Commentary`一段可以是长一些的说明文档，以及使用说明。最后是`;;; Code:`段，接下面就是真正的代码内容。可以使用`checkdoc`来检查头部的文档是否正确。还可以使用`package-lint`包的`package-lint-current-buffer`命令检查。

最后需要：

```elisp
(provide 'alarm)

;;; alarm.el ends here
```

完整的`alarm.el`如下：

```elisp
;;; alarm.el --- Alarm

;; Copyright (C) 2017, Emacs guys, all rights reserved.

;; Author: Kevim Jiang <wenlin1988@126.com>
;; Version: 20170724.1502
;; Keywords: alarm
;; Maintainer: Kevin Jaing <wenlin1988@126.com>
;; Created: 2017-07-24 12:00:00

;;; Commentary:

;; require alarm in you init.el: `(require 'alarm)'
;; Set an alarm with `alarm-clock', and enter the time;
;; cancel the alarm with `alarm-clock-cancel'

;;; Code:

(defvar alarm-version "20170724.1502")

(defvar alarm-clock-timer nil
  "Keep timer so that the user can cancel the alerm.")

(defun alarm-ding3 ()
  "Play 3 times ding."
  (dotimes (n 3)
    (play-sound-file
     (concat package-user-dir
             "/alarm-" alarm-version
             "/alarm.wav"))))

;;;###autoload
(defun alarm-clock-cancel ()
  "Cancel the alarm clock."
  (interactive)
  (cancel-timer alarm-clock-timer))

;;;###autoload
(defun alarm-clock ()
  "Set an alarm.
The time format is the same accepted by `run-at-time'.
For example 12:00am."
  (interactive)
  (let ((time (read-string "Time(example, 12:00am): ")))
    (setq alarm-clock-timer (run-at-time time nil 'alarm-ding3))))

(provide 'alarm)

;;; alarm.el ends here
```

## 打包

`tar -cvf alarm-20170724.1502.tar alarm-20170724.1502`

然后使用`package-install-file`安装此tar包，重启emacs后可以使用`alarm-clock`来设置一个闹钟。记得在`init.el`文件使用`(require 'alarm)`，小伙伴快来试试吧！

## 分享到melpa

首先得在github上clone melpa仓库，然后添加一个以包名命令的文件到`recipes`目录下，这里的文件是`alarm`：

```elisp
(alarm
 :repo "kevindragon/alerm.el"
 :fetcher github)
```

提交，然后提pull request，等待接收。

希望我的[提交](https://github.com/melpa/melpa/pull/4877)可以顺利通过。
