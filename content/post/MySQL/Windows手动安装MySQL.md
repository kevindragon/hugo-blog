---
title: "Windows手动安装MySQL"
date: 2022-08-14T08:55:39+08:00
tags: [MySQL, MySQL安装, MySQL手动安装]
draft: true
---

在Windows下安装MySQL常见的方式，一般是下载msi的安装包，直接下一步、下一步就搞定了。这次想看看有没有可能手动安装二进制分发包，到官网查了一下，还真有非常全面的手动安装MySQL的教程。

# 下载

首先需要下载二进制的分发包，进行到页面 https://dev.mysql.com/downloads/mysql/ 选择ZIP Achive下载。

# 解压

解压下载的二进制分发包，官方推荐解压到`c:\mysql`目录，解压到其他也是可以的。如果不是`c:\mysql`目录，等会需要在option文件指定安装的目录。我一般会安装在`D:\Software\mysql`目录。

> 使用msi安装方式会把MySQL安装在`C:\Program Files\MySQL`目录。

# 创建`Option`文件

