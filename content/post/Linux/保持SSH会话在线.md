---
title: "保持SSH会话在线"
date: 2022-11-29T20:53:42+08:00
tags: [SSH, SSH会话, SSH客户端]
categories: [Linux]
---

# 保持SSH会话在线

## 1. 概览

有多少次，我们想要保持SSH会话一直处于连接状态，以保持应用程序运行，或者只是避免在返回我们使用的SSH窗口时感到沮丧。
在本教程中，你将获得如何通过防止SSH会话超时，直到你关闭终端窗口。

## 2. 为什么SSH会关闭连接？

我们为了使用SSH登录到服务器上，目的服务器上的守护进程（sshd）一定是保持运行状态。如果SSH客户端一段时间没有发送到目的服务器，服务器会在超过一段时间后关闭连接。

为了防止关闭SSH连接，我们可以在客户端或者服务端进行配置。

<!-- more -->

## 3. 设置配置文件

有几个配置文件可以修改，以保持SSH会话的连接，避免超时。要看是从客户端配置还是服务端配置。

### 3.1 客户端配置

客户端文件位置

```
$HOME/.ssh/config
```

如果我们使用使用 `cat $HOME/.ssh/config` ，可能会得到一个错误信息 “no file found”。

```bash
$ cat $HOME/.ssh/config
cat: /.ssh/config: No such file or directory
```

如果我们看到这个错误消息，意味着我们需要手动创建这个配置文件。如果 `.ssh` 目录不存在，首先创建 `.ssh` 目录，使用命令 `mkdir $HOME/.ssh` 来创建目录，如果提示目录已经存在，我们将会看到一个消息 “File exists”，请忽略即可；如果目录创建成功，我们不会看到任何输出信息。

```bash
$ mkdir $HOME/.ssh
```

然后创建配置文件 `touch $HOME/.ssh/config`。

```bash
$ touch $HOME/.ssh/config
```

一旦我们创建好配置文件，我们还需要使用 `chmod` 修改配置文件的权限，不能让所有人都可以编辑这个配置文件。

```bash
$ chmod 600 $HOME/.ssh/config
```

现在我们可以任何编辑器来编辑这个配置文件了，比如 `nano` 或者 `vim` ，在终端使用 `vim $HOME/.ssh/config` 打开配置文件。

现在让我们来添加一些配置信息到配置文件里面。在 `vim` 按 `i` 进入编辑模式，然后输入下面的内容：

```
Host example
    Hostname example.com
    ServerAliveInterval 240
```

上面的配置信息，仅在SSH会议连接到**example**这个域的时候才会生效。

*ServerAliveInterval* 设置了客户端在发送保持连接信号之前的等待时间。

然后按 `ESC` 键，进入vim的命令模式，连续输入 `:wq` 保存退出vim编辑器。

另外可以把 `example` 换成 `*` 来指定所有的域的配置

```
Host *
    ServerAliveInterval 240
```

我们可以使用上面同样的步骤来编辑和保存配置文件。

### 3.2. 服务端配置文件

在某些情况下，我们可能可以访问服务器上的配置文件。如果是这种情况，我们可以配置何时希望服务器关闭SSH连接。

在服务器上编辑配置文件的过程与客户端配置文件类似，但有一些不同。

首先，服务器端配置文件的文件位置是 `/etc/ssh/sshd_config` ；

现在我们使用 vim 把 *`ClientAliveInterval`* 添加到配置文件，注意，这里是 `"Client"` 而不是 `"Server"` ，跟上面客户端的配置是不一样的。

```bash
ClientAliveInterval 60
```

`ClientAliveInterval` 是以秒为单位指定的超时间隔。如果服务器从客户端接收数据的时间超过了超时间隔，则服务器将向客户端发送请求响应的消息。

### 3.3. 为什么不设置为从不断开连接？

虽然将SSH会话设置为永不断开可能很诱人，但在某些情况下，我们更明智的做法是为SSH设置超时。

如果我们连接的服务器是我们自己维护的服务器，那么设置超时可能没有充分的理由。然而，如果我们在AWS E3这样的平台上托管我们的服务器，如果我们不设置超时，代价可能会很高。许多云托管平台使用服务器时每分钟收费，即使我们不积极使用SSH会话，保持SSH会话持续运行也会增加成本。

要在 `客户端` 上配置超时，我们可以在与上面相同的配置文件中使用ServerAliveCountMax配置项来设置尝试的次数：

```bash
Host *
    ServerAliveInterval 240
    ServerAliveCountMax 2
```

客户端继续每240秒发送一次信号，客户端现在还将侦听来自服务器的信号。如果它两次执行ServerAliveInterval而没有收到信号，它将关闭SSH会话。

同样的，我们可以在服务器上做同样的配置：

```
ClientAliveInterval 60
ClientAliveCountMax 2
```

> 注意，服务器上的配置没有 `Host *` 的配置项
>

这将每60秒从服务器向客户端发送一次信号。如果它经过两次 `ClientAliveInterval` 而没有从客户端收到信号，则服务器将关闭SSH会话。

SSH会话超时，这通常是由于客户端和服务器之间的网络断开。

在大多数情况下，客户端配置需要设置为低于服务器默认超时的值。默认情况下，服务器上禁用连接终止，因此如果不想更改此行为，我们不需要调整服务器配置。

如果没有为ServerAliveCountMax或ClientAliveCountMax提供值，则默认值3将应用于两者。

## 4. 结论

在本文中，我们学习了如何在客户端和服务器端计算机上创建SSH设置的配置文件。然后，我们研究了哪些配置可以防止SSH会话超时。最后，总结一下我们不应该将SSH会话设置为永不断开的一些原因。

有关配置文件的其他选项的更多信息，请查看 [client-side man file](https://man.openbsd.org/ssh_config) 或 [server-side man file](https://man.openbsd.org/sshd_config)。

## 关注我

关注我的公众号，请把你关心的问题发送给我，我会深入研究并为你解答。
关注公众号及时获取编程、软件工程、Linux知识、AI理论与落地技术实践信息。

![QRcode](/img/qrcode_for_gh_d49b3390adad_258.jpg)

如果博客内容对您有帮助，也可以请我喝杯咖啡，你的支持是我不停的创作源泉。

![WeChat_Pay](/img/wechat_pay_w200.jpg)
