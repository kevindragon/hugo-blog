---
title: 我来学Kotlin-基础之基础类型
date: 2017-06-14 11:45:13
tags: [Kotlin, Kotlin教程]
categories: [编程语言]
---

在Kotlin，一切皆对象，我们可以在任何变量上调用成员函数和属性。有些类型是内置的，因为它们的实现是经过优化的，但是对于用户来说，它们看起来就像普通的类。本节描述这些类型中的大多数： numbers, characters, booleans and arrays.

<!-- more -->

# 数字类型

Kotlin以一种接近Java的方式处理数字，但不完全一样。例如，对于数字没有隐式转换，而在某些情况下，字面量略有不同。Kotlin提供如下内置的数字类型（与Java接近）：

| Type   | Bit width |
| ------ | --------- |
| Double | 64        |
| Float  | 32        |
| Long   | 64        |
| Int    | 32        |
| Short  | 16        |
|  Byte  | 8         |

**注意：在Kotlin中字符不是数字**

## 字面量

整数字面量如下：

- 十进制：123
  + Long长整形使用L标识：123L
- 十六进制：0x0F
- 二进制：0b00001011

**注意：不支持八进制字面量**

Kotlin支持常见的浮点数字：

- Double双精度是默认的浮点数字：123.5, 123.5e10
- 单精度使用f或者F标识：123.5f

## 使用下划线（1.1或更高版本）

比较长的数字使用下划线可以更易于阅读，这和Swift的使用方法一样

```Kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

## Representation

在JVM平台，数字被物理地存储为JVM原始类型，除非我们需要一个可空的数字引用（例如Int?）或者泛型。在后一种情况下数字会被装箱（boxed）。

**注意：装箱后的数字不一定会保持引用**

```Kotlin
val a: Int = 10000
print(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
```

但是会保持值的相等（使用两个等号`==`）。三个等号（`===`）会判断是否是同一个对象，就是上面说的引用，内存地址相等。

```Kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```
## 显示转换

由于表示方法的不同，较小的类型不是较大类型的子类型。如果是，我们将会遇到以下问题：

```Kotlin
// 假设有如下代码，实际是不能编译的：
val a: Int? = 1 // 一个Int对象（在Java是java.lang.Integer）
val b: Long? = a // 隐式转换为Long对象（在Java是java.lang.Long）
print(a == b) // 惊讶！这会打印出`false`，两个等号使用的是equals方法来判断相等性，a和b的类型不一样，所以打开出来是false
```

所以不仅在引用上，而且相等性在这里也默默的消失了。因此较小类型不隐式的转换成较大类型。这意味着，在没有显示转换时，我们不能将一个`Byte`类型赋值给一个`Int`类型：

```Kotlin
val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
```

使用显示转换到更宽（较大）的数字：

```Kotlin
val i: Int = b.toInt() // OK: explicitly widened
```

每一个数字类型支持下面的转换：

- toByte(): Byte
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble(): Double
- toChar(): Char

隐式转换的缺失很少会被注意到，因为类型是从上下文推断出来的，算术运算符做了重载，例如：

```Kotlin
val l = 1L + 3 // Long + Int => Long
```

## 运算符

Kotlin支持标准的数学运算集，它们是以类成员的方式定义的（但是编译器会将调用优化为相应的指令）。

对于位运算，不是使用特殊字符（像>>, <<），而是以命名函数的方式进行中缀调用，例如：

```Kotlin
val x = (1 shl 2) and 0x000FF000
```

这里是完整的位运算列表（只针对`Int`和`Long`类型）：

- `shl(bits)` – 左移 (Java's <<)
- `shr(bits)` – 右移 (Java's >>)
- `ushr(bits)` – 不带符号的右移 (Java's >>>)
- `and(bits)` – 与
- `or(bits)` – 或
- `xor(bits)` – 异或
- `inv()` – 按位反转：00000001 -> 11111110

# 字符类型

字符使用`Char`表示，不能直接当作数字来对待（相对于Java字符与ascii整数是可以互换的）：

```Kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```

字符字面量使用单引号：`'1'`。特殊字条使用反斜杠进行转义，支持：`\t`, `\b`, `\n`, `\r`, `\'`, `\"`, `\\`, `\$`。其他的字符可以使用Unicode转义语法：`\uFF00`。我们可以显示的把字符转换成数字，例如把`'1'`转换成`1`：

```Kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```

跟数字一样，在需要可空类型的时候会进行装箱，并且不会保持引用的不变。

# 布尔类型

布尔类型使用`Boolean`来表示，只有两个值：`true`和`false`。同样在需要可空类型的时候会进行装箱。内置的运算符如下：

- `||` - 或
- `&&` - 与
- `!` - 非

# 数组

数组在Kotlin以`Array`类表示，它具有get和set方法（可以使用[]重载调用），以及size属性，还有一些其他有用的方法：

```Kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

要创建一个数组，我们可以使用库函数`arrayOf()`和传递一些值给函数，`arrayOf(1, 2, 3)`会创建一个数组[1, 2, 3]。`arrayOfNulls(size: Int)`库函数用来创建给定大小的数组，所有元素的值都为null。另一个选择是使用工厂方法，传一个大小值和一个给定索引值返回元素值的函数：

```Kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

上面说过，`[]`操作符全调用`get()`和`set()`方法。

**注意：不像Java，Kotlin的数组类型是不能转变的。这意味着不能把`Array<String>`赋值为`Array<Any>`，以防止可能的运行时错误（但是仍然可以使用`Array<out Any>`），这是Kotlin添加的协变和逆变概念**

Kotlin还有其他的类来表示其他的的基础类型数组而不需要装箱：`ByteArray`, `ShortArray`, `IntArray`等等。这些类并不是继承自`Array`，但是他们有着相关的方法和属性。每一个都有相应的库函数：

```Kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

# 字符串

字符串使用`String`类表示，并且是不可变的（immutable）。字符串是以字符组成，每一个元素是一个字条，可以使用索引来访问每个位置上的字符值：`s[i]`，可以使用for循环来迭代：

```Kotlin
for (c in str) {
    println(c)
}
```

## 字符串字面量

Kotlin有两种字面量：双引号和三个双引号包裹。前一种可以包含转义字符，后一种可以显示的换行。

```Kotlin
val s = "Hello, world!\n"
val text = """
    for (c in "foo")
        print(c)
"""
```

可以使用`trimMargin()`方法删除原始字符串前面的空白，默认使用竖线`|`，但是可以其他的字符达到同样的效果，例如`trimMargin(">")`：

```Kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

## 字符串模板

字符串可以包含模板表达式，例如可以把一小代码写在字符串里面，并且把代码的结果与字符串其他部分进行组装。字符串模板以`$`开头和一个简单的名字组成：

```Kotlin
val i = 10
val s = "i = $i" // 结果为："i = 10"
```

或者以任意被花括号包裹的表达式：

```Kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // 结果为："abc.length is 3"
```

两种字符串方式都支持字符串模板，如果想使用`$`本身怎么办呢，可以使用`${'$'}`：

```Kotlin
val price = """
${'$'}9.99
"""
val tpl = "${'$'}a"
```
