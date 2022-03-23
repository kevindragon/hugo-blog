---
title: yum通过ssh代理安装
date: 2018-07-07 18:48:55
tags: [yum, proxy, ssh]
---

在机房的机器有时候请第三方服务公司安装完系统后没有检查网络，结果上不了网，最后连安装个gcc都不行。没有办法，只能通过代理安装，方法很简单。

需要有一台可以上网的linux机器，假设是A机器IP是`192.168.1.100`，另外一台不能上网的是B机器IP是`192.168.2.200`

用Xshell（或者其他ssh客户端）登录B机器，使用下面的命令登录到A机器：

```bash
ssh -D 1080 root@192.168.1.100
```

输入A机器的密码，这时候ssh会开一个1080监听端口，所有发往这个端口的请求被转发到ssh所连接的机器，然后通过所连接的机器转发请求，这样就实现了ssh代理。ssh现在支持的协议是socks4和socks5。

然后在另外一个终端登录B机器，修改`/etc/yum.conf`文件，在文件最后添加下面一行：

```bash
proxy=socks5h://localhost:1080
```

保存退出，这时候你再运行`yum`命令，完全不障碍。

其实都已经开了1080端口了，其他任何支持代理设置的程序都可以使用此`socks5h://localhost:1080`进行代理，比如：

```bash
curl -x socks5h://localhost:1080 http://www.baidu.com
```

如果其他的方式，比如ftp之类的，也需要使用代码的话，可以把下面的内容保存到一个文件`proxy.sh`，然后使用`source proxy.sh`加载一下

```shell
#!/bin/sh

# add follows to the end (set proxy settings to the environment variables)
MY_PROXY_URL="socks5h://localhost:1080"

HTTP_PROXY=$MY_PROXY_URL
HTTPS_PROXY=$MY_PROXY_URL
FTP_PROXY=$MY_PROXY_URL
http_proxy=$MY_PROXY_URL
https_proxy=$MY_PROXY_URL
ftp_proxy=$MY_PROXY_URL

export HTTP_PROXY HTTPS_PROXY FTP_PROXY http_proxy https_proxy ftp_proxy
```

## 例子

### pip

pip使用代理的时候有一点区别，需要把上面`proxy.sh`里面的协议`socks5h`修改成`socks5`才可以使用。

Enjoy ^_^.
