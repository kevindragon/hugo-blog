---
title: 我来学Kotlin-基础之返回值与跳转
date: 2017-06-16 13:37:43
tags: [Kotlin, Kotlin教程]
categories: [编程语言]
---

Kotlin有三种结构性的跳转表达式：

- `return`默认从最近的函数或者匿名函数返回，后面的代码不再执行
- `break`跳出`break`最近的一个循环
- `continue`忽略后面的代码，跳转到下一个步骤

<!-- more -->

所有这些表达式都可以用作更大的表达式的一部分:

```Kotlin
val s = person.name ?: return
```

这个表达式的类型是`Nothing`类型

## break和continue标签

在Kotlin里所有的表达式都可以用一个标签（`label`）来标记。标签由一个合法的标识符跟上一个`@`符号缓存，例如：abc@, fooBar@都是合法的标签。要标记一个表达式，只需要把标签放在表达式前面即可：

```Kotlin
loop@ for (i in 1..10) {
    for (j in 1..10) {
        if (j > 5) break@loop
        println("i, j = $i, $j")
    }
}
```

上面这段代码的输出如下：

```
i, j = 1, 1
i, j = 1, 2
i, j = 1, 3
i, j = 1, 4
i, j = 1, 5
```

如果没有`loop`标签，那么break的行为是跳出里面一层的循环，输出结果还会有`i, j = 2, 1`等等，加了标签后直接跳出到最外层的循环。而`continue`是返回到标签处继续下一次的循环，下面的代码中带标签的`continue`和不带标签的`break`作用是一样的：

```Kotlin
abc@ for (i in 1..4) {
    for (j in 1..4) {
        if (j > 2) continue@abc
        println("i, j = $i, $j")
    }
}
for (i in 1..4) {
    for (j in 1..4) {
        if (j > 2) break
        println("i, j = $i, $j")
    }
}
```

## return

关键字`return`跟Java的差不多，但是功能要复杂一些，个人认为没有Scala的函数式那么纯粹。`return`是用在函数里面来返回一个值给调用者。

```Kotlin
fun foo(): Int {
    return 3
}
```

当执行到`return`时候，函数立即返回，仅仅只是最接近`return`的那个函数返回，看下面两个例子：

```Kotlin
fun baz() {
    (1..10).forEach(fun(value: Int) {
        if (value == 2) return
        println(value)
    })
}

fun jaz() {
    (1..10).forEach {
        if (it == 2) return
        println(it)
    }
}
```

baz函数会打印1, 3, 4, 5, 6, 7, 8, 9, 10，而没有2，因为在循环到2的时候匿名函数直接返回了，但是forEach还在继续执行。
而jaz函数仅仅打印1，这里使用的是lambda来代替显示的fun定义。

来看下下面两个函数`foo`和`bar`：

```Kotlin
fun foo() {
    (1..10).map lit@ {
        if (it == 2) {
            return@lit
        }
        println(it)
    }
}

fun bar(): Unit {
    (1..10).map {
        if (it == 2) {
            return
        }
        println(it)
    }
}
```

这两个函数与上面的baz, jaz是一样的，使用了标签的return，就像是使用匿名的fun定义一样，不会阻止循环的执行。

在使用forEach迭代的时候，默认有一个forEach@标签，可以直接使用`return@forEach`。另外使用了标签的return后面是可以带上一个值的，例如：`return@abc 123`
