---
title: Java的Exception体系
author: Kevin Jiang
date: 2022-03-21
---

众所周知，Java异常分为两大类：检查弄(checked)异常和非检查型(unchecked)异常。

## Java异常层级

先来看看Java异常层级图：

![](/img/Java/Java异常层次结构.png)

所有异常都是由Throwable继承而来，但在在下一层立即分解为两个分支：Error和Exception。

Error类层级结构描述了Java运行时系统的内部错误和资源耗尽错误，例如OutOfMemoryError。你的应用不应该抛出这种类型的对象。如果出现了这样的内部错误，除了通知用户，并尽力妥善终止程序之外，你几乎做了不什么处理。这各情况也非常少出现。

在设计Java程序时，着重关注Exception以及基子类。Exception又分解为两个分支：一个是派生于RuntimeException；另外一个包含其他异常。一般的使用规则是：由编程错误导致的异常属于RuntimeException；如果程序本身没有问题，但由于像I/O错误这类问题导致的异常属于其他异常。

派生于RuntimeException的异常通常包括如下问题：

- 错误的强制类型转换
- 数组访问越界
- 访问null指针

不派生于RuntimeException的异常包括：

- 访问一个不存在的文件
- 超载文件末尾继续读取数据
- 根据给定的字符串查找Class对象，而这个字符串表示的类并不存在

## 使用检查型和非检查型异常的基本准则

- 如果可以合理的期望客户端（调用端）从异常中恢复，则使用检查型异常
- 如果客户端（启用端）无法从异常中恢复，则使用非检查型异常

举个例子，在打开一个文件之前，我们需要先验证文件名。如果用户输入的文件名是非法的，我们可以抛出一个自定义的检查型异常。

```java
if (!isCorrectFileName(fileName)) {
    throw new InvalidFilenameCheckedException("非法的文件名");
}
```

这样的话，客户端（调用端）在得到异常之后，可以要求用户输入另外一个合法的文件名。

然而，如果客户端（调用端）传入了一个null或者空的字符串，意味着在代码当中出现了某些错误，这各情况可以抛出一个非检查型异常。

```java
if (fileName == null || fileName.isEmpty()) {
    throw new FilenameUncheckedException("未知错误");
}
```

## Spring Boot当中是使用RuntimeException还是其他的Exception

这个问题在我刚开始使用Spring Boot的时候困扰了我很久，现在我有了一个统一的答案，那就是上面的原则。但是再加一条，那就是优先考虑使用检查型(checked)异常，尽可能把问题都暴露出来，这样的话在别人调用代码的时候就知道什么异常可能会发生，然后做出必要的处理。而不是等到了上线之后发现一堆的runtime exception，这样对用户体验是非常差的。
