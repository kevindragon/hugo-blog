---
title: 如何通过Apache POI在Word当中添加引用（尾注、参考文献）
date: 2023-04-24T06:36:02+08:00
tags: [Apache POI, Java, 参考文献]
---

## 简介

Apache POI是一个用于读写Microsoft Office格式文件（如Word文档、Excel电子表格和PowerPoint演示文稿）的Java API。它是Apache软件基金会的一个开源项目，旨在提供一个易于使用的接口来处理Office文件。Apache POI支持各种Office文件格式，包括早期的二进制文件格式和新的XML文件格式。它提供了许多类和方法来读取、写入和操作Office文件，使得开发人员可以轻松地创建、修改和读取各种Office文件。Apache POI是Java开发人员的一个强大工具，可以帮助他们在Java应用程序中集成Office文件的处理功能。

<!--more-->

## 组件

Apache POI有非常多的组件，分别用于读写不同格式的文档：

POIFS - 读写OLE 2文档，这是一种二进制格式的Microsoft文档格式的组件

HSSF and XSSF - 读写Microsoft Excel XLS格式档案的组件

HWPF and XWPF - 读书Microsoft Word格式文档的组件，HWPF读写97到2003格式，XWPF读写最新的2007+格式

HSLF and XSLF - 读写Microsoft PowerPoint文档的组件

HPSF - 读写OLE 2文档属性的组件

HDGF and XDGF - 读Microsoft Visio文档的组件

HPBF - 读取Publisher文档的组件

HMEF for TNEF (winmail.dat) - 读取Outlook附件的组件

HSMF - 读取Outlook信息的组件

## 使用XWPF组件写一个带有引用的文档

### 创建文档

使用下面的语句创建一个空文档。类`XWPFDocument`还支持传入一个`InputStream`对象，读取一个已经存在的文档，还可以继续写入内容到这个文档当中。

```java
XWPFDocument document = new XWPFDocument();

// 读取一个存在的文档
// XWPFDocument document = new XWPFDocument(new FileInputStream(new File("input.docx");
```

### 追加段落

使用document对象创建一个段落对象，在`paragraph`对象上可以设置样式、缩进、对齐等段落属性。

```java
XWPFParagraph paragraph = document.createParagraph();
```

### 追加Run

这个Run是写入具体内容的对象，可以写入文档、图片等资源。Run对象是通过paragraph对象来创建的。

```java
XWPFRun run = paragraph.createRun();
```

### 写入文本内容

```java
run.setText("这是一个引用，引用的内容是Apache POI");
```

### 保存文档

```java
document.write(new FileOutputStream("example.docx"));
```

### 输出的结果

下面是保存后的效果

![simple](/img/Java/ApachePOIWordReference/simple.png)

从上面的过程来看，代码非常简单，几乎都是这么一个过程。

## 插入引用

![Word add reference](/img/Java/ApachePOIWordReference/word_add_reference.png)

如果使用Word文档插入一个尾注，需要找到“引用”菜单，然后使用工具栏上面的“插入尾注”就可以创建一个尾注了。

![Word added a reference](/img/Java/ApachePOIWordReference/word_added_a_reference.png)

尾注默认使用罗马数字来编号的，如果要改变编号格式，找到引用菜单的插入尾注工具栏组右下角的“脚注和尾注”，点击后会弹出脚注和尾注对话框，在格式一栏可以设置编号格式

![Format of reference number](/img/Java/ApachePOIWordReference/format_of_reference_number.png)

找到格式一栏，选择相应的编号格式

![Reference number selection](/img/Java/ApachePOIWordReference/reference_number_selection.png)

好了，上面是演示了如何使用Microsoft Word软件插入尾注，那么使用Apache POI软件包该如何做呢？

这里有一个事情需要先了解，在Apache POI的XWPF设置引用的编号格式非常的困难，我们使用变通的办法。先利用上面的方法创建一个阿拉伯数字编号的引用格式，然后把所有的内容都删除，保存为一个空的文档，用这个空的文档当作模板，在创建 `XWPFDocument` 对象的时候，传入这个空文档，那么在代码里面插入尾注的时候就会以阿拉伯数字编号了。

### 创建新的Run

使用paragraph对象创建一个新的`Run`对象，在这个新的`Run`对象上利用代码 `refRun.setSubscript(VerticalAlign.SUPERSCRIPT)` 设置引用编号为上标。然后使用refRun对象创建一个尾注的引用，但是这是尾注还没有创建，所以尾注的ID暂时还没有设置

```java
XWPFDocument document = new XWPFDocument(new FileInputStream(new File("template.docx")));
XWPFEndnotes endnotes = document.createEndnotes();
// ......
run.setText("这是一个引用，引用的内容是Apache POI");
// ......
XWPFRun refRun = paragraph.createRun();
refRun.setSubscript(VerticalAlign.SUPERSCRIPT);
CTFtnEdnRef ctFtnEdnRef = refRun.getCTR().addNewEndnoteReference();
```

### 创建一个尾注

上面的代码当中我们在`document`对象上创建了一个用于管理尾注的对象endnotes，就是利用这个对象来创建单个的尾注对象

```java
XWPFEndnote endnote = endnotes.createEndnote();
XWPFParagraph endnoteParagraph = endnote.createParagraph();
XWPFRun endnoteRun = endnoteParagraph.createRun();
endnoteRun.setText("  Apache POI - the Java API for Microsoft Documents, https://poi.apache.org");
```

一个尾注对象也可以像`document`对象一样，创建一个`paragraph`对象后，再创建一个`run`对象，然后使用`run`对象写入文本内容。

### 关联引用和尾注

把尾注的编号与内容当中的引用进行关联，关联的方式就是使用尾注的ID来达成的。

```java
ctFtnEdnRef.setId(endnote.getId());
```

最后来看看输出的结果

![Untitled](/img/Java/ApachePOIWordReference/result.png)

## docx其实就是一个包含一系列xml的压缩包

![Untitled](/img/Java/ApachePOIWordReference/file_tree_of_docx.png)

使用解压工具，对example.docx进行解压后，可以看到在example目录所有的xml文件，Apache POI的XWPF组件其实就是对这些xml的读取与写入操作。比如docProps目录下面的就是对整个文档的属性的设置，word目录下面是对内容的一些配置与数据，document.xml就是内容，endnotes.xml就是尾注的内容了。

根据上面的内容，我们看两个最先要的xml文件，word/document.xml和word/endnotes.xml

### word/document.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<w:document mc:Ignorable="w14 w15 w16se w16cid w16 w16cex w16sdtdh wp14"
    ...>
    <w:body>
        <w:p w14:paraId="73499B62" w14:textId="47C51F1E" w:rsidR="000E1D70"
            w:rsidRDefault="000E1D70">
            <w:pPr>
                <w:rPr>
                    <w:rFonts w:hint="eastAsia" />
                </w:rPr>
            </w:pPr>
        </w:p>
        <w:p>
            <w:r>
                <w:t>这是一个引用，引用的内容是Apache POI</w:t>
            </w:r>
            <w:r>
                <w:rPr>
                    <w:vertAlign w:val="superscript" />
                </w:rPr>
                <w:endnoteReference w:id="5" />
            </w:r>
        </w:p>
        <w:sectPr w:rsidR="000E1D70">
            <w:endnotePr>
                <w:numFmt w:val="decimal" />
            </w:endnotePr>
            <w:pgSz w:w="11906" w:h="16838" />
            <w:pgMar w:top="1440" w:right="1800" w:bottom="1440" w:left="1800" w:header="851"
                w:footer="992" w:gutter="0" />
            <w:cols w:space="425" />
            <w:docGrid w:type="lines" w:linePitch="312" />
        </w:sectPr>
    </w:body>
</w:document>
```

有一个三级的标签是 `<w:p><w:r><w:t>`，这是跟代码里面的 `paragraph→run→text` 这个层级是对应的。`<w:endnoteReference w:id=”5”>` 这个跟代码里面的`ctFtnEdnRef.setId`是对应的，设置引用与尾注的关联。`<w:endnotePr><w:numFmt>` 标签设置了引用编号的格式，`decimal`表示阿拉伯数字。

### word/endnotes.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<w:endnotes mc:Ignorable="w14 w15 w16se w16cid w16 w16cex w16sdtdh wp14"
    ...>
    <w:endnote w:type="separator" w:id="-1">
        <w:p w14:paraId="26DFFEF0" w14:textId="77777777" w:rsidR="00A603FE"
            w:rsidRDefault="00A603FE" w:rsidP="004947E6">
            <w:r>
                <w:separator />
            </w:r>
        </w:p>
    </w:endnote>
    <w:endnote w:type="continuationSeparator" w:id="0">
        <w:p w14:paraId="1C9C8E1D" w14:textId="77777777" w:rsidR="00A603FE"
            w:rsidRDefault="00A603FE" w:rsidP="004947E6">
            <w:r>
                <w:continuationSeparator />
            </w:r>
        </w:p>
    </w:endnote>
    <w:endnote w:type="normal" w:id="5">
        <w:p>
            <w:r>
                <w:rPr>
                    <w:rStyle w:val="FootnoteReference" />
                </w:rPr>
                <w:endnoteRef />
            </w:r>
            <w:r>
                <w:t xml:space="preserve">  Apache POI - the Java API for Microsoft Documents, https://poi.apache.org</w:t>
            </w:r>
        </w:p>
    </w:endnote>
</w:endnotes>
```

这个xml文件里面我们看到了 `<w:endnote … w:id=”5”>` 这么一个标签，与document.xml里面的`w:id`就对应起来了，里面还是 `<w:p><w:r><w:t>` 这样的层级结构。

## 总结

好了，使用Apache POI插入尾注就介绍到这里。我们介绍了如何使用Apache POI的XWPF组件快速创建与写入内容到docx文档当中，主要介绍了如何使用编程的方式设置引用和尾注，修改编号格式。欢迎关注我的公众号，会不定期更新编程与AI相关技术与实践。

## 关注我

关注我的公众号，请把你关心的问题发送给我，我会深入研究并为你解答。
关注公众号及时获取编程、软件工程、Linux知识、AI理论与落地技术实践信息。

![QRcode](/img/qrcode_for_gh_d49b3390adad_258.jpg)

如果博客内容对您有帮助，也可以请我喝杯咖啡，你的支持是我不停的创作源泉。

![WeChat_Pay](/img/wechat_pay_w200.jpg)
