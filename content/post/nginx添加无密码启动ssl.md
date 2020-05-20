---
title: nginx添加无密码启动ssl
date: 2016-01-10 10:26:28
tags: [nginx]
---

QQ: 380800878, 微信: kittenll

可以参考[官方文档](http://nginx.org/en/docs/http/configuring_https_servers.html)

在nginx启动的时候不用password需要pem和chain.cer文件：

    # cat xxxx.pem chain.cer > xxxx.crt
    # openssl rsa -in need_pwd.key -out nopwd.key

修改nginx配置，在server节点下面添加：

    ssl_certificate /usr/local/certificates/xxxx.crt;
    ssl_certificate_key /usr/local/certificates/nopwd.key;

使用openssl验证服务器上的证书信息：

    $ openssl s_client -showcerts -connect www.baidu.com:443 </dev/null

