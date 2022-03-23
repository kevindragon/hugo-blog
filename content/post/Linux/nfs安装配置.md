---
title: nfs安装配置
date: 2017-06-30 16:14:20
tags: [Linux]
---

记录一下nfs的安装配置以备忘

## 环境

### server
CentOS Linux release 7.3.1611

### client
CentOS Linux release 7.2.1511

## Server

### 安装

```bash
yum install -y nfs-utils rpcbind
```

修改配置文件`/etc/expots`

```
/opt/tmp *(rw,sync)
```

启动nfs server端

```bash
sudo systemctl start rpcbind
sudo systemctl start nfs
```

## Client

```bash
mkdir tmp
sudo mount -t nfs 10.123.4.215:/opt/tmp tmp
```
