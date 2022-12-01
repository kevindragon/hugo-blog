---
title: "Docker删除镜像的另外一种姿势"
date: 2022-12-01T12:44:21+08:00
tags: [Docker]
---

# Docker删除镜像的另外一种姿势

我们来看一下官方 `docker image rm` 的文档

![docker image rm](/img/CloudNative/Docker/docker-image-rm.png)

是的，你没有看错，就是这么简单，没有非常详细的解释。当你碰到下面的情况的时候，很有可能就不知道所措了。

## `<none> <none>`

当你使用使用 `docker images` 查看镜像列表的时候，很有可能你会看到下面这样的一些镜像列表

![docker none none](/img/CloudNative/Docker/docker-none-none.png)

还有可能多个tag对象到同一个镜像，像下面这样

![docker same image sha256](/img/CloudNative/Docker/docker-same-image-sha256.png)

这时候直接通过 `IMAGE ID` 删除镜像，会得到如下的提示内容：

```bash
$ docker image rm 3635ffe4971c
Error response from daemon: conflict: unable to delete 3635ffe4971c (must be forced) - image is referenced in multiple repositories
```

提示不能删除这个镜像，因为有多个 `repositories` 使用了这个镜像

上面第一种情况比较好处理，直接使用命令 `docker image rm IMAGE_ID` 即可删除，可是第二种情况就不行了。

## 使用@符号

这时我们使用另外一种方式，使用 `@` 符号把 `REPOSITORY` 和 `sha256` 连接起来进行删除。

首先得找到 `sha256`

```bash
$ docker images --digests

vision/front                snapshot             sha256:5e6012aa7c3c0143833c5fede8238aea09b3981d6b8f16c3fc638017f5d2adbc   3635ffe4971c   3 months ago        81.5MB
lookout/front               <none>               sha256:5e6012aa7c3c0143833c5fede8238aea09b3981d6b8f16c3fc638017f5d2adbc   3635ffe4971c   3 months ago        81.5MB
```

得到 `sha256` 哈希码之后

```bash
$ docker image rm lookout/front@sha256:5e6012aa7c3c0143833c5fede8238aea09b3981d6b8f16c3fc638017f5d2adbc

Untagged: lookout/front@sha256:5e6012aa7c3c0143833c5fede8238aea09b3981d6b8f16c3fc638017f5d2adbc
```

命令执行完成后，就会把这个tag删除。然后把剩余的一个 `repository` 删除即可。
