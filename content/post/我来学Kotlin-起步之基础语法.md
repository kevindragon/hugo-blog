---
title: 我来学Kotlin-起步之基础语法
date: 2016-03-12 13:35:02
tags: [Kotlin, Kotlin教程]
categories: [编程语言]
---

# 零、咱们先来看看[Kotlin](http://kotlinlang.org/)是啥语言。

JetBrains在2016年2月15日[Kotlin 1.0发布](http://blog.jetbrains.com/kotlin/2016/02/kotlin-1-0-released-pragmatic-language-for-jvm-and-android/)，这意味着Kotlin经过几年的发展已经稳定而且可以在生产环境使用。再来看一下其他人：

1. [如何评价 Kotlin 语言](https://www.zhihu.com/question/25289041)
2. [Android开发时你遇到过什么相见恨晚的工具或网站](https://www.zhihu.com/question/27140400/answer/54206284)
3. [Kotlin, the Swift of Android](https://blog.gouline.net/2014/08/31/kotlin-the-swift-of-android/)
4. [Using Project Kotlin for Android](https://docs.google.com/document/d/1ReS3ep-hjxWA8kZi0YqDbEhCqTt29hG8P44aA9W0DM8/edit?hl=zh-CN&forcehl=1#heading=h.v6i4iyf7kkxh)
5. [使用Kotlin进行Android开发](http://ragnraok.github.io/using-kotlin-to-write-android-app.html)

在网搜索一下就会发现已经有很多人在研究Kotlin并且使用其来开发Android，Android Studio原生就内置了Kotlin的工具和插件，可以方便的使用Kotlin开发Android。个人认为Kotlin没有Java那么多的历史包袱，而且写起来也没有Java那么的繁琐，但是又可以跟Java无缝结合，可以完全使用现在的各种库；相比Scala又简单不少。应该在JVM平台上可以发展得不错。我来学系列记录我自己学习的过程，同时希望可以能跟猿猴们一起探讨开发。下面的内容基本上是照着官方的文档来写，一来自己学习，二来跟大家一起讨论。

# 一、快速浏览语法
## 包定义
这个跟Java没有多少区别，在文件的最上面定义包的名字。如果需要引用其他包的内容，紧接着包定义的下面引入：

```Kotlin
package my.demo

import java.util.*

// ...
```

在Kotlin里面包的名字不一定要跟文件路径一致，一个源代码文件可以定义任意的包名。

## 定义函数
跟Java不同的是，在Kotlin里面可以定义单独的函数，而Java的函数必须属于某个类。下面是一个求和函数，把两个Int类型的变量相加，返回相加后的结果：

```Kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

还可以使用表达式的方式和类型推导来把上面的代码写得更加简洁：

```Kotlin
fun sum(a: Int, b: Int) = a + b
```

函数可以返回一个代表无值的值Unit，这有点像Java里面的void，但是跟void的意思大不相同。Unit类型表示一个值，但是这个值里面什么都没有：

```Kotlin
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
```

如果函数没有返回值，返回值缺省就是Unit

```Kotlin
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
```

## 定义变量
不可变的变量，原文是`Assign-once (read-only) local variable`，其他有一些语言里面称为`immutable`，个人更愿意使用immutable。也就是说一定赋了值之后，就不可以再被改变。immutable使用关键字`val`定义

```Kotlin
val a: Int = 1
val b = 1   // `Int` type is inferred
val c: Int  // Type required when no initializer is provided
c = 1       // definite assignment

// 上面都没有问题
// 如果再对任意一个变量赋值，编译的时候就会报错
c = 2
// Error:(11, 5) Kotlin: Val cannot be reassigned
```

跟immutable相对的就是mutable，使用关键字`var`定义：

```Kotlin
var x = 5 // `Int` type is inferred
x += 1
```

## 注释
跟Java一样，支持单行注释，使用两个斜杆。多行注释`/* your comments */`，使用/\*和\*/包裹。

```Kotlin
// This is an end-of-line comment

/* This is a block comment
   on multiple lines. */
```

## 字符模板
在字符串里面使用变量：

```Kotlin
fun main(args: Array<String>) {
    var a = 1
    // simple name in template:
    val s1 = "a is $a"

    a = 2
    // arbitrary expression in template:
    val s2 = "${s1.replace("is", "was")}, but now is $a"
    println(s2)
}
```

## 条件表达式
传统的在Java里面的方法：

```Kotlin
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
```

使用条件表达式后更加简洁。多说一句在Kotlin里面基本上所有的语句都是表达式，表达式有返回值，所以表达式可以用在任何需要值的地方，有点函数式编程的意思：

```Kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

## 可空类型，空值null检查
如果一个值有可能是空值，那么就应该标识类型为可能为空值，也就是在类型后面添加一个问号。

```Kotlin
fun parseInt(str: String): Int? {
    // ...
}
```

如果调用了函数，返回值为可空类型时，在使用这个值的时候必须先进行检查，像下面的变量x和y

```Kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    // 如果直接使用 `x * y` 会引发一个错误，因为x和y都是可空类型，使用前必须做检查
    if (x != null && y != null) {
        // 做了null check之后x和y会自动转换成对应的对应的类型，不再是可空类型
        println(x * y)
    }
    else {
        println("either '$arg1' or '$arg2' is not a number")
    }
}

fun main(args: Array<String>) {
    printProduct("6", "7")
    printProduct("a", "7")
    printProduct("a", "b")
}
```

## 类型查检，自动转换
使用`is`或者`!is`来判断一个对象是否为某类型。下面的代码里面使用了`Any`类型，我的理解是应该可以传递任何类型的值。如果一个immutable变量或者属性为检查的类型，那么可以把那个对象直接当做受检类型使用。

```Kotlin
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // `obj` is automatically cast to `String` in this branch
        return obj.length
    }

    // `obj` is still of type `Any` outside of the type-checked branch
    return null
}
```

或者像下面这相使用

```Kotlin
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` is automatically cast to `String` in this branch
    return obj.length
}
```

或者

```Kotlin
fun getStringLength(obj: Any): Int? {
    // `obj` is automatically cast to `String` on the right-hand side of `&&`
    if (obj is String && obj.length > 0) {
        return obj.length
    }

    return null
}
```

## 循环
在Kotlin里面使用`for`非常自然简洁。

```Kotlin
fun main(args: Array<String>) {
    val items = listOf("apple", "banana", "kiwi")
    for (item in items) {
        println(item)
    }
}
```

或者

```Kotlin
fun main(args: Array<String>) {
    val items = listOf("apple", "banana", "kiwi")
    for (index in items.indices) {
        println("item at $index is ${items[index]}")
    }
}
```

`while`循环

```Kotlin
fun main(args: Array<String>) {
    val items = listOf("apple", "banana", "kiwi")
    var index = 0
    while (index < items.size) {
        println("item at $index is ${items[index]}")
        index++
    }
}
```

`when`表达式，跟其他一些语言的模式匹配差不多

```Kotlin
fun describe(obj: Any): String =
when (obj) {
    1          -> "One"
    "Hello"    -> "Greeting"
    is Long    -> "Long"
    !is String -> "Not a string"
    else       -> "Unknown"
}

fun main(args: Array<String>) {
    println(describe(1))
    println(describe("Hello"))
    println(describe(1000L))
    println(describe(2))
    println(describe("other"))
}
```

## 范围 ranges
可以使用`in`检查变量是否在某个范围内

```Kotlin
fun main(args: Array<String>) {
    val x = 10
    val y = 9
    if (x in 1..y+1) {
        println("fits in range")
    }
}
```

或者使用`!in`检查不在某个范围

```Kotlin
fun main(args: Array<String>) {
    val list = listOf("a", "b", "c")

    if (-1 !in 0..list.lastIndex) {
        println("-1 is out of range")
    }
    if (list.size !in list.indices) {
        println("list size is out of valid list indices range too")
    }
}
```

在范围上使用循环

```Kotlin
fun main(args: Array<String>) {
    for (x in 1..5) {
        print(x)
    }
}
```

或者

```Kotlin
fun main(args: Array<String>) {
    for (x in 1..10 step 2) {
        print(x)
    }
    for (x in 9 downTo 0 step 3) {
        print(x)
    }
}
```

## 集合
循环集合同样非常简洁自然

```Kotlin
for (item in items) {
    println(item)
}
```

使用`in`检查一个对象是否在一个集合里面

```Kotlin
fun main(args: Array<String>) {
    val items = setOf("apple", "banana", "kiwi")
    when {
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }
}
```

使用高阶函数，lambda表达式过滤，处理集合的每个元素，Java在最新的版本里面也添加了这个特性，通过Stream Api实现。it就代表集合的每个元素

```Kotlin
fun main(args: Array<String>) {
    val fruits = listOf("banana", "avocado", "apple", "kiwi")
    fruits
    .filter { it.startsWith("a") }
    .sortedBy { it }
    .map { it.toUpperCase() }
    .forEach { println(it) }
}
```
