---
title: Solr开发分词插件
date: 2018-05-19 14:59:20
tags: [Solr]
---

如果已经有了一个自定义的分词库，那要怎么才能集成到Solr里面去呢，本文带你一步一步入坑--不对--，带你集成自定义分词库。

如果还没有自定义的分词库，希望写自己的Solr分词插件，可以先参考[自定义Lucene的分词器Analyzer](/2016/12/16/自定义Lucene的分词器Analyzer/)

<!-- more -->

根据[自定义Lucene的分词器Analyzer](/2016/12/16/自定义Lucene的分词器Analyzer/)一文，假设现在已经具备了`DemoTokenizer`类。因为在Solr里面可以只需要Tokenizer就可以了，而不一定需要Analyzer。

先看一下Solr里schema配置分词的定义：

```xml
<fieldType name="text_demo" class="solr.TextField" positionIncrementGap="100">
  <analyzer>
    <tokenizer class="com.example.lucene.DemoTokenizerFactory" wordType="hello" />
    <!-- some other filters -->
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
```

这里配置了一个字段类型为`text_demo`，使用了`com.example.lucene.DemoTokenizerFactory`这个工厂方法来处理对应字段的分词，同时还设置了一个属性`wordType`值为`hello`，在后面的代码中读取这个属性的值。这里配置了`DemoTokenizerFactory`分词处理类，那么就必须要有这么一个类了：

```java
package com.example.lucene;

import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.util.TokenizerFactory;
import org.apache.lucene.util.AttributeFactory;
import java.util.Map;

// 需要继承org.apache.lucene.analysis.util.TokenizerFactory类并实现create方法
public class DemoTokenizerFactory extends TokenizerFactory {

    // 可以保存在上面schema定义的额外的属性参数
    private String wordType = "";

    // 需要一个构造方法，通过构造方法的args参数来读取schema设置的自定义的参数
    public DemoTokenizerFactory(Map<String, String> args) {
        super(args);
        // 调用完父类的构造函数后，可以读取schema里tokenizer节点定义的wordType参数
        // 保存在类属性里面，然后在create方法来使用这个传入的参数
        wordType = args.getOrDefault("wordType", "");
    }

    @Override
    public Tokenizer create(AttributeFactory factory) {
        // 在这里直接new一个tokenizer对象即可
        return new DemoTokenizer();
    }
}
```

通过上面定义的`DemoTokenizerFactory`工厂类，就可以提供Solr需要的自定义分词功能了。如果你的自定义分词器需要额外的参数，比如调用一个rest api来进行分词，需要一个host和port，就可以通过在schema定义属性的办法来使分词可以更有弹性和适应不同的rest api而不需要改Solr自定义分词代码。

最后记得添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-core</artifactId>
        <version>7.2.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-analyzers-common</artifactId>
        <version>7.2.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```
