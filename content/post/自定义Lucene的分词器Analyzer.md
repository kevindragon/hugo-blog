---
title: 自定义Lucene的分词器Analyzer
date: 2017-10-01 15:04:54
tags: [Lucene]
---

## 0、先定义一下分词的结果

从最简单的做起，把输入的内容按一个字一个字进行切分。这里只是为了说明自定义的`Analyzer`以及`Tokenizer`如何写。`Analyzer`需要一个`Tokenizer`来进行切词，所以还需要定义一个自己的`Tokenizer`。

## 1、先从Analyzer开始

Lucene版本为5.5以上

定义自己的`Analyzer`需要继承`org.apache.lucene.analysis.Analyzer`，并且实现`TokenStreamComponents createComponents(String s)`方法。咱们的`Analyzer`名字叫`DemoAnalyzer`。上代码：

```java
import org.apache.lucene.analysis.Analyzer;

public class DemoAnalyzer extends Analyzer {
    @Override
    protected TokenStreamComponents createComponents(String s) {
        return new TokenStreamComponents(
                new DemoTokenizer()
        );
    }
}
```

在`createComponents`方法里面直接new一个`TokenStreamComponents`对象，对象里面包含的是自定义的`DemoTokenizer`对象，主要的分词工作就是在这个类里面完成。

## 2、定义Tokenizer

自定义的`Tokenizer`需要继承`org.apache.lucene.analysis.Tokenizer`，并且实现`boolean incrementToken()`方法。这个方法比较特殊：

1. 此方法并不是一次性调用，而是进行的迭代调用，也就是说一次是读取一个分词，第二次再读取下一个分词，依次类推。

2. 需要一系列的`Attribute`来存放对应的一个分词（也就是`term`）的信息，包括词本身、类型（type），位置，词的始末偏移量等等。这个是通过调用`addAttribute`方法实现的。

3. `input`这个`Reader`并不需要自己传入，这个就是输入的需要分词的文本在父类已经实现了`setReader(Reader input)`方法，Lucene自己会调用此方法传入文本

看代码

```java
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.apache.lucene.analysis.tokenattributes.OffsetAttribute;
import org.apache.lucene.analysis.tokenattributes.PositionIncrementAttribute;
import org.apache.lucene.analysis.tokenattributes.TypeAttribute;

import java.io.IOException;

// 自定义的DemoTokenizer需要继承org.apache.lucene.analysis.Tokenizer类
// 并实现incrementToken和reset方法，这个方法相当于一个iterator方法，Lucene框架会不停的调用这个方法
// 直到这个方法返回false。调用这个对象的大致的伪代码逻辑如下：
// obj = new DemoTokenizer(input);
// notEnd? = obj.incrementToken();
// if notEnd? {
//     read attributes from obj
// }
// attributes表示在这人类里面调用addAttribute方法添加的attribute，包括分出来的词，类型，位置等等
public class DemoTokenizer extends Tokenizer {

    // 保存当前读取的字符位置信息
    private int position = 0;

    // 保存分出来的词
    protected CharTermAttribute charAttr =
            addAttribute(CharTermAttribute.class);
    // 保存当前分出来的词的类型，可以随便定义
    protected TypeAttribute typeAttr = addAttribute(TypeAttribute.class);
    // 保存当前词的位置
    private final PositionIncrementAttribute positionAttr =
            addAttribute(PositionIncrementAttribute.class);
    // 保存当前词的偏移量
    private final OffsetAttribute offsetAtt = addAttribute(OffsetAttribute.class);

    @Override
    public boolean incrementToken() throws IOException {
        // 调用这个方法很重要，必须在下面设置attributes之前调用
        clearAttributes();

        // 保存当前读取到的字符
        char[] c = new char[1];

        // 读取一个字符到变量c里面
        int count = this.input.read(c, this.position, 1);

        // 返回false代表读完了
        if (count == -1) {
            return false;
        }

        // 把当前读取到的字符保存到定义的attributes
        charAttr.append(c[0]);
        typeAttr.setType("Char");
        positionAttr.setPositionIncrement(position+1);
        offsetAtt.setOffset(correctOffset(position), correctOffset(position+1));

        // 返回true表示后面还有词需要处理，当前还没有读完
        return true;
    }

    // 正确覆写此方法也非常重要，因为在Lucene框架外面很有可能这个类只会被初始化一次
    // 所以如果使用的一些位置信息不正确会导致读取对应的位置错误
    @Override
    public void reset() throws IOException {
        super.reset();
        // 需要把上一个字段使用过的位置信息归0，否则当前这个字段的位置是不正确的
        this.position = 0;
    }
}
```

首先在类的内容需要几个存放`Attribute`的属性，比如`CharTermAttribute charAttr`，在调用`incrementToken`的时候需要先清空再append进`charAttr`。

这里只是一个字一个字的进行切分，所以在`incrementToken`里面只需要读一个字符即可`this.input.read(c, this.position, 1)`，然后append到`charAttr`里面。

这个方法的返回值需要注意，返回`false`表示已经读完的`input`里面的内容，后面没有文本了。返回`true`表示后面还有文本需要切分。

最后是`reset`方法，这里我们使用了`position`这个成员变量在存放每次读取下一个字符的开始位置，在Lucene调用`reset`的时候应该把`position`归零。

## 结尾

这样就实现了自定义的`Analyzer`，如果想做其他方法的分词，就可以套用这个外壳了。
