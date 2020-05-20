---
title: Windows下Rust编译碰到lntddl not found
date: 2019-09-04 11:50:04
tags: [Rust]
---

在学习使用actix-web的时候，使用systemfd、 listenfd来自动编译加载，但是碰到了`gcc ... -lntdll not found`的提示。使用的toolchain是`stable-x86_64-pc-windows-gnu`，而且电脑上也安装了msys2，经过搜索找到了解决办法。

需要配置 `~/.cargo/config` 写入正面的内容：

```
[target.x86_64-pc-windows-gnu]
linker = "C:\\msys64\\mingw64\\bin\\gcc.exe"
ar = "C:\\msys64\\mingw64\\bin\\ar.exe"
```

然后`cargo build`一切正常