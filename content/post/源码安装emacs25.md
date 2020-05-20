---
title: 源码安装emacs25
date: 2017-06-21 17:03:29
tags: [Ubuntu, CentOS, Emacs]
---

# 在Ubuntu系列用源码安装Emacs25非常简单

## 下载源码包

```bash
wget 'http://mirrors.ustc.edu.cn/gnu/emacs/emacs-25.2.tar.gz'
tar -xzvf emacs-25.2.tar.gz
```

## 安装编译环境，依赖

首先需要把安装源码的选项打开。找到`Software & Updates`，把`Source code`勾上。另外一个办法是直接修改配置文件`/etc/apt/sources.list`，把前面两个`deb-src`前面的注释打开，然后运行一下`sudo apt update`。

```bash
sudo apt install build-essential
sudo apt build-dep emacs24  # 25版本和24版本的依赖是一样的
```

## 安装Emacs

```bash
cd emacs-25.2
./configure
make
sudo make install
```

## 中文输入法

修改`/etc/environment`，后面添加一行`LC_CTYPE="zh_CN.utf8"`

用`which emacs`找到`emacs`程序的位置，备份`sudo cp emacs emacs.bak`，然后新建一个`emacs`可执行shell脚本，记得`chmod +x emacs`，内容如下：

```bash
#!/bin/sh

LC_CTYPE="zh_CN.utf8" emacs-25.2
```

**重启一下系统**

现在使用Alt+F2，输入emacs，把输入法换成中文就可以输入中文了


# 在CentOS7用源码安装Emacs25也是非常简单

## 下载源码包

```bash
wget 'http://mirrors.ustc.edu.cn/gnu/emacs/emacs-25.3.tar.gz'
tar -xzvf emacs-25.3.tar.gz
```

## 安装编译环境，依赖

```bash
yum install -y ncurses-devel
```

## 安装Emacs

```bash
cd emacs-25.3
./configure
make
sudo make install
```

