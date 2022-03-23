---
title: nagios使用check_nrpe调用子命令 
date: 2016-01-24 22:50:54
tags: [nagios]
---

以http为例子，如果单独使用的话可以：

    $ ./libexec/check_http -H 192.168.1.1

通过check_nrpe使用nrpe检查http

    $ ./libexec/check_nrpe -H 192.168.2.142 -c check_http

调用上面的命令需要客户机安装了nrpe，并且配置了xinted，而且防火墙要开启对应的端口

以上备忘
