---
title: Solr7从搭建到入坑
date: 2017-10-08 14:10:26
tags: [Solr]
---

solr7搭建运行非常简单，首先下载压缩包，然后解压，到解压出来的目录执行：`bin/solr start`即可启动solr。以这种方式启动的solr是single core的模式。推荐部署cloud模式，可以有更好的伸缩性与扩展性，使用：`bin/solr start -c`可以启动cloud模式。下面主要针对cloud模式介绍solr7.2.1的搭建与查询。

<!-- more -->

solr7自带了几个例子，可以从例子可以深入solr。

## 以SolrCloud模式启动Solr
要启动solr，在Linux或者Unix执行：`bin/solr start -e cloud`，windows下执行：`bin\solr.cmd start -e cloud`，默认启动2个node组成的cloud server。这将会进入一个交互模式的启动过程，如果想跳过交互模式，启动命令添加参数`-noprompt`。如果是交互模式，一路回车到下面的提示：
```
Now let's create a new collection for indexing documents in your 2-node cluster.
Please provide a name for your new collection: [gettingstarted]
```
输入`techproducts`回车，我们创建的collection名字是`techproducts`。

继续回车到选择配置的地方：
```
Please choose a configuration for the techproducts collection, available options are:
_default or sample_techproducts_configs [_default]
```
这里有一个configSet的概念，configSet相当于是一个所有配置文件存放的地方，所有创建的collection都要在这里读取配置文件。
`_default`是一个基本的配置集，还有一个包含了`techproducts`的配置集，这个配置集可以用来支持我们想使用的techproducts样例数据。
输入`sample_techproducts_configs`回车。

恭喜，Solr已经启动好了！通过在浏览器访问`http://localhost:8983/solr`，这将是入坑的起点。
通过上面的步骤，创建了了两个节点（2 nodes），一个运行在7574端口，另外一个运行在8983端口，一个collection `techproducts`，两个shard（分片），每个分片又有两个拷贝，这样的配置可以在某个分片出现问题的时候快速恢复。

```
                            |-- 127.0.1.1:7574
               |-- shard1 --|-- 127.0.0.1:8983
techproducts --|
               |-- shard2 --|-- 127.0.1.1:7574
                            |-- 127.0.0.1:8983
```

## 索引techproducts样例数据
上面已经启动了solr，但是里面还没有数据，现在还不能做查询。

solr包含了一个可以提交多种数据格式的工具`bin/post`，下面使用这个脚本来提交样例数据。

> 目前`bin/post`还不兼容Windows，在Windows系统需要另外的办法提交数据

数据存放在`example/exampledocs`目录，这里面的数据格式包括JSON, CSV等等，这些格式的数据都可以一次性索引到solr里面。

#### Linux/Mac
```shell
bin/post -c techproducts example/exampledocs/*
```

#### Windows
```shell
java -jar -Dc=techproducts -Dauto example\exampledocs\post.jar example\exampledocs\*
```

恭喜！索引完数据就可以做查询了。

## 基本查询
Solr提供了基于http的REST接口以供多种方式进行查询。

浏览器访问`http://localhost:8983/solr/#/techproducts/query`界面，直接点击`Execut Query`按钮，右边将会显示10条以JSON格式返回的数据。在查询表单可以有很多的参数可以填写，最基本的就是`q`参数，需要查询的内容都可以通过`q`字段指定。还有很多其他的参数以后用到再一一介绍。

### 单个词查询
单个词的意思是不能再进行分词的某个词，指定`q`等于这个词即可，浏览器访问下面的url：
```
http://localhost:8983/solr/techproducts/select?q=foundation
```
可以得到下面的结果（实际有4条数据，下面截取了第一条数据）：
```json
{
  "responseHeader":{
    "zkConnected":true,
    "status":0,
    "QTime":8,
    "params":{
      "q":"foundation"}},
  "response":{"numFound":4,"start":0,"maxScore":2.7879646,"docs":[
      {
        "id":"0553293354",
        "cat":["book"],
        "name":"Foundation",
        "price":7.99,
        "price_c":"7.99,USD",
        "inStock":true,
        "author":"Isaac Asimov",
        "author_s":"Isaac Asimov",
        "series_t":"Foundation Novels",
        "sequence_i":1,
        "genre_s":"scifi",
        "_version_":1574100232473411586,
        "price_c____l_ns":799}]
}}
```
结果当中返回了文档的所有字段，如果只想返回`id`字段，需要指定`fl`参数：
```
http://localhost:8983/solr/techproducts/select?q=foundation&fl=id
```

### 字段查询
Solr的查询都是基于某些字段进行搜索的，上面默认是搜索的`_text_`字段，当然这个设置可以在配置集中修改。这里的`_text_`是一个copy field，包含的是其他字段的内容，如果只是想搜索某一个指定的字段，或者某几个指定的字段进行搜索，就需要指定字段名字了。假设需要搜索分类包含electronics的文档，参数`q`应该是`cat:electronics`，完整的url如下：
```
http://localhost:8983/solr/techproducts/select?q=cat:electronics
```
返回的数据如下：
```json
{
  "responseHeader":{
    "zkConnected":true,
    "status":0,
    "QTime":6,
    "params":{
      "q":"cat:electronics"}},
  "response":{"numFound":12,"start":0,"maxScore":0.9614112,"docs":[
      {
        "id":"SP2514N",
        "name":"Samsung SpinPoint P120 SP2514N - hard drive - 250 GB - ATA-133",
        "manu":"Samsung Electronics Co. Ltd.",
        "manu_id_s":"samsung",
        "cat":["electronics",
          "hard drive"],
        "features":["7200RPM, 8MB cache, IDE Ultra ATA-133",
          "NoiseGuard, SilentSeek technology, Fluid Dynamic Bearing (FDB) motor"],
        "price":92.0,
        "price_c":"92.0,USD",
        "popularity":6,
        "inStock":true,
        "manufacturedate_dt":"2006-02-13T15:26:37Z",
        "store":"35.0752,-97.032",
        "_version_":1574100232511160320,
        "price_c____l_ns":9200}]
     }}
```

### 短语查询
短语查询可以理解为：多个词的精确匹配（顺序一致），比如要查询文档中包含"multiple terms here"短语，需要使用双引号包起来。例如查询`CAS latency`：
```
http://localhost:8983/solr/techproducts/select?q="CAS+latency"
```
结果为：
```
{
  "responseHeader":{
    "zkConnected":true,
    "status":0,
    "QTime":7,
    "params":{
      "q":"\"CAS latency\""}},
  "response":{"numFound":2,"start":0,"maxScore":5.937691,"docs":[
      {
        "id":"VDBDB1A16",
        "name":"A-DATA V-Series 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - OEM",
        "manu":"A-DATA Technology Inc.",
        "manu_id_s":"corsair",
        "cat":["electronics",
          "memory"],
        "features":["CAS latency 3,   2.7v"],
        "popularity":0,
        "inStock":true,
        "store":"45.18414,-93.88141",
        "manufacturedate_dt":"2006-02-13T15:26:37Z",
        "payloads":"electronics|0.9 memory|0.1",
        "_version_":1574100232590852096},
      {
        "id":"TWINX2048-3200PRO",
        "name":"CORSAIR  XMS 2GB (2 x 1GB) 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) Dual Channel Kit System Memory - Retail",
        "manu":"Corsair Microsystems Inc.",
        "manu_id_s":"corsair",
        "cat":["electronics",
          "memory"],
        "features":["CAS latency 2,  2-3-3-6 timing, 2.75v, unbuffered, heat-spreader"],
        "price":185.0,
        "price_c":"185.00,USD",
        "popularity":5,
        "inStock":true,
        "store":"37.7752,-122.4232",
        "manufacturedate_dt":"2006-02-13T15:26:37Z",
        "payloads":"electronics|6.0 memory|3.0",
        "_version_":1574100232584560640,
        "price_c____l_ns":18500}]
  }}
```

### 组合搜索
默认情况下，搜索多个词的时候，solr使用或的逻辑来查询，也就是说只要多个词中的一个匹配到了就是命中了。当需要组合逻辑搜索的时候可以使用`+`、`-`前缀，`+`表示必须匹配的词，`-`表示不能匹配的词。比如要查询文档中包含`electronics`和`music`，查询表达式为`+electronics +music`。solr的查询请求url需要进行编码，请求语句变成如下：
```
http://localhost:8983/solr/techproducts/select?q=%2Belectronics%20%2Bmusic
```
返回一个结果

如果要搜索文档中包含`electronics`，*但是*不包含`music`，查询表达式为`+electronics -music`。
```
http://localhost:8983/solr/techproducts/select?q=%2Belectronics+-music
```
这个搜索会得到13个结果
