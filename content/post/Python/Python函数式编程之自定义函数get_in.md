---
title: "Python函数式编程之自定义函数get_in"
date: 2023-05-02T14:52:25+08:00
tags: ["Python", "函数式编程", "get in"]
categories: [编程语言]
keywords: ["Python", "函数式编程"]
---

## Python函数式编程之 get_in

编程语言支持通过以下几种方式来解构具体问题：

- 大多数的编程语言都是 **过程式** 的，所谓程序就是一连串告诉计算机怎样处理程序输入的指令。C、Pascal 甚至 Unix shells 都是过程式语言。
- 在 **声明式** 语言中，你编写一个用来描述待解决问题的说明，并且这个语言的具体实现会指明怎样高效的进行计算。 SQL 可能是你最熟悉的声明式语言了。 一个 SQL 查询语句描述了你想要检索的数据集，并且 SQL 引擎会决定是扫描整张表还是使用索引，应该先执行哪些子句等等。
- **面向对象** 程序会操作一组对象。 对象拥有内部状态，并能够以某种方式支持请求和修改这个内部状态的方法。Smalltalk 和 Java 都是面向对象的语言。 C++ 和 Python 支持面向对象编程，但并不强制使用面向对象特性。
- **函数式** 编程则将一个问题分解成一系列函数。 理想情况下，函数只接受输入并输出结果，对一个给定的输入也不会有影响输出的内部状态。 著名的函数式语言有 ML 家族（Standard ML，Ocaml 以及其他变种）和 Haskell。

        一些语言的设计者选择强调一种特定的编程方式。 这通常会让以不同的方式来编写程序变得困难。其他多范式语言则支持几种不同的编程方式。Lisp，C++ 和 Python 都是多范式语言；使用这些语言，你可以编写主要为过程式，面向对象或者函数式的程序和函数库。在大型程序中，不同的部分可能会采用不同的方式编写；比如 GUI 可能是面向对象的而处理逻辑则是过程式或者函数式。
        在函数式程序里，输入会流经一系列函数。每个函数接受输入并输出结果。函数式风格反对使用带有副作用的函数，这些副作用会修改内部状态，或者引起一些无法体现在函数的返回值中的变化。完全不产生副作用的函数被称作“纯函数”。消除副作用意味着不能使用随程序运行而更新的数据结构；每个函数的输出必须只依赖于输入。

---

## 真实案例

上面是一些基本的理论，在实际的工作当中，经常会看到从一个多层嵌套的字典当中取一个值，会使用连续的下标，但是如果中间某个值并非一个字典，就会导致程序的运行时错误。而这种错误经常得等到代码上线之后才会被发现，开发的时候经常都是测试的正常数据。

例如：

```python
# 查询Elasticsearch，获得检索结果
resp = es_request("_search", dsl)
# 从检索结果当中取出结果总数
total = resp["hits"]["total"]["value"]
```

上面的代码通常不会有问题，因为Elasticsearch的检索结果总是会包含 hits/total/value 这个值，如果没有结果，这个值就为0。但是，看看下面的动态取值：

```python
value = resp["data"]["aggs"][aggs_key]["value"]
```

这是在封装了Elasticsearch查询之后的结果，假设是一个叫search_service的微服务，如果aggs下的值为空，那么再取动态的 aggs_key 的值就会报 Key Error 的错误。

这是在我们产品当中真实发现的情况。下面介绍一个我自己写的，对字典、列表、嵌套字典与列表的取值的通用方法 `get_in` ，名字来源于 `Clojure` 的一个内置方法 `[get-in](https://clojuredocs.org/clojure.core/get-in)` 。

如果是取一个字典的值，同时又不想得到Key Error的错误，Python提供的一个便捷的方法：

```python
# 定义一个字典对象
d = {'a': 1, 'b': 2}
# 从字典当中取出键为 a 的值，如果键 a 不存在，返回 default value
v = d.get('a', 'default value')
```

字典对象在Python当中有一个get方法，还可以传一个默认值，这样就不会报错了。来看看这个方法的定义与文档

```python
get(key[, default])
Return the value for key if key is in the dictionary, else default. If default is not given, it defaults to None, so that this method never raises a KeyError.
```

### get_in函数

下面正式介绍我们的get_in函数，看来看函数定义：

```python
from typing import List, Union, Dict

def get_in(data: Union[Dict, List], ks: List, default={}):
    """根据ks指定的键或者索引从字典或者列表当中取值.

    Args:
        data (dict|list): 字典或者数组，或者两者的嵌套
        ks (list): 取值路径
        default (Any): 当没有ks对应的路径是的默认值

    Examples:
        >>> get_in({'a': 1}, ['a'])
        1

        >>> get_in({'a': {'b': 2}}, ['a', 'b'])
        2

        >>> get_in({'a': [1, 2, 3]}, ['a', 1])
        2

        >>> get_in({'a': {'b': 2}}, ['a', 'c'], default=9)
        9

    Returns:
        Any: 数组或者字典的值
    """
    cursor = data
    for k in ks:
        if isinstance(k, int) and isinstance(cursor, list):
            if len(cursor) == 0:
                return default
            cursor = cursor[k]
        elif isinstance(cursor, dict):
            cursor = cursor.get(k, None)
        else:
            return default
        if cursor is None:
            return default
    return cursor
```

咱们对这个函数做一个解释

第一行的导入语句，从 `typing` 模块导入类型定义。

> 虽然Python是强类型的动态语言，但是Python的类型系统是隐式的，在Python 3.7及之后的版本，对类型系统有了比较完善的支持。在定义变量，形式参数的时候都是可以指定变量和参数的类型，这对理解代码是非常有帮助的。
>

函数定义 `get_in(data: Union[Dict, List], ks: List, default={})` ，通过形式参数的类型定义可以知道，第一个参数是一个字典或者列表类型的数据，就是我们希望从中获得数据的对象；第二个参数是一个列表，希望传入一个层级的键的一个列表；第三个参数是默认值，如果data当中取不到层级的ks对应的值，则返回default默认值

通过几个例子来熟悉 `get_in` 函数的用法

例一

```python
data = {'a': 1, 'b': 2}
ks = ['a']
v = get_in(data, ks, default=-1)
```

这个比较简单，对一个简单的字典取值，默认返回-1

例二

```python
data = [1, 2, 3, 4, 5]
ks = [1]
v = get_in(data, ks, default=-1)
```

对一个列表取下标为 `1` 的值，默认返回-1

例三

```python
data = {'a': [1, 2, 3], 'b': {'c': 9}}
ks = ['a', 1]
v = get_in(data, ks, default=-1)
```

这个数据稍微复杂一点了，字典data的健a对应的是一个列表了。ks表示取键a对应的值下面列表的下标为 `1` 的值，默认返回-1。最后 `v` 的结果是 `2`

例四

```python
data = {'a': {'b': {'c': 9}}}
ks = ['a', 'b', 'c']
v = get_in(data, ks, default=-1)
```

这样就可以取到 9 这个值了

再看一下取不到值的情况

```python
data = {'a': {'b': {'c': 9}}}
ks = ['a', 'z', 'c']
v = get_in(data, ks, default=-1)
```

`v` 的值为 `-1` ，同时还不会有 Key Error的错误

`get_in` 函数还可以处理非常复杂，嵌套得非常深的字典与列表的数据。还可以对里面的判断做一个修改，添加自定义类型的支持，对这个函数进行扩展。

## 结论

Python是一门多范式的编程语言，通过灵活的语言特性可以得到非常灵活、优雅、健壮的代码。看似一下简单的函数，可以通用得处理非常复杂的数据结构。
