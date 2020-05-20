---
title: 我来学Kotlin-面向对象之类和继承
date: 2017-06-16 14:58:07
tags: [Kotlin, Kotlin教程]
categories: [编程语言]
---

使用关键字`class`来定义类：

```Kotlin
class Invoice {
}
```

类的定义由header（类名，主构造函数，参数及类型等）和header构成，主体使用花括号包裹。头和主体都不是必须的，如果没有主体花括号可以省略。

```Kotlin
class Empty
```

<!-- more -->

## 构造函数

类有一个主构造函数以及一个或多个辅构造函数。主构造函数是header的一部分，紧跟在类名的后面。

```Kotlin
class Person constructor(firstName: String) {
}
```

如果主构造函数没有注解或者可见性修饰符，关键字`constructor`可以省略：

```Kotlin
class Person(firstName: String) {
}
```

主构造函数不能包含任何的求值表达式，仅仅是定义参数及类型。如果要做一些初始化的工作，可以放在以`init`关键字开头的代码块当中：

```Kotlin
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```

主构造函数的参数可以在初始化代码块中使用。同样也可以在属性的定义及初始化的时候使用：

```Kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```

属性的定义及初始化可以简单的通过构造函数完成，同时可以定义属性的可变性（mutable/immutable），`var`是可变，`val`是不可变：

```Kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
    // ...
}
```

如果有可见性及注释修饰符，那么`constructor`关键字是不可以省略的，而且要请在`constructor`前面：

```Kotlin
class Customer public @Inject constructor(name: String) { ... }
```

## 辅助构造函数

辅助构造函数就像Java的多构造函数类似。在Kotlin使用`constructorr`关键字定义，在Java中是使用与类同名的函数名定义的。

```Kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

如果类有辅助构造函数同时还有主构造函数，那么辅助构造函数必须要显示的调用主构造函数，直接调用，或者调用其他的辅助构造函数。调用其他的构造函数可以使用`this`关键字：

```Kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

如果类没有定义任何的构造函数（主或者辅助构造函数），默认会有一个无参数的主构造函数，且其可见性是`public`的。如果要修改可见性，必须显示的定义一个空的构造函数：

```Kotlin
class DontCreateMe private constructor () { }
```

总结一下辅助构造函数，一句话：不管是直接调用主构造函数，可以调用其他的构造函数，一个辅助构造函数最终必须调用到主构造函数。

- 辅助构造函数 --> 主构造函数
- 辅助构造函数 --> 其他的辅助构造函数 --> 主构造函数

> 注：
> 在JVM平台（Kotlin可以编译到js或者native），如果主构造函数的所有参数都有默认值，编译器会生成另外一个无参数的构造函数，无参数构造函数会调用主构造函数。这使得可以不用传递参数来生成相应的对象
> `class Customer(val customerName: String = "")`
> 可以直接调用`Customer()`来实例化一个对象

## 创建实例

创建实例非常简单，就像调用一个函数一样。不像Java需要`new`关键字。

```Kotlin
class Invoice
class Customer(val customerName: String = "")
val invoice = Invoice()
val customer = Customer("Kevin Jiang")
val customer = Customer()
```

## 类成员

类可以包含：

- 构造函数和初始化代码块
- 方法
- 属性
- 嵌套类，内部类
- 单例对象

## 继承

所有的类都有一个共同的父类`Any`，这是没有指定父类的默认父类：

```Kotlin
class Example // 默认继承自`Any`
```

`Any`不是Java的`java.lang.Object`，`Any`只包含`equals()`, `hashCode()`, `toString()`三个方法。可以通过把父类放在冒号后面来指定父类，而且必须显示的指定调用父类的哪个构造函数，如果父类是无参娄的构造，也需要显示的写明。而且父类必须明确标记为`open`，不然就是`final`不能被继承：

```Kotlin
open class Base(p: Int)
class Derived(p: Int) : Base(p)
```

如果类没有主构造函数，每个辅助构造函数必须使用`super`关键字调用父类的构造函数，或者调用自己其他的构造函数。在下面的例子中，不同的构造函数可以调用父类不同的构造函数：

```Kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

**在Kotlin里，类默认是`final`的--也就是不能被继承的，必须使用`open`关键字标记才可以被继承**

总结一下继承当中的的构造函数定义，两句话：子类的主构造函数要调用父类的主构造函数；子类的辅助构造函数必须最终调用到父类的主构造函数：

- `class Sub(a: Int): Parent(a)`子类的主构造函数调用父类的主构造函数
- 子类有主构造函数时候，子类的辅助构造函数调用自己本身的主构造函数或者自己本身的辅助构造函数，这个情况跟上面辅助构造函数的总结一样，最后调用到了子类的请构造函数，最终还是调用到了父类的主构造函数
- 子类没有主构造函数，那么子类的辅助构造函数就需要使用`super`关键字调用父类的构造函数（主构造函数或者辅助构造函数），最后也是调用到了父类的主构造函数

## 方法覆写（Overriding Methods）

在Kotlin里，覆写父类的方法需要显示的指明，使用`override`关键字可以覆写父类的方法，而且父类的方法必须标注为`open`才可以被子类覆写：

```Kotlin
open class Base {
    open fun v() {}
    fun nv() {}
}
class Derived() : Base() {
    override fun v() {}
}
```

使用了`override`关键字的方法，默认是`open`的，可以被其子类覆写，如果不想让子类覆写，需要在前面使用`final`关键字指定：

```Kotlin
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```

## 属性覆写（Overriding Properties）

属性的覆写跟方法的覆写差不多，`opeen`和`override`是必不可少的：

```Kotlin
open class Foo {
    open val x: Int get { ... }
}
class Bar1 : Foo() {
    override val x: Int = ...
}
```

使用`val`定义的属性，可以被覆写为`var`，反过来是不行的。

可以在主构造函数对属性进行覆写，像下面的例子，在Foo接口中定义了一个`count`属性，Bar类在主构造函数使用`override`关键字对Foo接口中的count进行了覆写：

```Kotlin
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```

## 覆写规则

Kotlin和Java一样，只允许继承一个类，可以继承多个接口。当一个类有多个继承（多个接口，一个类+一个或多个接口）时，如果出现了同名的方法，那么子类必须自己实现这个方法，像下面的例子，类`C`必须实现`f()`方法，而`a()`，`b()`可以是继承自父类和父接口。调用父类或父接口的语法使用`super`关键字，使用`<>`括起来父类或父接口

```Kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // interface members are 'open' by default
    fun b() { print("b") }
}

class C() : A(), B {
    // The compiler requires f() to be overridden:
    override fun f() {
        super<A>.f() // 指明调用父类A.f()
        super<B>.f() // 或者指明调用父接口B.f()
        // 或者实现自己的逻辑
    }
}
```

这一点是为了避免像C++和Python那么可以继承多个类的时候，发生菱形继承的问题，Kotlin通过上面的规则来消除菱形继承问题，这样就显得非常的明确，而不会产生歧义。

## 抽象类

抽象类是含有抽象方法的类，抽象方法是没有具体实现而只有定义的一个方法，像下面抽象类`Derived`的抽象方法`f()`。如果继承了这个抽象类，必须实现这个抽象方法，或者继续用`abstract`标记这个方法和类为抽象的。这一点和Java是一样的。

```Kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

## 伴生对象（在Scala是这么称呼）

不像Java和C#，Kotlin是没有静态方法的，Kotlin提倡定义包级别的函数（因为函数可以使用`fun`关键字直接定义在包下面）。

另外还可以使用伴生对象，请看伴生对象一节。
