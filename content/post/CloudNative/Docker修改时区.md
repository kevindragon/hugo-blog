---
title: "Docker修改时区"
date: 2022-07-08T10:34:39+08:00
tags: [Docker, 时区]
---

修改各Linux发行版本时区为上海

# Debian系统

在系统命令行使用

```shell
echo "Asia/Shanghai" > /etc/timezone
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

在Dockerfile使用

```Dockerfile
RUN echo "Asia/Shanghai" > /etc/timezone && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

# Ubuntu 20.04

在Dockerfile使用

```Dockerfile
ENV DEBIAN_FRONTEND=noninteractive TZ="Asia/Shanghai"
RUN apt-get update && apt-get install -y tzdata
```

这里需要注意两点：
1. 需要设置`DEBIAN_FRONTEND=noninteractive`，否则在安装tzdata的时候会有交互的选择，会阻止后面的命令运行
2. 设置环境变量`TZ`

# Apline

```Dockerfile
RUN apk --update add tzdata && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    apk del tzdata && \
    rm -rf /var/cache/apk/*
```
