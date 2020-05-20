---
title: Haskell在Windows上的起步
date: 2019-08-17 21:48:21
tags: [Haskell]
---

学习Haskel痛苦又快乐的经历

# 使用Haskell Platform安装的Haskell

安装的时候最好指定安装在`c:\Hs`目录，因为路径短。然后确保下面三个配置在cabal的配置里面是正确的路径。

```
extra-prog-path: C:\Hs\msys\usr\bin
extra-lib-dirs: C:\Hs\mingw\lib
extra-include-dirs: C:\Hs\mingw\include
```

这种方式安装的Haskell可以顺序安装stack，但是stack不会使用全局的ghc，如果有库使用到了同一个版本的ghc，stack还是傻傻的再去下载一个ghc到自己的目录里面（不应该这么傻，有可能什么都想自己控制吧）。这个情况通过下面的命令设置使用全局的ghc

```shell
stack config set system-ghc --global true
```

# 安装Haskell IDE engine

这个玩意在windows安装是不成功的，可能是路径过长的问题（Windows最长是260个字符），可以把stack的根路径设置到C盘的根目录，设置环境变量`STACK_ROOT`为`C:\stack`。

设置完成后，上面一个步骤的system-ghc设置就无效了，因为stack会读取`c:\stack\config.yaml`以及`c:\stack\global-project\stack.yaml`两个文件来决定stack更新的方式。这是要注意的。