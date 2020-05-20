---
title: Solr控制脚本参考
date: 2018-03-19 15:52:38
tags: [Solr]
---

Solr包含了一个命令行工具`bin/solr`可以用来执行非常多的常用操作。可以启动、停止Solr，创建删除collection/core，对ZooKeeper的操作，以及检查solr状态和配置分片。

`bin/solr`的使用格式如下：

```
bin/solr cmd [options]
```

<!-- more -->

# 启动 Start

## 启动与重启

`start`命令用来启动solr。`restart`可以用来重启solr，不管是从停止状态还是在运行状态。

`start`和`restart`有很多的选项，可以指定运行在SolrCloud模式，可以使用自带的样例配置集，同时还可以指定hostname, port, ZooKeeper。

```
bin/solr start [options]
bin/solr start -help
bin/solr restart [options]
bin/solr restart -help
```

当使用`restart`命令时，命令参数必须与`start`的命令参数一致。重启的过程是先执行`stop`命令停止solr，然后再启动solr，所以`restart`的参数必须要跟`start`的参数一致。

## 启动参数

`bin/solr`命令行脚本提供了非常多的选项来定制solr服务，例如改变监听的端口。其实默认的参数已经能够满足很多情况的需求了。

### `-a "<string>"`

指定JVM参数，例如`-X`。如果是以`-D`传递的JVM参数，-a选项可以忽略。

**Example**：
```
bin/solr start -a "-Xdebug -Xrunjdwp:transport=dt_socket, server=y,suspend=n,address=1044"
```

### `-cloud`

在SolrCloud模式启动，同时启动内置的ZooKeeper。这个选项可以简写为：`-c`。如果已经有一个ZooKeeper实例在运行，可以使用`-z`选项参数替换内置的ZooKeeper。查看更多[SolrCloud](http://lucene.apache.org/solr/guide/7_2/solr-control-script-reference.html#solrcloud-mode)细节。

**Example**：`bin/solr start -c` 

### `-d <dir>`

定义server目录，默认为`server`。server目录的作用是提供web服务，rest api。这个选项一般不需要覆盖。当在同一台机器启动多个solr实例时，更常用的办法是使用`-s`选项来指定不同的solr.home，以存放不同的配置和数据。

**Example**：`bin/solr start -d newServerDir`

### `-e <name>`

以样例配置启动solr。solr提供的这些样例配置以帮助你快速开始熟悉solr的使用，或者某些特性。

这些样例包括：
* cloud
* techproducts
* dih
* schemaless

点击[Running with Example Configurations](http://lucene.apache.org/solr/guide/7_2/solr-control-script-reference.html#running-with-example-configurations)查看详情。

**Example**：`bin/solr start -e schemaless`

### `-f`

在前台运行。使用`-e`选项的时候此选项不能使用。

**Example**：`bin/solr start -f`

### `-h <hostname>`

指定hostname，如果此选项没有指定，默认使用localhost。

**Example**：`bin/solr start -h kevinjiang.info`

### `-m <memory>`

指定启动时JVM最小（-Xms）、最大（-Xmx）堆内存。

**Example**：`bin/solr start -m 1g`

### `-noprompt`

些选项会隐藏所有的提示信息，所有的配置都使用默认值。例如，当使用cloud样例时，会出现一个交互session，以步骤的方式来选择或输入一些配置来启动SolrCloud集群，可以使用-noprompt接受默认值。

**Example**：`bin/solr start -e cloud -noprompt`

### `-p <port>`

指定端口，默认是8983端口，如果没有使用-z选项指定ZooKeeper，ZooKeeper端口默认+1000，为：9983

**Example**：`bin/solr start -p 8655`

### `-s <dir>`

指定`solr.solr.home`参数，Solr的数据与配置将会写到这个目录下面。通过这个选项可以启动多个实例，同时可以使用`-d`选项指定相同的server目录。

如果指定此选项，相应的目录需要包含一个名为`solr.xml`的文件，除非`solr.xml`已经存在于ZooKeeper。默认值为：`server/solr`

如果使用`-e`选项，此选项将被忽略，`solr.solr.home`会根据不同的样例而不同。

**Example**：`bin/solr start -s newHome`

### `-v`

指定输出更多的日志信息，日志级别会从`INFO`修改为`DEBUG`，如果修改`log4j.properties`，也可以达到同样的效果。

**Example**：`bin/solr start -f -v`

### `-q`

指定输出更少的日志信息，日志级别会从`INFO`修改为`WARN`，如果修改`log4j.properties`，也可以达到同样的效果。这在生产环境是比较实用的，可以指定只记录warning和error级别的日志。

**Example**：`bin/solr start -f -q`

### `-V`

启动此选项将会在命令行输出更为详细的信息

**Example**：`bin/solr start -V`

### `-z <zkHost>`

指定ZooKeeper连接字符串，些选项仅与`-c`选项同时使用来启动SolrCloud模式。如果些选项没有指定，在SolrCloud模式下，Solr会启动内置的ZooKeeper实例。

**Example**：`bin/solr start -c -z server1:2181,server2:2181`

### `-force`

Solr不能使用root账户启动，退出的同时会输出错误信息：以root账户运行Solr会导致安全问题。指定此参数可以用root账户启动。

**Example**：`sudo bin/solr start -force`

下面两个命令是等价的：

```
bin/solr start
bin/solr start -h localhost -p 8983 -d server -s solr -m 512m
```

## 设置JVM参数
跟启动Java程序一样，可以使用`-D`选项指定属性值。例如，指定自动提交频率为3秒：

```
bin/solr start -Dsolr.autoSoftCommit.maxTime=3000
```

# 停止 Stop

`stop`命令发送一个`STOP`请求到运行状态节点，这个姿势可以优雅的关闭Solr。此命令会先等待180秒的时间进行优雅的停止，如果还没有停止则强制杀掉进程（kill -9）

```
bin/solr stop [options]
bin/solr stop -help
```

## 停止参数

### `-p <port>`

停止运行在指定端口的Solr。如果运行多个Solr实例，或者是SolrCloud模式，可以分开指定不同的端口，或者`-all`来停止。

**Example**：`bin/solr stop -p 8983`

### `-all`

停止所有运行中的Solr实例

**Example**：`bin/solr stop -all`

### `-k <key>`

指定需要停止的Solr的key，默认是solrrocks，可以通过`-DSTOP.KEY=solrrocks`来指定其他的名字。这个选项不常用

**Example**: `bin/solr stop -k solrrocks`

# 系统信息

## 版本

`version`命令可以查看Solr的版本信息：

```
bin/solr version
```

## 状态

`status`用来查看本地运行中的Solr节点的状态信息，以JSON格式输出状态信息。`status`会读取`SOLR_PID_DIR`环境变量指定的目录来查找pid文件，默认是`bin`目录。

```
bin/solr status
```

输出信息包含每个节点的状态，例如：

```
Found 2 Solr nodes:

Solr process 39920 running on port 7574
{
  "solr_home":"/Applications/Solr/example/cloud/node2/solr/",
  "version":"X.Y.0",
  "startTime":"2015-02-10T17:19:54.739Z",
  "uptime":"1 days, 23 hours, 55 minutes, 48 seconds",
  "memory":"77.2 MB (%15.7) of 490.7 MB",
  "cloud":{
    "ZooKeeper":"localhost:9865",
    "liveNodes":"2",
    "collections":"2"}}

Solr process 39827 running on port 8865
{
  "solr_home":"/Applications/Solr/example/cloud/node1/solr/",
  "version":"X.Y.0",
  "startTime":"2015-02-10T17:19:49.057Z",
  "uptime":"1 days, 23 hours, 55 minutes, 54 seconds",
  "memory":"94.2 MB (%19.2) of 490.7 MB",
  "cloud":{
    "ZooKeeper":"localhost:9865",
    "liveNodes":"2",
    "collections":"2"}}
```

## Assert 检查安装正确

`assert`命令可以检查一般的安装问题，检查目录权限，目录是否存在，指定的URL是否可用。此命令会输出错误信息，或者改变exit code（Linux可以使用`echo $?`查看）。下面是一个例子：

```
bin/solr assert --exists /opt/bin/solr
```

输出结果如下：

```
ERROR: Directory /opt/bin/solr does not exist.
```

使用`bin/solr assert -help`查看更多`assert`命令帮助

## 健康检查

`healthcheck`命令会生成SolrCloud模式下指定collection的JSON格式的健康报告。报告提供关于所有分片的复制的状态信息，包括提交的文档数，当着状态。

```
bin/solr healthcheck [options]
bin/solr healthcheck -help
```

### 选项参数

#### `-c <collection>`

指定生成报告的collection的名字

**Example**：`bin/solr healthcheck -c gettingstarted`

#### `-z <zkhost>`

指定ZooKeeper连接字符串，默认是`localhost:9983`。如果Solr不是运行在8983端口，就需要指定ZooKeeper连接字符串，一般是solr的端口+1000.

**Example**：`bin/solr healthcheck -z localhost:2181`

下面是一个健康检查命令和结果执行：

```
$ bin/solr healthcheck -c gettingstarted -z localhost:9865
```

```json
{
  "collection":"gettingstarted",
  "status":"healthy",
  "numDocs":0,
  "numShards":2,
  "shards":[
    {
      "shard":"shard1",
      "status":"healthy",
      "replicas":[
        {
          "name":"core_node1",
          "url":"http://10.0.1.10:8865/solr/gettingstarted_shard1_replica2/",
          "numDocs":0,
          "status":"active",
          "uptime":"2 days, 1 hours, 18 minutes, 48 seconds",
          "memory":"25.6 MB (%5.2) of 490.7 MB",
          "leader":true},
        {
          "name":"core_node4",
          "url":"http://10.0.1.10:7574/solr/gettingstarted_shard1_replica1/",
          "numDocs":0,
          "status":"active",
          "uptime":"2 days, 1 hours, 18 minutes, 42 seconds",
          "memory":"95.3 MB (%19.4) of 490.7 MB"}]},
    {
      "shard":"shard2",
      "status":"healthy",
      "replicas":[
        {
          "name":"core_node2",
          "url":"http://10.0.1.10:8865/solr/gettingstarted_shard2_replica2/",
          "numDocs":0,
          "status":"active",
          "uptime":"2 days, 1 hours, 18 minutes, 48 seconds",
          "memory":"25.8 MB (%5.3) of 490.7 MB"},
        {
          "name":"core_node3",
          "url":"http://10.0.1.10:7574/solr/gettingstarted_shard2_replica1/",
          "numDocs":0,
          "status":"active",
          "uptime":"2 days, 1 hours, 18 minutes, 42 seconds",
          "memory":"95.4 MB (%19.4) of 490.7 MB",
          "leader":true}]}]}
```

# 数据集合（collections/cores）

> Collection, Core不知道怎么用中文描述，称作库也不太合适。暂且称为数据集合吧。

`bin/solr`命令行工具可以帮助创建/删除数据集合（collections/cores），collections是在SolrCloud模式下的概念；cores是非SolrCloud模式下的概念。

> 以前的版本没有cloud模式的时候就叫core。

## 创建数据集合

创建数据集合使用`create`命令，要求Solr必须是运行状态，具体是创建的collection还是core要根据Solr运行的模式来定。

```
bin/solr create [options]
bin/solr create -help
```

### 创建数据集合的选项参数

#### `-c <name>`

指定collection/core的名字

**Example**：`bin/solr create -c mycollection`

#### `-d <confdir>`

指定使用的配置的目录名字，默认是`_default`。详情参考[Configuration Directories and SolrCloud](http://lucene.apache.org/solr/guide/7_2/solr-control-script-reference.html#configuration-directories-and-solrcloud)

**Example**：`bin/solr create -d _default`

#### `-n <configName>`

指定配置的名字，默认创建同名的collection/core。

**Example**：`bin/solr create -n basic`

#### `-p <port>`

指定本地Solr运行的端口来发送创建命令，默认会探测运行中的solr实例的端口。此选择对于一个机器运行多个实例的情况非常有用，可以通过指定端口来指定在不同的实例当中创建collection/core。

**Example**：`bin/solr create -p 8983`

#### `-s <shards>`或`-shards`

指定创建的时候数据集合分成多少个分片（所有分片里面的数据合起来才是完整的一个数据集），默认是1，仅当运行在SolrCloud模式的才用到。

**Example**：`bin/solr create -s 2`

#### `-rf <replicas>`或`-replicationFactor`

指定每个文档的复制数据，默认是1（没有复制）。

**Example**：`bin/solr create -rf 2`

#### `-force`

如果是以root账号运行的此命令，需要此选项来强制运行，否则是运行不了的。

**Example**：`bin/solr create -c foo -force`

# 删除数据集合

:TODO
