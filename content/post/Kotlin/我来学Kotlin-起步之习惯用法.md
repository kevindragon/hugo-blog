---
title: 我来学Kotlin-起步之习惯用法
date: 2017-06-13 16:44:02
tags: [Kotlin, Kotlin教程]
categories: [编程语言]
---

## 数据类

```Kotlin
data class Customer(val name: String, val email: String)
```

Customer提供了以下方法：

- getters，如果成员变量使用var定义那么还默认提供setters
- equals()
- hashCode()
- toString()
- copy()
- component1(), component2() ...

这跟Scala的`case class`基本上差不多，一般在做数据交互/传输时用

## 函数默认值

```Kotlin
fun foo(a: Int = 0, b: String = "") { ... }
```

## 列表过滤

```Kotlin
val positives = list.filter { x -> x > 0 }
```

或者更短一点

```Kotlin
val positives = list.filter { it > 0 }
```

默认使用it来接收单个的参数

## 字符串插值

```Kotlin
println("Name $name")
```

## 对象检查

```Kotlin
when (x) {
    is Foo -> ...
    is Bar -> ...
    else   -> ...
}
```

非常精简，再也不要写Java那个烦人的`instanceof`

## 遍历映射/列表

```Kotlin
for ((k, v) in map) {
    println("$k -> $v")
}
```

k, v分别是map的key和value，是list的index和value

## 使用范围

```Kotlin
for (i in 1..100) { ... }  // closed range: includes 100
for (i in 1 until 100) { ... } // half-open range: does not include 100
for (x in 2..10 step 2) { ... }
for (x in 10 downTo 1) { ... }
if (x in 1..10) { ... }
```

## 只读映射/列表

```Kotlin
val list = listOf("a", "b", "c")
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
```

如果是集合是不是还有`setOf`，命名的统一性也是Kotlin的一个亮点，如果你使用过php的话，感觉函数总是记不住。

## 映射访问

```Kotlin
println(map["key"])
map["key"] = value
```

## 懒值(Lazy)

```Kotlin
val p: String by lazy {
    // compute the string
}
```

这段代码在定义的时候是不会执行变量的值的，只有在使用这个变量的时候才会求值

## 扩展

这是Kotlin一个非常强大的功能，可以对原来已经存在的类添加一系列的新方法，扩展别人的包非常方便

```Kotlin
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```

## 单例

做Java是不是会经常被问单例模式是怎么实现的，Kotlin直接使用object来定义，直白得不要不要的，这和Scala也是一样的

```Kotlin
object Resource {
    val name = "Name"
}
```

## null值检查

```Kotlin
val files = File("Test").listFiles()
println(files?.size)
println(files?.size ?: "empty")
```

相比Java的`if (files != null)`优雅太多了。files如果是空值，新手写代码如果不做Null检查，就会碰到最常见的NullException。Kotlin使用问号来检查，并且null值会一值传递到最后。

```Kotlin
val data = ...
val email = data["email"] ?: throw IllegalStateException("Email is missing!")
```

Kotlin使用`?:`也是非常的简洁，如果`data["email"]`有值，直接使用，如果没有就抛出异常。如果想只有在有值的情况下

```Kotlin
val data = ...

data?.let {
    ... // execute this block if not null
}
```

映射空值检查

```Kotlin
val data = ...

val mapped = data?.let { transformData(it) } ?: defaultValueIfDataIsNull
```

## when表达式作为返回值

Kotlin是基于表达式，而不是基于语句的。也就是说任何的语言结构都是有返回值的，这也是和Scala是一样的。

```Kotlin
fun transform(color: String): Int {
    return when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
}
```

这里`when`是一个表达式，表达式是有值的，如果color是Green，那么这个`when`的值就是2

## try/catch

同样try/catch也是有值的

```Kotlin
fun test() {
    val result = try {
        count()
    } catch (e: ArithmeticException) {
        throw IllegalStateException(e)
    }

    // Working with result
}
```

## if

if也是表达式，也是有值的

```Kotlin
fun foo(param: Int) {
    val result = if (param == 1) {
        "one"
    } else if (param == 2) {
        "two"
    } else {
        "three"
    }
}
```

如果上面最后那个else没有写的话，编译是通不过的，如果`if`作为一个表达式，那么必须覆盖所有的情况。

## 使用with调用多个方法

这个特性也比较酷，结构跟python的差不多：

```Kotlin
class Turtle {
    fun penDown()
    fun penUp()
    fun turn(degrees: Double)
    fun forward(pixels: Double)
}

val myTurtle = Turtle()
with(myTurtle) { //draw a 100 pix square
    penDown()
    for(i in 1..4) {
        forward(100.0)
        turn(90.0)
    }
    penUp()
}
```

## Java 7的try-with-resources语句

在Java 7的写法如下

```Java
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
}
```

在Kotlin写法更加直观

```Kotlin
val stream = Files.newInputStream(Paths.get("/some/file.txt"))
stream.buffered().reader().use { reader ->
    println(reader.readText())
}
```

## 泛型

```Kotlin
//  public final class Gson {
//     ...
//     public <T> T fromJson(JsonElement json, Class<T> classOfT) throws JsonSyntaxException {
//     ...

inline fun <reified T: Any> Gson.fromJson(json): T = this.fromJson(json, T::class.java)
```

## 可空布尔类型使用

```Kotlin
val b: Boolean? = ...
if (b == true) {
    ...
} else {
    // `b` is false or null
}
```
