---
title: linux下使用极点五笔
date: 2011-11-26 21:40:39
tags: [Linux]
---

QQ: 380800878, 微信: kittenll

像我我种原始人还在用着五笔，没办法，已经习惯了，如果没有五笔我的世界将全是英文，但我英文又不好，没办法装五笔。以前是用ibus自带的五笔，在ubuntu下面有的时候系统启动的时候切换到五笔，五笔的图标就是一个stop图标----shit，装上Linux Mint 12改用scim下面的极点五笔，以前在windows下就用过。

安装方法：

首先安装scim

    $ sudo apt-get install scim

装中文支持包

    $ sudo apt-get install scim-tables-zh

下载[极点五笔](http://www.linuxidc.com/upload/2008_08/08081807103223.zip)，把*.bin 复件到 /usr/share/scim/tables/下面

    sudo cp ***.bin /usr/share/scim/tables/

重启scim(如果已经启动)，设置...

    sudo pkill scim
    scim-setup

