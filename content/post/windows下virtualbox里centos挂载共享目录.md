---
title: windows下virtualbox里centos挂载共享目录
date: 2017-06-28 09:33:39
tags: [virtualbox]
---

## 环境

Windows 10
Virtualbox 5.1.18
CentOS 6.6 minimal

## 依赖

virtualbox的增强功能需要用到linux的源码，所以需要先把linux的开发包装上，还有编译工具。还需要把开发包目录链接到`/usr/src/linux`：

```bash
yum install kernel-devel kernel-devel-$(uname -r) gcc perl
ln -s /usr/src/kernels/2.6.32-696.3.2.el6.x86_64 /usr/src/linux
```

Ubuntu

```bash
sudo apt-get install linux-headers-$(uname -r)
```

<!-- more -->

## 安装

首先得把centos安装好，然后挂载好增强功能的iso文件（VBoxGuestAdditions.iso）--选择菜单`设备`->`安装增强功能`

![安装增强功能](/img/virtualbox/install_guest_additions.png)

然后在centos挂载iso文件，执行安装程序：

```bash
mkdir /media/cdrom
mount /dev/cdrom /media/cdrom
cd /media/cdrom
./VBoxLinuxAdditions.run
```

如果没有error说明安装成功，如果有关于X的错误，有可能是你没有安装X环境，如果只是使用命令行，这个也不用管。安装完增强功能后最好把centos重启一下。

## 配置共享目录

首先设置windows到centos的共享目录，在virtualbox的`设置`->`共享文件夹`添加对应的目录

![设置共享目录](/img/virtualbox/shared_folder.png)

这里有一个`共享文件夹名称`，这个名称可以直接在centos里面使用。然后在centos挂载这个目录:

```bash
cd ~
mkdir folder
mount -t vboxsf workspace folder
```

`mount`只有有root权限的才可以执行，如果是普通用户使用`sudo`，修改`/etc/sudoers`，把下面一行的注释去掉：

```bash
# %wheel	ALL=(ALL)	ALL
# -> 去掉注释变成下面的
%wheel	ALL=(ALL)	ALL
```

然后把普通用户（我这里是www用户）加入到`wheel`组即可使用`sudo`

```bash
usermod -a -G wheel www
```

到这已经把共享目录挂载完成了。如果不想每次都手动挂载，就把这个挂载写到`/etc/fstab`文件里，每次开机自动挂载：

在`/etc/fstab`后面添加一行:

`workspace /home/www/workspace  vboxsf defaults 0 0`
