---
title: "Python Docker FastAPI快速开发部署RESTful API"
date: "2023-06-06T19:32:18+08:00"
tags: ["Python", "FastAPI", "Docker"]
categories: [编程语言]
---

本文介绍如何快速使用Python, Docker, FastAPI快速开发RESTful API，以及如何编译为Docker镜像，使用Docker容器运行程序。

<!-- more -->

## 预备知识

在开始开发之前，需要先安装Python和Docker。

Python是一种流行的编程语言，因此在许多操作系统上都可以很容易地安装。可以使用[Miniconda](https://docs.conda.io/en/latest/miniconda.html)，也可以使用官方提供的标准安装包进行安装，个人喜欢Miniconda，不会像[Anaconda](https://www.anaconda.com)那么大，同时又提供了非常多必要的依赖模块。

而[Docker](https://www.docker.com/)则是一种容器化平台，可帮助我们轻松地将应用程序封装在容器中，便于部署和管理。目前Windows、Linx、macOS对Docker的支持也非常的完善了。

[FastAPI](https://fastapi.tiangolo.com/zh/) 是一个用于构建 API 的现代、快速（高性能）的 web 框架，使用 Python 3.6+ 并基于标准的 Python 类型提示。

## 初始化项目及运行环境

Python没有像Java那样极其成熟的一站式的Maven项目管理工具，但是Python有其自身的简洁与规范。像[pyproject.toml](https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/)，以及最近用Rust写的[rye](https://github.com/mitsuhiko/rye)，都是很出色的工具。本文使用的例子，完全使用手动管理即可😄

创建一个项目目录my-project，创建文件`requirements.txt`，`main.py`，`Dockerfile`，后面逐一填写里面的内容。

上面我们安装了Miniconda，建议使用conda来创建一个独立的运行环境，这样即不会影响其他项目的运行环境，还可以保持本项目运行环境的整洁。

```Shell
conda create -n my-project python=3.9 pip
```

这个命令创建一个名为`my-project`的运行环境，同时安装3.9版本的Python，和最新的`pip`工具。

然后激活这个运行环境

```Shell
conda activate my-project
```

## 添加依赖模块

我们使用`requirements.txt`来管理程序依赖的模块，`requirements.txt`文件内容如下

```Shell
-i https://mirrors.aliyun.com/pypi/simple/

fastapi
uvicorn[standard]
```

本文只需要用到这两个模块。第一行的`-i`指定pip使用阿里pypi镜像地址，如果在国内这会大大加快依赖安装的速度，可以换成阿里或者其他的镜像源。

在本地开发电脑上可以使用下面的命令安装依赖

```Shell
pip install -r requirements.txt
```

## 编写代码

### Python代码

FastAPI非常容易使用，也非常适合快速开发RESTful API。以下是我们快速入门的示例代码：

```Python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return "Hello World"


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}

```

在这个简单的示例中，我们创建了一个FastAPI应用程序，它包括两个路由。第一个路由处理根路径并返回一个简单的“Hello World”字符串。第二个路由处理传递给它的路径参数，并返回这些参数的值。第二个路由会以json格式响应内容给调用者（FastAPI框架会自动帮我们字典转换为json数据）。

### 本地运行

编写完上面的代码，使用命令`uvicorn main:app --host 0.0.0.0 --port 8080 --reload`启动程序。`--reload`选项开启了文件更改后自动重启应用。`main`是模块的名称，冒号后面的`app`是实例化FastAPI对象的变量名，可以更换成适合应用场景的名称。

FastAPI还有一个贴心的功能是内置了`swagger-ui`的支持，访问 http://localhost:8080/docs 既可看到所有接口列表，如下图

![Python FastAPI Docker](/img/Python/FastAPI%20Swagger-ui.png)

写生产代码的时候，除了单元测试，还可以通过这个界面进行调试与开发，相当的方便。

### Docker配置文件

先来看看整个目录结构

```Plain Text
|-- Dockerfile
|-- main.py
`-- requirements.txt
```

上面写的Python代码本文希望放在一个Docker容器来运行，需要编写一个Dockerfile配置文件，进行Docker镜像的编译。将FastAPI应用程序部署到Docker容器中可以方便地处理依赖关系，并且易于部署到各种环境中。以下是如何编写一个简单的Dockerfile：

```Shell
FROM python:3.9

WORKDIR /app

RUN echo “Asia/Shanghai” > /etc/timezone && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

COPY requirements.txt /app/

RUN pip3 install -r requirements.txt --no-cache-dir

COPY main.py /app/main.py

EXPOSE 8080

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```

1. 指定使用python 3.9基础镜像
2. 指定容器工作目录为`/app`
3. 设置容器的时区
4. 拷贝`requirements.txt`到容器
5. 使用`requirements.txt`安装python依赖，并且不缓存下载的依赖包
6. 拷贝代码到工作目录`/app`
7. 暴露端口到容器外
8. 指定容器启动时的执行命令

## 构建和运行Docker容器

在终端窗口中，导航到Dockerfile所在的目录，并输入以下命令以构建Docker镜像：

```Shell
docker build -t my_fastapi_app .
```

此命令将构建名为my_fastapi_app的Docker镜像。接下来，您可以使用以下命令运行容器：

```Shell
docker run -d --name my_fastapi_container -p 8080:8080 my_fastapi_app
```

此命令将启动Docker容器，并将容器端口8080映射到主机的端口8080。运行后，可以使用`http://localhost:8080`访问FastAPI应用程序。

## 总结

通过以上步骤，我们成功地将FastAPI应用程序部署到了Docker容器中。这种方式使得我们可以轻松地管理应用程序的依赖关系，并且能够在不同的环境中快速部署应用程序。
