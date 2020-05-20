---
title: 我来学Kotlin-起步之代码规范
date: 2017-06-14 10:31:10
tags: [Kotlin, Kotlin教程]
categories: [编程语言]
---

Kotlin官方提供了一份代码规范，主要介绍了命名，冒号，匿名函数，类定义，方法与属性以及Unit，比较简短。

<!-- more -->

下面是Kotlin官方提倡的一份代码规范

## 命名

如果有不确定的可以参考Java代码规范，例如：

- 使用驼峰法来命名，避免使用下划线
- 类型/类使用大写字母开头
- 方法/属性/函数以小宝字母开头
- 使用4个空格缩进
- public方法应该写上注释，以便生成文档的时候会显示注释内容

## 冒号

如果冒号出现在类型与类型之间，前后都应该有一个空格。而如果出现在变量名与其类型之间，前面不要空格，后面应该有一个空格：

```Kotlin
interface Foo<out T : Any> : Bar {
    fun foo(a: Int): T
}
```

## 匿名函数/Lambdas

匿名函数在花括号内部两边应留一个空格，与前面调用的函数名之间留一个空格，剪头两边也需要有空格：

```Kotlin
list.filter { it > 10 }.map { element -> element * 2 }
```

匿名函数应该尽量简短，避免嵌套。推荐使用`it`代替自定义的参数名。在嵌套匿名函数里，最好显示的自定义参数名称。

## 类定义格式

如果类包含较少的参数，应该写在一行：

```Kotlin
class Person(id: Int, name: String)
```

如果比较长的话，构造函数的参数一个写一行，前面使用4个空格缩进，右圆括号另起一行。如果使用了继承，那么父类的构造函数或者接口应该写在括号后面：

```Kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name) {
    // ...
}
```

如果继承了父类以及多个接口，父类与括号写在一行，每个接口单独写一行：

```Kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker {
    // ...
}
```

## Unit

函数返回`Unit`时，函数的返回值类型可以省略：

```Kotlin
fun foo() { // ": Unit" is omitted here
}
```

## 方法与属性

有些情况下，无参数的方法与只读属性表达的意思一样，可以互换使用。尽管在语义上是相似的，但还是有一些约定在什么情况使用什么方式。下面的情况提倡使用属性：

- 不会抛出异常
- 时间复杂度为`O(1)`
- 求值非常廉价，或者求值后有缓存
- 返回固定值
