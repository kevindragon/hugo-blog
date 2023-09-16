---
title: "Linux下文件拆分，多个文件打包后拆分"
date: "2023-09-16T15:04:16+08:00"
tags: ["Linux"]
categories: ["压缩"]
---

处理和传输大文件，以及处理和传输大量小文件一直很有挑战。特别是在多个存储介质，以及网络之间传输的时候。大文件无法放入一张4.7G的光盘当中；大量小文件传输需要系统打开非常多文件描述符，会造成资源浪费的情况。

本文介绍如何应对这两种情况，使用文件分割、文件压缩以及压缩时分割的方法处理文件，然后如何还原分割后的文件。

## 分割文件

在Linux下分割文件最常用的方法之一是使用`split`命令。`split`命令可以按固定大小分割，还可以按分割后的数量进行分割

假如我们有一个50M的文件，我们需要把它分成5个10M的部分：

```bash
# 使用dd命令创建一个50M的test.txt文件
dd if=/dev/zero of=test.txt bs=50M count=1

# 可以看到文件创建成功了
# $ls -alth test.txt
# -rw-r--r-- 1 Kevin None 50M Sep 14 10:34 test.txt

split -b 10M test.txt test_
```

上面的`split`命令把50M的test.txt文件拆成了5个以 test_ 开头的10M大小的文件，查看分割后的文件列表：

```bash
$ ls -alth test_*
-rw-r--r-- 1 Kevin None 10M Sep 14 10:36 test_ae
-rw-r--r-- 1 Kevin None 10M Sep 14 10:36 test_ab
-rw-r--r-- 1 Kevin None 10M Sep 14 10:36 test_ac
-rw-r--r-- 1 Kevin None 10M Sep 14 10:36 test_ad
-rw-r--r-- 1 Kevin None 10M Sep 14 10:36 test_aa
```

解释一下`split`命令

<aside>
💡 Linux man手册的解释如下：
Output pieces of FILE to PREFIXaa, PREFIXab, ...; default size is 1000 lines, and default PREFIX is 'x'.

</aside>

把文件拆分成参数指定的大小，默认输出为 PREFIXaa，PREFIXab 格式，默认以 aa, ab, ac 来创建不同的文件名。命令的格式如下：

```bash
split [OPTION]... [FILE [PREFIX]]
```

`-b` 或者 `--bytes` 参数指定生成分割后的文件大小，可以使用 K,M,G,T,P,E,Z,Y （1024的倍数）为单位进行指定，或者 KB,MB,… （这是1000的倍数）。

`-d` 参数指定使用数字方式生成文件，例如 test_01, test_02

还可以按分割后的文件数量进行分割：

```bash
split -d -n 3 test.txt test_
```

分割后

```bash
$ ls -alth test_*
-rw-r--r-- 1 Kevin None 17M Sep 14 10:57 test_02
-rw-r--r-- 1 Kevin None 17M Sep 14 10:57 test_01
-rw-r--r-- 1 Kevin None 17M Sep 14 10:57 test_00
```

## 合并文件

使用命令cat就可以把分割的多个文件进行合并了

```bash
cat test_* > another_test.txt
```

查看合并后的文件

```bash
$ ls -alth another_test.txt
-rw-r--r-- 1 Kevin None 50M Sep 14 10:59 another_test.txt

# 使用md5sum验证文件内容
$ md5sum test.txt
25e317773f308e446cc84c503a6d1f85 *test.txt

$ md5sum another_test.txt
25e317773f308e446cc84c503a6d1f85 *another_test.txt
```

## 使用zip在压缩的时候同时进行分割

zip命令是Linux中的一个命令行工具，它允许我们创建文件和目录的存档。除此之外，它还提供了许多用于操作归档的功能。

### 安装

要在Ubuntu/Debian中安装zip命令行工具，我们可以使用包管理器apt-get（或者apt）：

```bash
$ sudo apt-get update
$ sudo apt-get install -y zip
```

此外，我们可以使用软件包管理器yum在基于RHEL的Linux中安装zip命令行工具：

```bash
$ sudo yum update
$ sudo yum install -y zip
```

zip命令语法格式：

```bash
zip [options] [zipfile [files...]] [-xi list]
```

基础用法：

```bash
$ zip test.zip test_001.txt test_002.txt
  adding: test_001.txt (deflated 100%)
  adding: test_002.txt (deflated 100%)
```

重点来了，在使用zip归档和压缩文件的时候，同时把归档的文件进行分割，这时需要使用到`-s`参数，意思就是使用split命令进行分割

```bash
$ zip -s 200M test.zip redis-6.2.6.tar neo4j-4.4.15.tar
  adding: redis-6.2.6.tar (deflated 66%)
  adding: neo4j-4.4.15.tar (deflated 37%)
```

查看归档及分割后的文件列表

```bash
$ ls -alth test.*
-rw-r--r-- 1 Kevin None 185M Sep 14 11:14 test.zip
-rw-r--r-- 1 Kevin None 200M Sep 14 11:13 test.z01
```

### 解压分割后的文件

首先把多个文件合并成一个zip文件

```bash
zip -FF test.zip --out test_single.zip
```

然后使用unzip命令进行解压

```bash
unzip test_single.zip -d ./temp
```

## 总结

在本文中，我们学习了将文件拆分为多个部分，然后将各个部分重新连接在一起。接下来，我们描述了如何应用流行的压缩实用程序zip和unzip来处理拆分的归档。

## 公众号

欢迎关注我的公众号，同步更新

![木木小小孩](/img/qrcode_for_gh.jpg)
