---
title: Scala学习笔记之正则表达式
date: 2016-08-23 15:27:00
tags: [Scala, Scala正则表达式]
---

QQ: 380800878, 微信: kittenll

如果还不熟悉正则表达式的可以先查看[正则表达式30分钟入门教程](http://deerchao.net/tutorials/regex/regex.htm)，非常经典的入门教程。

Scala提供的正则表达式功能是通过类`Regex`来实现，函数签名为：

```scala
Regex(regex: String, groupNames: String*)
```

第一个参数为正则表达式的字符串，后面是可变参数列表，是给regex里面的分组起一个名字。Scala的正则表达式的功能非常强大，这要得益于Scala本身特别的设计，尤其是pattern match，是其他编程语言很难超越的。

先来看第一个例子，匹配小写英文字母（如下代码中单独一行的=>表示接下来是结果）:

```scala
import scala.util.matching.Regex
val str = "abc"
val regex = new Regex("[a-z]+")
println(regex.findAllIn(str).toList)

=>
List(abc)
```

可以结果看到是一个元素的的List，匹配到字符串abc。如果要匹配到大写和小写，正则表达式需要修改成`[a-zA-Z]`，还有另外一个办法就是为正则表达式添加选项，让`[a-z]`可以直接匹配到大写和小写字母，正则表达式就需要写成`(?i)[a-z]+`

```scala
import scala.util.matching.Regex
val str = "abcEFGhijk"
val regex = new Regex("(?i)[a-z]+")
println(regex.findAllIn(str).toList)

=>
List(abcEFGhijk)
```

所以在Scala添加选项的正确姿势就是在正则表达式的前面用使用括号问号+选项字母，这与PHP的正则完全不同，PHP使用如下的方式，使用相同的字符来做开头和结尾，这里就是`/`，其实在PHP里面使用`%`, `|`这些字符也是完全可以的，然后在后面的分割符后面加上选项字母

```
/[a-z]+/i
```

匹配简单的电话号码：

```scala
import scala.util.matching.Regex
val str = "13907461234"
val regex = new Regex("""^1[358]\d{9}$""")
println(regex.findAllIn(str).toList)

=>
List(13907461234)
```

这里使用了`"""`raw string，三个双引号括起来的内容不需要做转义，正则表达式本身就比较难看，如果再加上转义就更难于阅读和理解。

匹配简单的电子邮件：

```scala
val str = "email: abc@xyz.com"
val regex = new Regex("""(?i)[a-z0-9._-]+@[a-z0-9._-]+(\.[a-z0-9._-]+)+""")
println(regex.findAllIn(str).toList)

=>
List(abc@xyz.com)
```

Scala的正则表达式还有一个非常有用的功能，那就是模式匹配（pattern match），不仅可以查找所有匹配的的字符串，还可以直接提取分组的内容，请看下面api手册的例子：

```scala
val date = """(\d\d\d\d)-(\d\d)-(\d\d)""".r //可以直接使用.r生成一个Regex对象
"2004-01-20" match {
  case date(year, month, day) => s"$year was a good year for PLs."
}

=>
2004 was a good year for PLs.
```

可以看到上面的例子直接把年份的分组提取了出来，month, day同样也可以提取出来。

```scala
import scala.util.matching.Regex
"2004-01-20" match {
  case date(_*) => "It's a date!"
}

=>
It's a data!
```

可以忽略匹配到的参数。

上面仅当字符串为XXXX-XX-XX的时候才可以使用，如果一个字符串不仅仅是只是一个年份，模式匹配是否也可以提取呢？答案是Yes，使用unanchored就可以做到，看下面的例子：

```scala
val date = """(\d\d\d\d)-(\d\d)-(\d\d)""".r
val embeddedDate = date.unanchored
"Date: 2004-01-20 17:25:18 GMT (10 years, 28 weeks, 5 days, 17 hours and 51 minutes ago)" match {
  case embeddedDate("2004", "01", "20") => "A Scala is born."
}

=>
A Scala is born.
```

给正则表达式的分组命名：

```scala
import scala.util.matching.Regex
val str = "2016-08-23"
val date = new Regex("""(\d\d\d\d)-(\d\d)-(\d\d)""", "year", "month", "day")
val matches = date.findAllIn(str)
println("year: " + matches.group("year"))

=>
year: 2016
```

Scala的正则表达式默认使用单行模式去匹配文本内容

```scala
import scala.util.matching.Regex
val str = "abc\nefg"
val regex = new Regex("^[a-z].*")
println(regex.findAllIn(str).toList)

=>
List(abc)
```

只是匹配到了`abc`，把正则表达式加上`s`选项那么`.`就可以匹配到换行符。结果就`abc\nefg`可以全部匹配到。如果想以多选的方式匹配一段文本内容，那么就需要使用`m`选项：

```scala
import scala.util.matching.Regex
val str = "abc\nefg"
val regex = new Regex("(?m)^[a-z].*")
println(regex.findAllIn(str).toList)

=>
List(abc, efg)
```

测试一个字符串是否匹配一个模式：

```
val str = "abc123"
val pattern = "[a-z]{3}[0-9]{3}"
println(str.matches(pattern))

=>
true
```
