---
title: Python项目结构实践
date: 2019-06-26 19:00:10
tags: [Python]
---

相比于Java和Rust，Python的项目结构没有一个明确的规范，最近在做Python机器学习项目的时候碰到很多的情况是执行文件非常多，有些同学并没有把一些公用的方法和功能进行抽取，而是每个文件都含有读取/写入文件的重复代码。在对setuptools进行一些学习研究，其实Python项目的结构也可以做得非常好，只是大学在赶功能的时候并没有进行一定的规划。下面做一些总结，有了这个结构，可以把随手写的一些项目分享给其他的项目使用。

<!-- more -->

# Setuptools

Setuptools是用到构建和分发Python包（Packages或者叫项目）的一个工具，可以：

1. 自动查找/下载/安装/升级项目的依赖包
2. 项目内部多个模块之间的import更加方便
3. 可以把项目用到的数据文件打包到一个压缩文件内
4. 自动查找项目内的包/模块
5. 自动生成可执行程序
6. 以及其他一些好用的功能...

最主要的功能还是管理依赖，进行源代码的分发（可以把代码分发到Pypi上面，或者分发给其他项目使用）

首先需要确保你已经安装了这个工具:

```shell
pip install -U setuptools
```

# 基本用法

在你的项目根目录下面创建一个`setup.py`文件，写入下面的内容：

```python
from setuptools import setup, find_packages
setup(
    name="HelloWorld",
    version="0.1",
    packages=find_packages(),
)
```

这是一个最基本的配置脚本，定义了包名`name`，版本号`version`，以及这个项目包含的包列表`packages`，包列表使用setuptools提供的`find_packages`自动在目录里面进行查找。

如果你是在开发阶段可以使用下面的命令来安装你的包，以开发模式安装的包不会安装在site-packages目录，而是在site-packages目录下面会建立一个链接（快捷方式）指向到你的开发目录，这样你所做的改动会马上体现出来。

```shell
pip install -e .
```

# 两种目录结构

假设我们的包名为`hello`，下面以两种结构分别介绍如何来进行配置

## 结构一

```
hello               (1)
|- hello            (2)
|  |- __init__.py
|  `- world.py
|- main.py
`- setup.py
```

这种目录结构是pythonic的风格，以包名为源代码的根目录名称，然后把所有的模块都放在这个目录下面（即hello(2)目录）。

这种结果的`setup.py`如下：

```python
from setuptools import setup, find_packages
setup(
    name='hello',
    version='0.1.0',
    description=''
    packages=find_packages(),
    install_requires=[]
)
```

## 结构二

```
hello
|- src
|  `- hello
|     |- __init__.py
|     `- world.py
|- main.py
`- setup.py
```

这种目录结构把所有的源码文件都放在src子目录里面，这样的习惯可能是Java程序员比较喜欢的。

这种结果的`setup.py`如下：

```python
from setuptools import setup, find_packages
setup(
    name='hello',
    version='0.1.0',
    description=''
    packages=['hello'],
    package_dir={'': 'src'},
    install_requires=[]
)
```

这两种结构的区别是写`setup.py`的时候`packages`参数的不同。

萝卜白菜各有所爱吧，但是结构一是pythonic风格，也是推荐的一种风格。