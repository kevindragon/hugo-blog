---
title: Solr7中的learnin to rank使用
date: 2018-03-19 14:32:06
tags: [Solr]
---

什么是learning to rank在此往篇文章不做介绍，这里只介绍如何使用。主要针对cloud模式启动的solr。使用的版本是solr-7.1.0，本文以外的请参考[官方文档](http://lucene.apache.org/solr/guide/7_1/learning-to-rank.html#installation-of-ltr)

# 安装

`solr-ltr-7.1.0.jar`已经自带在solr的发行包里面，只需要在`solrconfig.xml`配置即可，把下面的内容添加到`solrconfig.xml`

```xml
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-ltr-\d.*\.jar" />
<queryParser name="ltr" class="org.apache.solr.ltr.search.LTRQParserPlugin"/>
<cache name="QUERY_DOC_FV"
       class="solr.search.LRUCache"
       size="4096"
       initialSize="2048"
       autowarmCount="4096"
       regenerator="solr.search.NoOpRegenerator" />
<transformer name="features" class="org.apache.solr.ltr.response.transform.LTRFeatureLoggerTransformerFactory">
  <str name="fvCacheName">QUERY_DOC_FV</str>
</transformer>
```

添加了以上的配置后还需要把`dist`目录下的`solr-ltr-7.1.0.jar`拷贝到`server/solr-webapp/webapp/WEB-INF/lib/`目录下面，如果不添加的话，用rest api设置model的时候会报class not found exception。

<!-- more -->

# 上传feature

```json
[
    {
        "store": "featureStore",
        "name": "title",
        "class": "org.apache.solr.ltr.feature.SolrFeature",
        "params": {
            "q": "{!func}query({!v=title:${query}})"
        }
    },
    {
        "store": "featureStore",
        "name": "content",
        "class": "org.apache.solr.ltr.feature.SolrFeature",
        "params": {
            "q": "{!func}query({!v=content:${query}})"
        }
    },
    {
        "store": "featureStore",
        "name": "originalScore",
        "class": "org.apache.solr.ltr.feature.OriginalScoreFeature",
        "params": {}
    },
    {
        "store": "featureStore",
        "name": "level",
        "class": "org.apache.solr.ltr.feature.FieldValueFeature",
        "params": {
            "field": "level"
        }
    },
    {
        "store": "featureStore",
        "name": "year",
        "class": "org.apache.solr.ltr.feature.FieldValueFeature",
        "params": {
            "field": "year"
        }
    }
]
```

上面是四个feature：
1. 取标题的score，使用query函数，通过参数的形式传入查询词，[External Feature Information](http://lucene.apache.org/solr/guide/7_1/learning-to-rank.html#external-feature-information)。${query}是一个占位符，传参数是这样的efi.query='YOUR KEYWORD'
2. 取内容字段content的score，同上
3. originalScore是不使用ltr的原来默认的score值
3. 直接取level字段的值
4. 同上3，取year字段的值

使用下面的命令上传feature

```bash
curl -XPUT 'http://localhost:8983/solr/collection_name/schema/feature-store' -d "your json string" -H 'Content-type:application/json'
```

查看feature可以使用下面的url
`http://host:port/solr/collection_name/schema/feature-store`

查看具体的feature
`http://host:port/solr/collection_name/schema/feature-store/featureStore`

## 提取feature的值
上传完feature，就可以通过下面的url查看每个文档各个feature的值。

```
http://host:port/solr/collection_name/query?q=YOUR_KEYWORD&fl=id,score,[features store=featureStore efi.query='YOUR_KEYWORD']
```

# 上传model

```json
{
  "class": "org.apache.solr.ltr.model.LinearModel",
  "store": "featureStore",
  "name": "modelStore",
  "features": [
    {"name": "title"},
    {"name": "content"},
    {"name": "originalScore"},
    {"name": "level"},
    {"name": "year"}
  ],
  "params": {
    "weights": {
      "title": 0.420081550114,
      "content": -0.054514139045,
      "originalScore": 0.0413832540555,
      "level": -0.0150990612159,
      "year": 1.0
    }
  }
}
```

这个模型配置使用一个[线性函数](https://lucene.apache.org/solr/7_1_0//solr-ltr/org/apache/solr/ltr/model/LinearModel.html)来对搜索结果进行再次打分

查找模型
`http://host:port/solr/collection_name/model-store`

# 打分
在查询的时候指定rq参数使用ltr，然后指定使用的model，传入需要的参数即可。针对上面的feature和model使用下面的url进行查询

```
http://host:port/solr/collection_name/query?q=YOUR_KEYWORD&rq={!ltr model=modelStore reRankDocs=100 efi.query='YOUR_KEYWORD'}&fl=id,score,[features store=featureStore efi.query='YOUR_KEYWORD']
```

这个url可以使用ltr进行打分，同时还可以取到各个feature的值。
