---
title: "Solr实现句子段落搜索"
date: 2018-04-17 12:14:30
tags: [Solr]
---

如果想要在Solr实现在某个句子/段落搜索的话，需要一点点技巧来达到这个需求。首先是在索引的时候做一点点手脚，然后再查询的时候再用点魔法就可以实现了。

<!-- more -->

# 索引处理

假设有一个文档，内容字段包含如下文本：

```
sentence abc. sentence axx.

another paragraph, sentence xyz.
```

这里有四个句子，以半角的.分割；两个段落，空行分割。

在索引的时候，先把句子拆开，使用`content.sentence`字段来索引每个句子，`content.sentence`为多值类型的字段。同样的处理方法把段落分开，使用`content.paragraph`字段来索引每个段落。配置如下

```xml
<field name="content.sentence" type="text" indexed="true" stored="true" multiValued="true"/>
<field name="content.paragraph" type="text" indexed="true" stored="true" multiValued="true"/>
```

索引里面的一点小技巧就是在配置分词的时候，设置`positionIncrementGap`参数，假设下面是配置`text`这个fieldtype：

```xml
<fieldType name="text" class="solr.TextField" positionIncrementGap="501">
```

这里设置`positionIncrementGap`为501，为啥是501后面详细说明。索引数据的时候记得拆分句子和段落到对应的字段，对应字段是数组类型，如句子：

```json
{"content.sentence": ["sentence abc.", "sentence axx.", "another paragraph.", "sentence xyz."]}
```

到这里索引就配置完了。

# 查询魔法

说是魔法，其实也就是一点小技巧，使用Solr的phrase query，然后设置slop即可。比如在句子中查询：

```
q=content.sentence:"sentence abc"~500
```

这个意思就是在`content.sentence`字段查询短语`sentence abc`，slop设置为500。

`slop`的意思是经过多少个步骤可以转换成`sentence abc`。这个跟[编辑距离](https://zh.wikipedia.org/zh-cn/%E7%B7%A8%E8%BC%AF%E8%B7%9D%E9%9B%A2)的概念是一样的，具体Solr/Lucene内部使用得是什么距离需要去阅读源码（或者看文档）。

多值字段每一个内容在保存位置信息的时候会根据`positionIncrementGap`的值设置偏移，如果第一个值最后的position是2，那么第二个值的开始就是`2+positionIncrementGap`。

这里的查询用得是500，上面索引配置的是501，差一个位置就可以确保前一个值和后一个值中间隔开500个字符，由于这个值是固定的，所以要确保一个段落要在500个词以内才可以。句子也是一样的道理。
