---
title: "Solr教程-安装运行"
date: 2019-03-05 20:13:28
tags: [Solr,Lucene,搜索引擎]
---

Solr是一个分布式高性能全文搜索引擎，是基于Lucene开发的非常流行的企业搜索引擎平台。Solr提供了非常丰富的搜索功能，可以用来构建非常复杂的搜索业务。

这个系列教程旨在介绍Solr的常用功能。

<!-- more -->

# 准备

在开始之前你需要准备一些必要的硬件及软件设施：

1. [JDK 1.8](https://jdk.java.net/8/) (或者更高版本)
2. [Solr 7.5.x](http://www.apache.org/dyn/closer.lua/lucene/solr/7.5.0)
3. [Postman](https://www.getpostman.com/) （可选）
4. [CentOS 7.x](https://www.centos.org/) (本系列以此OS为准)
5. [VirtualBox](https://www.virtualbox.org/) (可选)

CentOS可以选择安装，Solr是支持多操作系统的，但是在Linux下体验更好，大部分生产环境也都是Linux系统，建议使用CentOS来学习以及使用Solr。

VirtualBox可以选择安装，如果你是Windows操作系统，建议使用VirtualBox安装CentOS来学习使用Solr。

后面的操作都是基于CentOS 7.x版本操作系统来运行。

# 安装Solr

安装Solr需要Java 1.8或者更高版本，可以使用`java -version`命令检查你系统上安装的Java版本。由于Java语言有好几个实现，有些实现并不能非常好的支持Solr，根据[Lucene JavaBugs](https://wiki.apache.org/lucene-java/JavaBugs)来检查你的Java是否对Solr有很好的支持。Solr支持的操作系统包括Linux, MacOS, Windows。

到[https://lucene.apache.org/solr/mirrors-solr-latest-redir.html](https://lucene.apache.org/solr/mirrors-solr-latest-redir.html)下载对应操作系统的7.5.0版本的Solr安装包。

* `solr-7.5.0.tgz` 是针对Linux/Unix/MacOS系统的安装包
* `solr-7.5.0.zip` 是针对Windows系统的安装包
* `solr-7.5.0-src.tgz` 是源码包

```shell
tar zxf solr-7.5.0.tgz
cd solr-7.5.0
bin/solr start
```

然后使用浏览器打开[http://localhost:8983](http://localhost:8983)，你就可以看到Solr Admin UI的界面了，这是Solr自带的管理界面，使用非常方便，在里面可以查看状态，管理core，进行查询。

以`bin/solr start`运行的solr是以standalone模式（非分布式）运行的，如果是学习可以使用此模式快速的尝试。但是建议使用后面将要介绍的SolrCloud模式来运行，SolrCloud模式可以很好的进行扩展，应对海量数据的实时搜索，动态剔除/增加机器节点，动态转移数据分片(shard)，所有这些操作都可以不停机进行操作。生产环境也建议使用SolrCloud模式运行。

这是以SolrCloud模式启动solr的命令，只需要添加一个`-c`参数即可

```shell
bin/solr start -c
```

详细的命令行参数请参考[Solr控制脚本参考](https://kevinjiang.info/2018/03/19/Solr%E6%8E%A7%E5%88%B6%E8%84%9A%E6%9C%AC%E5%8F%82%E8%80%83/)

SolrCloud启动之后会占用8983端口，默认会启动一个内置的zookeeper实例，占用9983端口（在solr的端口上加1000）。

# 多节点的SolrCloud安装-生产环境部署

一般生产环境部署会把zookeeper单独部署，其他的solr节点单独部署。

## 外部Zookeeper

下面假设是生产环境部署zookeeper，建议至少需要三个zookeeper节点。 _开发环境以及测试使用一个zookeeper也没有关系。_

假设已经有三个机器来运行zookeeper来组建一个小的zookeeper集群：

1. 192.168.1.2
2. 192.168.1.3
3. 192.168.1.4

步骤为：下载[zookeeper-3.4.13](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz) -> 解压 -> 配置conf/zoo.cfg -> 启动

```shell
curl -o zookeeper-3.4.13.tar.gz 'https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz'
tar -xzvf zookeeper-3.4.13.tar.gz
cd zookeeper-3.4.13
cp conf/zoo_sample.cfg conf/zoo.cfg
sed -i 's/dataDir=.*/dataDir=.\/data/g' conf/zoo.cfg
mkdir data
echo -e "\nserver.1=192.168.1.2:2888:3888\nserver.2=192.168.1.3:2888:3888\nserver.3=192.168.1.4:2888:3888" >> conf/zoo.cfg
echo 1 > data/myid
```

解释一下上面的一些命令：

前四行就不过多解释，下载zookeeper-3.4.13安装包，解压，拷贝conf/zoo_example.cfg为conf/zoo.cfg配置文件。conf/zoo.cfg为必须的配置文件，否则zookeeper是无法启动的。

第五行使用`sed`命令把conf/zoo.cfg配置文件里面的dataDir设置为安装目录下面的data目录。接着第六行创建dataDir指定的目录，否则也无法正确启动zookeeper。第七行写入三台机器互相通信的配置，2888是连接leader端口，3888是选举端口，两个端口都可以修改。

**最后一行也是最重要的一行，每台机器在data/myid文件里面写入一个不重复的值来标识每个节点。所以在其他两台机器就不能再写入1了，分别写入2和3即可。**

上面拷贝conf/zoo_example.cfg为conf/zoo.cfg的时候，已经默认设置了2181端口为zookeeper客户端连接的端口，下面在solr启动的时候使用。

最后运行`bin/zkServer.sh start`启动三台机器上的zookeeper即可。

## 启动多个solr节点

假设现在也有三个机器安装了solr（下载下来直接解压即可），分别为`192.168.1.10`、`192.168.1.11`、`192.168.1.12`，解压的目录都为`solr-7.5.0`。命令行切换到solr-7.5.0目录运行如下命令：

```shell
bin/solr start -c -z 192.168.1.2:2181,192.168.1.3:2181,192.168.1.4:2181
```

三台机器运行同样的命令后，整个SolrCloud集群就已经启动好了，可以访问[http://192.168.1.10:8983](http://192.168.1.10:8983)查看solr admin ui。

## 负载均衡

这个话题比较有意思，很多人问我SolrCloud已经使用了Zookeeper来选举leader了，SolrCloud里面的leader shard也会把请求分发到其他的shard上，为什么还要做负载均衡呢。如果你深入了解过solr的search component你就会明白，接收请求的那个机器做的计算是要比其他机器多的，如果你在代码当中只是访问其中一台机器，那么这台机器的负载会高于其他的机器。所以你需要在你的应用程序和SolrCloud之间搭建一个负载均衡，把请求均匀的分布到各个机器上去。

常用的有nginx, haproxy这两个做负载均衡的软件了。最后要保证高可用，还需要做一个负载均衡层面的热备的配置，这样你的SolrCloud可以说已经非常坚固了。

<img src="/img/wechat.jpg" width="100" />
