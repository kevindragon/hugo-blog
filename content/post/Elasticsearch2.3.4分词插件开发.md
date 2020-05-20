---
title: Elasticsearch2.3.4分词插件开发
date: 2016-12-16 15:42:54
tags: [Elasticsearch, 分词]
---

## 0. Elasticsearch的版本是2.3.4

首先你得有自己的一`Analyzer`和`Tokenizer`，可以参考前面的[一篇文章](/2016/12/16/自定义Lucene的分词器Analyzer)

## 1. pom.xml

使用maven来构建项目，需要包含elasticsearch的依赖，否则编译不了

```xml
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>2.3.4</version>
    <scope>provided</scope>
</dependency>
```

然后es的plugin需要一个描述文件`plugin-descriptor.properties`，把这个文件放在`src/main/resources`这个目录，打包后安装到es里面就会出现在插件目录下了。这个文件需要写入下面的内容：

```properties 
description=${project.description}
version=${project.version}
name=${elasticsearch.plugin.name}
site=${elasticsearch.plugin.site}
jvm=${elasticsearch.plugin.jvm}
classname=${elasticsearch.plugin.classname}
java.version=${elasticsearch.plugin.java.version}
elasticsearch.version=${elasticsearch.version}
```

这些才使用了maven在`pom.xml`定义的内容，直接定义在`pom.xml`的根路径下：

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <elasticsearch.plugin.name>elasticsearch-analysis-demo</elasticsearch.plugin.name>
    <elasticsearch.plugin.site>false</elasticsearch.plugin.site>
    <elasticsearch.plugin.jvm>true</elasticsearch.plugin.jvm>
    <elasticsearch.plugin.java.version>1.8</elasticsearch.plugin.java.version>
    <elasticsearch.version>2.3.4</elasticsearch.version>
    <elasticsearch.plugin.classname>org.elasticsearch.plugin.AnalysisDemoPlugin</elasticsearch.plugin.classname>
</properties>
```

最重要的就是`plugin-descriptor.properties`里面的`classname`，也就是`pom.xml`定义的`elasticsearch.plugin.classname`节点里面的类名，这个类就是es加载plugin的入口。

## 2. 继承Plugin

现在来看Java代码，分词插件入口类需要继承`org.elasticsearch.plugins.Plugin`类，并实现三个方法：

1. public String name()  返回插件名字
2. public String description  返回插件的描述
3. public void onModule(AnalysisModule model)  可以使用`model`这个`AnalysisModule`对象添加自定义的分词器

上代码：

```java
package org.elasticsearch.plugin;

import com.lexiscn.elasticsearch.hylanda.HylandaAnalysisBinderProcessor;
import org.elasticsearch.index.analysis.AnalysisModule;
import org.elasticsearch.plugins.Plugin;

public class AnalysisDemoPlugin extends Plugin {

	public static String PLUGIN_NAME = "elasticsearch-analysis-demo";

    @Override
    public String name() {
        return PLUGIN_NAME;
    }

    @Override
    public String description() {
        return PLUGIN_NAME;
    }

    public void onModule(AnalysisModule model) {
        model.addProcessor(new HylandaAnalysisBinderProcessor());
    }
}
```

在`model.addProcessor`这个方法调用的时候，需要一个`org.elasticsearch.index.analysis.AnalysisModule.AnalysisBinderProcessor`对象。所以还需要实现一个继承处`AnalysisBinderProcessor`的类：

## 2. 实现AnalysisBinderProcessor

```java
package org.elasticsearch.demo;

import org.elasticsearch.index.analysis.AnalysisModule;

public class DemoAnalysisBinderProcessor extends AnalysisModule.AnalysisBinderProcessor {

    @Override
    public void processAnalyzers(AnalyzersBindings analyzersBindings) {
        analyzersBindings.processAnalyzer("hylanda_analyzer", DemoAnalyzerProvider.class);
    }

    @Override
    public void processTokenizers(TokenizersBindings tokenizersBindings) {
        tokenizersBindings.processTokenizer("hylanda_tokenizer", DemoTokenizerFactory.class);
    }
}
```

这个继承自`AnalysisBinderprocessor`的类根据需要实现`processAnalyzers`和`processTokenizers`方法。一个添加analyzer，一个是添加tokenizer，第一个参数就是analyzer或者tokenizer的名字。这里就可以添加自己的分词器了，不过还需要一点点努力。如果是Analyzer还需要一个继承自`org.elasticsearch.index.analysis.AbstractIndexAnalyzerProvider`的类；如果是Tokenizer还需要一个继承自`org.elasticsearch.index.analysis.AbstractTokenizerFactory`的类。

## 3. 实现AbstractTokenizerFactory

先来看继承自`AbstractTokenizerFactory`的`Tokenizer`的工厂类:

```java
package org.elasticsearch.demo;

import org.apache.lucene.analysis.Tokenizer;
import org.elasticsearch.common.inject.Inject;
import org.elasticsearch.common.inject.assistedinject.Assisted;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.index.Index;
import org.elasticsearch.index.analysis.AbstractTokenizerFactory;
import org.elasticsearch.index.settings.IndexSettingsService;


public class DemoTokenizerFactory extends AbstractTokenizerFactory {
    public DemoTokenizerFactory(Index index, Settings indexSettings, String name, Settings settings) {
        super(index, indexSettings, name, settings);
    }

    /**
     * 必须实现此方法，而且需要标记为@Inject
     *
     * @param index
     * @param indexSettingsService
     * @param name
     * @param settings
     */
    @Inject
    public DemoTokenizerFactory(
            Index index, IndexSettingsService indexSettingsService,
            @Assisted String name, @Assisted Settings settings) {
        super(index, indexSettingsService.getSettings(), name, settings);
    }

    @Override
    public Tokenizer create() {
        return new DemoTokenizer();
    }
}
```

在2.3.4的es版本里面使用的是Guice依赖注入框架，所以必须实现上面标注了@Inject的构造方法。还要实现`public Tokenizer create()`方法，然后在create方法里面new自己的Tokenizer即可。

## 4. 实现AbstractIndexAnalyzerProvider

再来看看Analyzer的Provider：

```java
package org.elasticsearch.demo;

import org.apache.lucene.analysis.Analyzer;
import org.elasticsearch.common.inject.Inject;
import org.elasticsearch.common.inject.assistedinject.Assisted;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.env.Environment;
import org.elasticsearch.index.Index;
import org.elasticsearch.index.analysis.AbstractIndexAnalyzerProvider;
import org.elasticsearch.index.settings.IndexSettingsService;

public class DemoAnalyzerProvider extends AbstractIndexAnalyzerProvider {
    public DemoAnalyzerProvider(Index index, Settings indexSettings, String name, Settings settings) {
        super(index, indexSettings, name, settings);
    }

    /**
     * 必须实现此方法，而且需要标记为@Inject
     *
     * @param index
     * @param indexSettingsService
     * @param env
     * @param name
     * @param settings
     */
    @Inject
    public DemoAnalyzerProvider(Index index, IndexSettingsService indexSettingsService, Environment env,
                                @Assisted String name, @Assisted Settings settings) {
        super(index, indexSettingsService.getSettings(), name, settings);
    }

    @Override
    public Analyzer get() {
        return new DemoAnalyzer();
    }

}
```

跟上面的Tokenizer一样，必须实现上面代码标注了@Inject的构造方法，然后就需要实现`public Analyzer get()`方法，在这个get方法里面可以new自己的Analyzer

## 5. package

这需要使用到maven的assembly插件来对代码以及相应的资源打包，这样就可以使用es提供的plugin命令行工具进行安装了。

首先需要在`pom.xml`添加assembly插件：

```xml
<plugin>
	<artifactId>maven-assembly-plugin</artifactId>
	<configuration>
		<outputDirectory>${project.build.directory}/releases/</outputDirectory>
		<descriptors>
			<descriptor>${basedir}/src/main/assembly/plugin.xml</descriptor>
		</descriptors>
	</configuration>
	<executions>
		<execution>
			<phase>package</phase>
			<goals>
				<goal>single</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

然后添加`src/main/assembly/plugin.xml`，这个文件也可以放到其他地方，主要是上面的`descriptor`节点指定

```xml
<?xml version="1.0"?>
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd">
    <id>release</id>
    <formats>
        <format>jar</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>
        <fileSet>
            <directory>${project.basedir}/src/main/plugin-metadata</directory>
            <outputDirectory>/</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>${project.basedir}/lib</directory>
            <outputDirectory>/</outputDirectory>
        </fileSet>
    </fileSets>
    <files>
        <file>
            <source>${project.basedir}/src/main/resources/plugin-descriptor.properties</source>
            <filtered>true</filtered>
            <outputDirectory>/</outputDirectory>
        </file>
    </files>
    <dependencySets>
        <dependencySet>
            <outputDirectory>/</outputDirectory>
            <useProjectArtifact>true</useProjectArtifact>
            <excludes>
                <exclude>org.elasticsearch:elasticsearch</exclude>
                <exclude>org.apache.lucene:lucene*</exclude>
                <exclude>com.spatial4j:spatial4j</exclude>
            </excludes>
        </dependencySet>
    </dependencySets>
</assembly>
```

根据自己需要做相应修改。基本上就是上面一些东西，这些坑都填平了，其他就就可以进行打包安装了

## 6. 打包安装

在source根目录下使用`mvn clean install`。然后到ES根目录使用下面的命令进行插件的安装:

    ./bin/plugin install file:///absolute/path/to/source/target/release/xxxx.jar

Windows使用：

    .\bin\plugin.bat install file:///C:/absolute/path/to/source/target/release/xxxx.jar
    `

好了。
