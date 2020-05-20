---
title: solr7配置同义词
date: 2018-01-10 14:59:47
tags: [Solr]
---

Solr的同义词可以满足绝大部分的需求，可以配置双向，也可以配置单向的同义词列表，功能非常的强大。

<!-- more -->

Solr的同义词可以在两个方面设置，一是索引的时候；二是查询的时候；或者你同时设置两者，同时设置的时候会非常复杂，会产生意想不到的结果。

# 先从基本配置开始

假设我们有同义词列表`synonyms_en.txt`
```
ability,capability
```

配置Solr的`fieldtype`

```
<fieldType name="text_en" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <!-- synonym -->
    <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms_en.txt" ignoreCase="true" expand="false"/>
    ......
  </analyzer>
  <analyzer type="query">
    ......
  </analyzer>
</fieldType>
```

先尝试索引时候设置同义词，直接在`analyzer type="index"`内部配置即可，`SynonymGraphFilterFactory`暂时还是一个试验性质的功能，以后的版本可能会有所改动。class还可以设置成`solr.SynonymFilterFactory`

Solr自带的同义词配置有很多的参数：
1. synonyms：指定同义词列表文件
2. ignoreCase：顾名思义是忽略大小写
3. expand：是否扩展同义词，意思是a和b是同义词，如果设置为true，那么搜索a的时候，同时搜索a和b，如果设置为false，那么搜索a的时候就搜索a，搜索b的时候也搜索a。也就是说单向双向的概念：如果设置为true，那么就是双向的；如果设置为false就是单向的，后面的词会转换成前面的词搜索。
4. tokenizerFactory：指定同义词列表使用的分词器
5. dedup：是否忽略重复，true为忽略

根据上面的设置，如果搜索`ability`，那么最终搜索的也是`ability`。如果搜索的是`capability`，最终搜索的时候会把`capability`转换成`ability`搜索。这样就达到了同义词搜索的功能。

## 查询时使用同义词

只需要把index部分的配置移到query部分即可，如下：

```
<fieldType name="text_en" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
    ......
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <!-- synonym -->
    <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms_en.txt" ignoreCase="true" expand="false"/>
    ......
  </analyzer>
</fieldType>
```

如果这时候把expand设置成true，那么搜索任何一个词，最终都会转换成搜索：`SynonymQuery(Synonym(field:ability field:capability))`。注意你看到的可能会是`SynonymQuery(Synonym(title.en:abil title.en:capabl))`，这是使用了porter stemming之后的结果。

我的请求语句如下：
```
http://192.168.0.106:8983/solr/lnc/select?debug=query&df=title.en&q=capability
```

# 单向，双向概念

这两个概念弄得还是有点模糊。第一：如果设置expand=false，那么同义词都会转换成第一个词进行索引（或者查询）；第二：如果设置expand=true，那么一个词会扩展成多个词同时查询；第三：使用`ability => capability`的配置时，如果查询`ability`，不管expand是true还是false，都会转换成对`capability`的搜索。

我理解的单向可以做成下面的列表：

```
ability => capability
ability => ability
```

这样搜索`ability`的时候，搜索会变成：`SynonymQuery(Synonym(field:ability field:capability))`，而搜索`capability`的时候，只会搜索`capability`。

通过这样的设置来达到单向同义词的目标。

# 中文分词为多个词的配置

这个问题的由来是这样的，假设我们使用的分词器叫`ABCTokenizerFactory`。现在有下面的同义词列表`synonyms_cn.txt`：

```
中华人民共和国,中国
```

这里假设`中华人民共和国`分词为`中华/人民/共和国`三个词

schema如下：

```
<fieldType name="text_cn" class="solr.TextField" positionIncrementGap="100">
    <analyzer type="index">
        <tokenizer class="ABCTokenizerFactory"/>
        ...
    </analyzer>
    <analyzer type="query">
        <tokenizer class="ABCTokenizerFactory"/>
        <filter class="solr.SynonymGraphFilterFactory"
                synonyms="synonyms_cn.txt"
                ignoreCase="true"
                expand="true"/>
        ...
    </analyzer>
</fieldType>
```

这样配置的时候，你搜索`中华人民共和国`是没有办法匹配到`中国`的。因为同义词有自己的分词器，如果这两个分词器的分词结果不相同，那么这个同义词设置基本上就是无效的，Synonyms的源码也可以体现这一点：

```
public class SolrSynonymParser extends SynonymMap.Parser {
  private final boolean expand;
  
  public SolrSynonymParser(boolean dedup, boolean expand, Analyzer analyzer) {
    super(dedup, analyzer);
    this.expand = expand;
  }
}
```

可以看到有一个analyzer，这个analyzer就是对同义词列表进行分词用的。

所以需要设置synonym的tokenizerFactory。如下：

```
<fieldType name="text_cn" class="solr.TextField" positionIncrementGap="100">
    <analyzer type="index">
        <tokenizer class="ABCTokenizerFactory"/>
        ...
    </analyzer>
    <analyzer type="query">
        <tokenizer class="ABCTokenizerFactory"/>
        <filter class="solr.SynonymGraphFilterFactory"
                synonyms="synonyms_cn.txt"
                ignoreCase="true"
                expand="true"
                tokenizerFactory="ABCTokenizerFactory"/>
        ...
    </analyzer>
</fieldType>
```

这样就解决了分词为多个的问题了。

# 总结

1. 基本用法，可以直接设置到只有一个analyzer节点的fieldtype中，这是同时对查询和索引生效
2. 单向双向的问题
3. 中文多分词的问题
