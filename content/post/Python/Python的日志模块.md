---
title: Python的日志模块
date: 2018-03-27 17:12:32
tags: [Python]
---

Python3的日志模块（`logging`）简单而又强大，通过简单的配置可以实现日志写入不同的地方，对于开发和线上环境的debug有很大的帮助。

# 超简单例子

## 一个简单的例子

```python
import logging
logging.warning('Watch out!')  # 打印输出到控制台
logging.info('I told you so')  # 不会打印输出
```

如果保存为python文件（`example.py`），然后在控制台运行（`python example.py`）将看到下面的输出：

```
WARNING:root:Watch out!
```

默认情况`logging`模块只会输出`WARNING`级别以及往上的`ERROR`, `CRITICAL`级别的消息。

<!-- more -->

## 写入到文件

```python
import logging
logging.basicConfig(filename='example.log',level=logging.DEBUG)
logging.debug('This message should go to the log file')
logging.info('So should this')
logging.warning('And this, too')
```

保存为`example2.py`，运行`python example2.py`后，打开`example.log`文件，内容如下：

```
DEBUG:root:This message should go to the log file
INFO:root:So should this
WARNING:root:And this, too
```

这个例子设置了日志级别为`logging.DEBUG`，日志文件也会记录相应级别的日志消息。

需要注意`logging.basicConfig`需要放在最前面调用，之后才可以调用`logging.debug`, `logging.info`。`logging.basicConfig`是一次性调用的函数，调用过一次，其他的调用将不再生效。

多次运行上面的例子，日志文件只会记录一次日志消息，如果要记录每次运行的消息，需要指定filemode参数：

```
logging.basicConfig(filename='example.log', filemode='w', level=logging.DEBUG)
```

## 从多个模块记录日志

假设你的项目由多个模块缓存，应该在程序入口处（或者在其他的调用`info`, `debug`函数之前）对日志模块进行配置。

```python
# myapp.py
import logging
import mylib

def main():
    logging.basicConfig(filename='myapp.log', level=logging.INFO)
    logging.info('Started')
    mylib.do_something()
    logging.info('Finished')

if __name__ == '__main__':
    main()
```

```python
# mylib.py
import logging

def do_something():
    logging.info('Doing something')
```

运行`myapp.py`（`python myapp.py`）文件将会在`myapp.log`日志文件看到如下的输出信息：

```
INFO:root:Started
INFO:root:Doing something
INFO:root:Finished
```

# 日志模块简析

## 概念

### Logger

相当于是不同的记录日志的方式，可以给一个Logger命名。Logger对象不需要初始化，而是通过`logging.getLogger(name)`方法获取，多个`getLogger`通过相同的`name`参数获取到的是相同的Logger对象。

### Levels

日志的不同级别。一般的，日志都会分不同的级别，例如开发环境一般设置为DEBUG级别，以记录更多的信息利于开发过程的跟踪调试；而线上环境设置了INFO或者更高的级别。下表列出来了所有的日志级别：

|  Level   | Numeric value |
|----------|---------------|
| CRITICAL | 50            |
| ERROR    | 40            |
| WARNING  | 30            |
| INFO     | 20            |
| DEBUG    | 10            |
| NOTSET   | 0             |

数值越大级别越高，消息代表的问题越严重

### Handler

处理日志消息。设置日志级别、格式、过滤、写入等的操作。

### Formatter

日志格式。常用格式有下面一些：

| format        | description                                                    |
|---------------|----------------------------------------------------------------|
| %(asctime)s   | 默认格式为：`2003-07-08 16:49:45,896`，可以通过datefmt进行配置    |
| %(levelname)s | 日志级别：INFO, DEBUG等                                         |
| %(name)s      | Logger对象的name                                                |
| %(message)s   | 消息内容                                                        |

更为详细的格式参考[LogRecord attributes](https://docs.python.org/3/library/logging.html#logrecord-attributes)

### Filter

过滤器。常用的方法是在`getLogger(name)`函数调用的时候使用，name参数就是一个过滤器。假如name为`A.B`，那么以`A.B`开头的都会被记录（`A.B.C`, `A.B.C.D`, `A.B.D`），而`A.BB`是不会被`A.B`记录的。

### 使用步骤

使用的步骤为先进行配置，然后调用`logging`模块提供的方法：`debug()`，`info()`，`warning()`，`error()`，`critical()`。配置的方式大概有三种：

1. 代码里面配置
2. 调用logging.config.fileConfig配置
3. 调用logging.config.dictConfig配置

## 日志消息流程

```
            |-- root handler         |                 | -- console
            |                        |                 | -- file
message --> |-- console/file handler | -- write to --> | -- http
            |                        |                 | -- smtp
            |-- other handles        |                 | -- syslog, etc.
```

# 更长的例子

下面以一个更长的例子说明上面提到的一些概念和使用方法，这个是通过代码的方式进行配置

```python
import logging

formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# ---------- (1) ----------
# create logger
logger = logging.getLogger('simple.example')
logger.setLevel(logging.DEBUG)
# create console handler and set level to debug
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(formatter)
# add ch to logger
logger.addHandler(ch)
# 'application' code
logger.debug('debug message')
logger.info('info message')
# ---------- (1) ----------

# ---------- (2) ----------
logger2 = logging.getLogger('simple.example.2')
logger2.setLevel(logging.INFO)
ch2 = logging.StreamHandler()
ch2.setLevel(logging.INFO)
ch2.setFormatter(formatter)
logger2.addHandler(ch2)
logger2.debug('debug message from example2')
logger2.info('info message from example2')
# ---------- (2) ----------

logging.error('logging info message')
```

输出：

```
2018-03-27 18:23:56,625 - simple.example - DEBUG - debug message
2018-03-27 18:23:56,626 - simple.example - INFO - info message
2018-03-27 18:23:56,626 - simple.example.2 - INFO - info message from example2
2018-03-27 18:23:56,626 - simple.example.2 - INFO - info message from example2
ERROR:root:logging info message
```

首先创建了一个formatter以格式化输出的消息，这个formatter可能给不同的Logger共用。

在(1)的部分创建了一个Logger对象logger，同时设置记录日志的级别为DEBUG。然后创建了一个Handler，默认会输出日志消息到控制台，Handler也可以设置不同的日志级别。以两个日志级别最高的为准。通过调用`logger.addHandler`为Logger对象添加Handler，一个Logger对象可以添加多个Handler以输出同一条消息到不同的地方。

在(2)的部分创建了另外一个Logger对象logger2，同时创建了另外一个Handler ch2，使用跟ch相同的formatter，日志级别为INFO。

最后调用logging模块的`error`方法，logging模块本身有一个默认的Logger对象，名字是`root`，Handler和Formatter也是默认的。

在输出的结果里面可以看到有两个`simple.example.2 - INFO`的输出，这是因为(1)里面的logger对象的名字是`simple.example`，是`simple.example.2`的前缀，所以`simple.example.2`的消息也会被`simple.example`捕获到。

# 通过配置文件使用日志模块

配置主要介绍`dictConfig`的方式，可以是json，或者yaml文件，通过相应的模块解析成字典传递给`dictConfig`函数进行配置。

`logging.yml`

```yml
version: 1
root:
  level: DEBUG
  handlers: [console, file]
loggers:
  example:
    level: INFO
    handlers: [console]
handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: standardFormatter
    stream: ext://sys.stdout
  file:
    class: logging.FileHandler
    level: INFO
    formatter: standardFormatter
    filename: example.log
formatters:
  standardFormatter:
    format: '%(asctime)s %(name)8s %(levelname)-6s %(message)s'
```

> 首先version是必须要有的，不然会报错
> 然后配置默认的Logger对象root，记录日志级别为DEBUG，通过两个Handler来处理消息，分别写到控制台和文件
> loggers配置了一个Logger对象example，也是使用的console这个Handler
> Handler的配置包含两个对象的配置：控制台和文件。控制台使用StreamHandler对象，文件使用FileHandler对象。分别记录DEBUG和INFO级别的消息
> 两个Handler都使用同一个formatter来格式化消息
> 最后配置Formatter对象standardFormatter。这里有个8s和-6s的格式。8s是靠右对齐，不足8个字符左边以空格补齐；-6s相反，靠左对齐，不足6个字符右边以空格补齐

`example.py`

```python
import logging
import yaml
import logging.config

logging_config = yaml.load(open('logging.yml', 'r'))

logging.config.dictConfig(logging_config)

logging.debug('debug message')
logging.info('info message')

logger = logging.getLogger('example')
logger.info('logger info message')
logger.debug('logger debug message')
```

> 使用dictConfig配置日志模块需要一个字典，所以通过yaml模块加载我们的配置文件
> 然后直接调用logging模块的debug和info方法即可，非常方便
> 最后获取了一个名为example的Logger对象，在上面的yaml配置文件当中只记录INFO级别的消息，所以debug的消息是不会输出的

输出会有两个地方，一个是控制台，一个是`example.log`文件

`控制台输出`

```
2018-03-28 08:45:51,134     root DEBUG  debug message
2018-03-28 08:45:51,134     root INFO   info message
2018-03-28 08:45:51,134  example INFO   logger info message
2018-03-28 08:45:51,134  example INFO   logger info message
```

`example.log`

```
2018-03-28 08:49:25,615     root INFO   info message
2018-03-28 08:49:25,615  example INFO   logger info message
```

通过配置文件可以使得代码量更少，也可以更加灵活的配置各种日志。
