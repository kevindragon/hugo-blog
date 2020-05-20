---
title: 在AWS上部署Elasticsearch集群
date: 2019-01-08 18:27:18
tags: [Elasticsearch, AWS]
---

下面介绍一下在AWS上面部署Elasticsearch集群，结合AWS提供的负载均衡器(Elastic Load Balancing)和Auto Scaling Group功能实现，Elasticsearch的集群搭建。演示的版本为Elasticsearch 6.5.4。

<!-- more -->

## 启动EC2实例

首先启动一个EC2的实例，在这个实例里面安装Elasticsearch。
![](/img/aws/aws_es_cluster/1.png)
启动新实例，选择`Amazon Linux 2 AMI (HVM), SSD Volume Type`的AMI（Amazon Machine Image），然后点`下一步:配置实例详细信息`。配置详细信息界面，只需要选择一下子网即可，然后保留默认配置即可，然后点击`下一步:添加存储`。保持默认配置，继续点`下一步:添加标签`。添加标签不需要做配置，继续点`下一步:配置安全组`。在配置安全组界面，选择一个现有的安全组，或者创建一个都可以，但是有一点需要注意的：保证你选择（创建）的安全组可以让你自己或者所有IP访问。这里选择现有的安全组，后面还会介绍对安全组的配置。在`安全组 ID`表格里面选择`default`那一行，然后点击`审核和启动`。信息没有问题最后点`启动`。

在弹出的`选择现有密钥对或创建新密钥对`对话框中选择你已经存在的或者创建一个新的密钥对，这个密钥对是通过ssh来访问aws的ec2实例的。如果这一步不清楚就选择创建新密钥对。

## 安装Elasticsearch

假设上一次最后创建的密钥对为`aws.pem`，实例公有DNS(域名)为`ec2-100-26-149-233.compute-1.amazonaws.com`，使用ssh登录ec2实例：

```shell
ssh -i aws.pem ec2-user@ec2-100-26-149-233.compute-1.amazonaws.com
```

实例的公有DNS（IPv4）到ec2的管理界面，点击前面启动的实例，在下面的信息面板就可以看到。

Amazon本身提供的AMI是基于CentOS系列的Linux系统，可以直接使用yum来安装Elasticsearch。根据Elasticsearch官方提供的[文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)进行安装。

修改文件`/etc/yum.repos.d/amzn2-core.repo`，在文件后面添加下面的内容：

```conf
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

然后执行下面的命令：

```shell
sudo yum install -y java-1.8.0
sudo yum install -y elasticsearch
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
```

后面两条systemctl命令设置Linux开机启动Elasticsearch。这样就安装好了最新版本的Elasticsearch，下面就需要对其进行配置，以及制作自己的AMI了。

## 配置Elasticsearch以及制作AMI

### AWS的IAM

IAM是个什么鬼呢。IAM就是AWS在一个账户下面，提供一种机制在SDK或者aws命令行来操作AWS服务的认证方式。一个AWS账户可以创建多个IAM访问权限，Elasticsearch需要使用access_key和secret_key。

在服务里面找到IAM，进入到控制面板里面，点击左侧的用户，然后按`添加用户`，用户名输入：`elasticsearch`，在访问类型一栏，把`编程访问`勾上，然后点击`下一步:权限`。在设置权限界面选择`直接附加现有策略`，在下面的列表当中选择`AdministratorAccess `，然后点击`下一步:标签`，保持默认，继续下一步。最后点击`创建用户`。

创建成功后需要把`访问密钥 ID`和`私有访问密钥`保存下面，在下面的配置当中需要使用。

### 配置Elasticsearch

首先需要对Linux系统做一点配置。修改`/etc/security/limits.conf`文件，添加如下内容到文件后面：

```text
* - nofile 65536
* - nproc 4096
```

还需要安装一下[EC2 Discovery Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/6.5/discovery-ec2.html)插件：

```shell
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install discovery-ec2
```

配置access_key：

```shell
sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add discovery.ec2.access_key
```

输入access_key，就是上面创建IAM用户的`访问密钥 ID`。

配置secret_key:

```shell
sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add discovery.ec2.secret_key
```

输入上面保存的secret_key。

修改Elasticsearch的配置文件`/etc/elasticsearch/elasticsearch.yml`。添加如下内容到文件后面：

```text
cluster.name: es
node.name: ${HOSTNAME}
network.host: _ec2_
discovery.zen.hosts_provider: ec2
discovery.ec2.endpoint: ec2.us-east-1.amazonaws.com
```

这个配置需要注意的一点是`discovery.ec2.endpoint`需要改成你自己实例所在区域，可以查看[AWS文档](https://docs.aws.amazon.com/zh_cn/general/latest/gr/rande.html#ec2_region)

另外，如果你启动的ec2实例内存只有1g，最好把Elasticsearch启动的内存设置为512M。修改配置文件`/etc/elasticsearch/jvm.options`，把`-Xms1g`和`-Xmx1g`改为`-Xms512M`和`-Xmx512M`。

配置已经好了，启动一下elasticsearch试试看。

```shell
sudo systemctl start elasticsearch
```

访问`http://你的EC2公有DNS(IPv4):9200`你会看到下面类似的信息（有可能会部分不太一样），说明实例内部的Elasticsearch安装成功了。

```json
{
  "name" : "ip-172-31-11-76.ec2.internal",
  "cluster_name" : "es",
  "cluster_uuid" : "-tedL2ggQpKQ-1sqF5abLA",
  "version" : {
    "number" : "6.5.4",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "d2ef93d",
    "build_date" : "2018-12-17T21:17:40.758843Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

好了，单个实例内部的Elasticsearch已经安装成功，然后就是制作AMI映像了。

## 制作AMI

先执行下面两个命令，把Elasticsearch停止，把保存节点信息的目录删掉

```shell
sudo systemctl stop elasticsearch
sudo rm -r /var/lib/elasticsearch/nodes
```

这个步骤非常简单，在ec2控制台里面的实例列表里面，在创建的实例上面点击右键->映像->创建映像。然后填入映像名称和映像描述，假设我们取名为`Elasticsearch-6.5.4-v1`。点右下角的创建映像，稍等一会就可以在左侧菜单的映像下面的AMI里面看到刚刚创建的映像了。

制作好了AMI映像之后就可以把用来制作ec2的实例停止，或者终止了，已经不再需要这个实例了。

## 创建Auto Scaling Group

简单介绍一下AWS的Auto Scaling Group作用，这个功能可以根据系统一些状态和指标来自动扩展或者缩实例数量，这样就可以在负载或者请求增加的时候AWS自动的增加机器来应对更大的请求，当请求下降的时候AWS自动减少机器以达到节省开支的作用。

下面就来创建一个自动弹性组。左侧菜单AUTO SCALING下面Auto Scaling组，点击`创建Auto Scaling组`按钮，然后点击右下角的`开始使用`按钮。Auto Scaling组需要一个启动配置，就是一个启动模板，AWS在增加机器的时候会按照这个模板来启动机器，模板里面包括了AMI、存储、安全组

先是创建启动配置。第一步需要选择一个AMI，点击我的AMI，选择我们创建好的`Elasticsearch-6.5.4-v1`映像，像下图这样。
![](/img/aws/aws_es_cluster/2.png)

实例类型根据自己的需要选择，这里使用默认的即可，点击`下一步: 配置详细信息`。填入启动配置的名称`es-startup-template`其他保持默认，点击`下一步:添加存储`。存储保持默认即可，继续点击`下一步:配置安全组`。选择一个现有的默认安全组，点击`审核`，然后点击`创建启动配置`，弹出的选择密钥对的对话框跟上面一样，使用上面创建上密钥对。

接下来就是配置Auto Scaling组了。输入组名`es-auto-scaling-group`，组大小里面输入`2`，保持最小/正常实例数量为2个。子网随便选择一个即可，点击`下一步:配置扩展策略`。扩展策略选择`使用扩展策略调整此组的容量`，下面的第二个输入框，输入`4`，保持实例数量在2-4之间弹性扩展和缩减。`扩展组大小`使用平均CPU利用率，在目标值输入框填入2，表示在CPU利用率在2%以上就会增加实例数量，点击`下一步:配置通知`。在通知里面可以填入你的邮箱地址，在实例变化的时候就可以收到通知邮件了，需要注意的一点是`发送通知到:`这个输入框里面不能输入空格（中文没有试过）。点击`审核`，最后点击`创建Auto Scaling组`。

然后到ec2实例列表就可以看到刚刚创建的Auto Scaling组里面的实例了，初始就是启动2个实例。启动完成之后在ec2实例列表就可以看到已经启动好了两个实例了
![](/img/aws/aws_es_cluster/3.png)

然后拷贝其中一个实例的公有DNS(IPv5)到浏览器，加上`:9200/_cat/nodes?v`访问就可以看到当前集群当中有2个节点了。

![](/img/aws/aws_es_cluster/4.png)

好了，到这里一个Elasticsearch的集群已经在aws上搭建好了，而且在请求增加，整个集群平均CPU利用率上升的时候就会自动的增加实例数量了。如果需要在负载下降到某个点的时候把实例数量减少怎么配置呢。还是到Auto Scaling组列表里面，选择`es-auto-scaling-group`，在下面的面板当中选择`扩展策略`。在扩展策略里面点击`添加策略`，然后点击`创建一个简单的扩展策略`链接，输入名称，`执行策略的时间:`使用后面的`创建新警报`，然后选择每当Avarage中的CPU利用率`是 < 10 百分比`，就是这里把`>=`修改成`<`，输入框里面填入`10`，然后点击`创建警报`。在`请执行以下操作:`里面选择`删除`，后面的数字填入`1`，每次减少一个实例。

现在就把自动增加和减少实例的配置做好了，当负载达到10以上的时候就会自动增加实例数量，当负载减少到10以下的时候就会一个一个实例的减少了。

这里面有一个问题，当你的app在访问Elasticsearch集群的时候，你要以哪个实例的域名（公有DNS(IPv4)）访问呢？

有两个办法，一个办法可以在Auto Scaling组里面设置某个实例不会被关掉，一直保持运行，然后以这个实例的域名访问即可，如下图。

![](/img/aws/aws_es_cluster/5.png)

另外一个办法就是创建一个负载均衡器，负载均衡器指向到这个Auto Scaling组即可，这样就可以在app指定访问负载均衡器的域名就可以了。下面介绍负载均衡器的配置方法。

## 配置负载均衡器

选择左侧负载均衡器菜单，点击创建负载均衡器按钮，选择HTTP/HTTPS那个点击创建。
![](/img/aws/aws_es_cluster/6.png)

输入负载均衡器名称`elb-es`，侦听器里面端口填入`9200`，在`可用区`里面选择你Auto Scaling组所在的区，然后再勾选一个其他的（至少要先两个），点击`下一步:配置安全设置`。有一个警告，先忽略，直接点击`下一步:配置安全组`，保持默认选择一个现有的默认安全组，继续点击`下一步:配置路由`。名称里面填入`elb-es-target`，端口改成`9200`，点击`下一步:注册目标`。保持默认，继续点击`下一步:审核`，然后点创建。

这时候会创建一个目标群组，名字为`elb-es-target`，这个群组是负载均衡器用来查询所有存活的机器的一个配置。然后到Auto Scaling组里面，选择`es-auto-scaling-group`，点击上方的`操作`里面的`编辑`，在弹出框里面的`目标组`一项选择刚刚创建的目标群组`elb-es-target`。配置完成之后就做完了，这时候就可以使用elb-es里面的那个公有DNS(IPv4)来作为app访问的一个url了，注意同样需要加上9200端口。

上面设置Auto Scaling组的目标组的作用是，在增加或者减少实例数量的时候会自动把实例注册到目标群组里面。

上面配置完成之后需要等待一会，等待Auto Scaling组里面的机器注册完成，可以到负载平衡下面的目标群组菜单里面选择`elb-es-target`，查看下面`目标`标签里面的已注册目标，检查是否已经注册成功。

## 总结

虽然配置比较多，但是每一个步骤相对还是比较简单的，综合起来就可以缓存一个稳定的Elasticsearch集群了，当你的业务请求量减少的时候，机器数量会自动减少以节省你的开支，当请求增加的时候可以自动增加机器数量以应对业务的增加，而不需要你关心硬件的事情。云计算很方便，但也不便宜。
