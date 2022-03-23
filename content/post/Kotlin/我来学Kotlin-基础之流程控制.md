---
title: 我来学Kotlin-基础之流程控制
date: 2017-06-15 12:24:00
tags: [Kotlin, Kotlin教程]
categories: [编程语言]
---

流程控制与Java差不多，但是更加强大，像`if`, `when`是语句也是表达式，表达式有返回值，可以赋值给一个变量。

<!-- more -->

## if表达式

在Kotlin里，`if`是一个表达式，也就是他会返回一个值。因此没有三目运算符（? :），因为完全可以用`if`替代

```Kotlin
// Traditional usage
var max = a
if (a < b) max = b

// With else
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}

// As expression
val max = if (a > b) a else b
```

`if`分支可以是代码块，代码块的最后一个表达式的值就是`if`的返回值。

```Kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

如果`if`被当作表达式而不是语句使用时，这个表达式必须要有`else`分支。

## when表达式

`when`代替了像c系语言的`switch`结构。最简单的形式如下：

```Kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

`when`会把参数与每个分支进行匹配，直到匹配成功。`when`也可以用作语句或者表达式。如果被当作表达式使用，那么匹配到的分支的代码就是`when`表达式的值；如果是语句，整个语句的值会被忽略。（跟`if`是一样的，如果分支代码是代码块，代码块的最后一个表达式的值就是`when`的值）。

`else`分支在没有任何其他的分支匹配的时候进行求值。如果`when`被当作表达式使用，那么`else`分支是必须的，除非编译器可以推断已经包含了所有的情况。

如果多个情况需要做同样的处理，那么可以使用逗号组合多个条件：

```Kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

我们还可以使用任意的表达式作为条件：

```Kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

还可以检查值是否在或者不在某个范围或者集合内：

```Kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

另外还可以检查一个值是否是某个类型，由于有智能转换，在判断类型后可以直接访问该值的方法和属性，而不需要额外的检查：

```Kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

`when`可以作为`if-else`分支结构的替代，如果没有参数，那么分支的条件是一个返回布尔值的表达式，当分支条件为`true`的时候支持分支内的代码：

```Kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

## for循环

`for`可以在任何提供了迭代器的东西上迭代，语法如下：

```Kotlin
for (item in collection) print(item)
for (item: Int in ints) {
    // 可以使用代码块
}
```

上面提到可以迭代任何提供了迭代器的东西，也就是：

- 有一个成员或者扩展方法为`iterator()`，且`iterator`的返回类型具有如下特点：
  - 有一个成员或者扩展方法为`next()`
  - 有一个返回`Boolean`值的成员或者扩展方法为`hasNext()`

所有以上三个方法必须被标记为`operator`。

数组在`for`上的循环被编译成基于索引的循环，而不需要创建迭代器对象。

如果想在数组或者列表上使用索引，可以用下面的方法：

```Kotlin
for (i in array.indices) {
    print(array[i])
}
```

`array.indices` 这个属性其实是返回一个范围`range`，在范围上迭代会被编译优化而不需要使用额外的对象开销。

另一个选择，你可以使用`withIndex`函数同时得到索引和值：

```Kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

## while循环

`while`和`do..while`跟我们常见的是一样的：

```Kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```

## break和continue

Kotlin在循环中支持`break`和`continue`语句。`Scala`是没有像Java和Kotlin这样的`break`和`continue`语句的
