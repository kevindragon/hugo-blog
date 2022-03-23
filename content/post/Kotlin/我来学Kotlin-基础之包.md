---
title: 我来学Kotlin-基础之包
date: 2017-06-14 17:43:44
tags: [Kotlin, Kotlin教程]
categories: [编程语言]
---

包对于模块化是非常重要的，Kotlin的包与Java的定义/语法类似，但是限制完全不同。

<!-- more -->

一个源文件的包定义应该放在代码开始处：

```Kotlin
package foo.bar

fun baz() {}

class Goo {}

// ...
```

所有的内容被包含在此包的定义下面，在上面的例子中，`baz()`的全名是`foo.bar.bax`，`Goo`的全名是`foo.bar.Goo`。如果没有指定包名，那么些文件的代码属于默认没有名字的包。

## 默认引入

有一些包被默认引入到每一个源文件中，不需要再另外显示的引入：

- kotlin.*
- kotlin.annotation.*
- kotlin.collections.*
- kotlin.comparisons.* (从1.1开始)
- kotlin.io.*
- kotlin.ranges.*
- kotlin.sequences.*
- kotlin.text.*

根据不同的平台还会默认引入其他的包：

- JVM:
  - java.lang.*
  - kotlin.jvm.*
- JS:
  - kotlin.js.*

## 引入 (Imports)

除了默认引入的一些包以外，每个文件应该包含自己的引入指令。

可以引入一个单独的名称，如：

```Kotlin
import foo.Bar // Bar可以直接使用，而不需要前面的前缀foo
```

或者一个包下面的所有内容（包，类，对象等等）：

```Kotlin
import foo.* // foo下面的所有内容都可以直接访问
```

如果有名称冲突了，可以使用`as`关键字在文件范围内重命名来消除冲突：

```Kotlin
import foo.Bar // Bar可以直接访问
import bar.Bar as bBar // 使用bBar代替'bar.Bar'
```

关键字`import`不局限于引入类，还可以引入其他的定义/声明：

- 顶层（包级别）的函数和属性
- 单例对象内定义的方法和属性
- 枚举常量

跟Java不同的一点是，Kotlin没有`import static`语法，所有的定义都是通过`import`关键字实现

## 顶层定义可见性

如果声明/定义被标记为了`private`，那么这个声明/定义仅仅对当前文件可见，其他包不能引用。
