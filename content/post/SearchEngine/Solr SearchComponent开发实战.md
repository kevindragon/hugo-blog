---
title: Solr SearchComponent开发实战
date: 2019-03-01 20:18:54
tags: [Solr]
---

先介绍一下Solr的`SearchHandler`，`SearchHandler`是用来处理查询请求的配置。一个SearchHandler里面包含了若干个`SearchComponent`，实际的逻辑处理就在SearchComponent里面进行处理，还可以自行开发符合自己业务逻辑的SearchComponent，然后经过配置可以灵活的组合各种SearchComponent。各个SearchComponent就像流水线一样处理请求，各自负责自己的逻辑处理。

<!-- more -->

基于Solr 7.5.0测试

## 配置

最常使用的`/select`查询，使用的就是下面默认的配置：

```xml
<requestHandler name="/select" class="solr.SearchHandler">
  <lst name="defaults">
    <str name="echoParams">explicit</str>
    <int name="rows">10</int>
  </lst>
</requestHandler>
```

里面配置的defaults可以参考[官方文档](https://lucene.apache.org/solr/guide/7_6/requesthandlers-and-searchcomponents-in-solrconfig.html#default-components)

除了最常使用的默认搜索配置之外，这个`/select`配置还支持`First-components`和`Last-componets`的配置，也就是说在defaults的前面或者后面可以添加一些SearchComponent。具体的流程可以参考下面的图：

```
                           defaults
                         +-----------+
                         | query     |
+------------------+     | facet     |     +-----------------+
| First-Components | --> | mlt       | --> | Last-Components |
+------------------+     | highlight |     +-----------------+
                         | ......    |
                         +-----------+
```

如果添加了First-Components和Last-Components之后，完整的配置看起来就是下面的样子：

```xml
<requestHandler name="/select" class="solr.SearchHandler">
  <arr name="first-components">
    <str>mycomponent1</str>
    <str>mycomponent2</str>
  </arr>
  <lst name="defaults">
    <str name="echoParams">explicit</str>
    <int name="rows">10</int>
  </lst>
  <arr name="last-components">
    <str>spellcheck</str>
    <str>mycomponent3</str>
  </arr>
</requestHandler>
```

还可以完全自定义整个查询的handler的流程，不使用默认的的SearchComponent：

```xml
<requestHandler name="/select_2" class="solr.SearchHandler">
  <arr name="components">
    <str>mycomponent1</str>
    <str>query</str>
    <str>mycomponent2</str>
    <str>debug</str>
    <str>mycomponent3</str>
  </arr>
</requestHandler>
```

可以结合自己实现的SearchComponents以及官方提供的一起来实现一个符合自己业务逻辑的查询请求。

## 代码实现

先上代码，看一下一个TestComponent的实现最基本的方法需要实现哪些：

```java
package test.solr.components;
import java.io.IOException;
import org.apache.solr.handler.component.ResponseBuilder;
import org.apache.solr.handler.component.SearchComponent;
public class TestComponent extends SearchComponent {  // 1
    @Override
    public void prepare(ResponseBuilder rb) throws IOException {}  // 2
    @Override
    public void process(ResponseBuilder rb) throws IOException {}  // 3
    @Override
    public void finishStage(ResponseBuilder rb) {  // 4
      super.finishStage(rb);
    }
    @Override
    public String getDescription() {
      return null;
    }
}
```

### 继承

首先需要继承Solr的`SearchComponent`类(1)，最少需要实现`prepare`和`process`两个方法(2和3)，finishStage方法有默认的实现，如果没有需要可以不重写该方法。

在SearchHandler类里面有处理components的整个流程，一共有5个阶段：

1. ResponseBuilder.STAGE_START
2. ResponseBuilder.STAGE_PARSE_QUERY
3. ResponseBuilder.STAGE_EXECUTE_QUERY
4. ResponseBuilder.STAGE_GET_FIELDS
5. ResponseBuilder.STAGE_DONE

在SearchHandler与SearchComponent的执行的过程中，需要注意区分是不是分布式的执行方式。像SolrCore或者一个Shard的情况下就不是分布式；还有在请求从master分发到每个shard的时候，也不是分布式了。

#### prepare方法

首先执行的是prepare方法。

prepare方法在请求到达的机器那个节点会执行一次，然后在每个实际处理的shard上会再执行一次，假如一个collection有两个shard，那么prepare会被执行三次。可以通过`rb.isDistributed()`方法判断是第一次，还是在shard上面执行的。

#### process方法

然后就是执行process方法，这个方法是在shard上面执行的，所有跟索引数据相关的操作（查询，高亮等等）都可以放到在这个方法里面进行处理。

#### finishStage方法

上面提到了components的5个阶段，每个阶段都会调用这个finishStage方法，第5个方法ResponseBuilder.STAGE_DONE是不会调用到finishStage方法的，所以最后一个方法被调用的阶段是ResponseBuilder.STAGE_GET_FIELDS，如果有需要合并的数据需要判断一下是不是在这个阶段。

### 最后

展示一下最后的示例代码，在返回的结果没有添加一个字段：

```java
public class CheckComponent extends SearchComponent {
    @Override
    public void prepare(ResponseBuilder responseBuilder) throws IOException {}
    @Override
    public void process(ResponseBuilder responseBuilder) throws IOException {}
    @Override
    public void finishStage(ResponseBuilder rb) {
        if (rb.stage != ResponseBuilder.STAGE_GET_FIELDS) return;
        rb.rsp.add("check_info", "add by check component");
    }
    @Override
    public String getDescription() {
        return null;
    }
}
```

在finishStage方法里面，为rb.rsp这个返回的数据里面添加了一个`check_info`节点，内容为`add by check component`


后续再丰富本文内容......