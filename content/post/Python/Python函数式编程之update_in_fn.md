---
title: "Python函数式编程之 update_in_fn"
date: 2023-05-19T20:53:42+08:00
tags: [SSH, SSH会话, SSH客户端]
categories: [Linux]
---

接上一篇内容 [Python函数式编程之自定义函数get_in](/post/python/python函数式编程之自定义函数get_in) 。这次我们来编写函数式风格的修改字典/列表值的通用函数。

我们先来假设一种情况，你有一个嵌套的动态字典，想要更新里面某个键对应的值，我们用代码来说明

```python
data = {'a': {'b': 1}}
```

现在有一个变量`data` 他的值是一个两层的字典，如果我要更新 `b` 的值到 `2` 常规的做法是这样的

```python
data['a]['b'] = 2
```

如果我有一个更加深的字典

```python
data = {'a': {'b': {'c': {'d': 1}}}}
```

更新值的写法

```python
data['a']['b']['c']['d''] = 2
```

这里有一个情况发生了，当 `data` 是一个动态数组，你不确定中间的某个键是否存在的时候就会有报错

![嵌套字典更新](/img/Python/nested_dict_update.png)

假如这个数据是你的系统里面其他团队开发的一个微服务，报错的信息是`int` 对象是不可读取索引的，这会让人很奇怪，2这个int对象明明是一个值，而不是用来查找索引数据的。

这时候程序的健壮性是比较差的，要想让程序保证健壮，我们可以使用 `if-else` 来判断每个键是否存在，就像这样

![嵌套字典更新](/img/Python/nested_dict_update_if_else.png)

这里的三个`if` 看起来一点都不够优雅，如果其他地方也有同样的写法，只是键的顺序和名字不一样而已呢。能不能封装一个通用的函数来处理类似的情况呢？

# update_in

Python支持面向过程、面向对象、函数式等多范式编程。我们可以使用函数式编程思想来达到这一点。先来看一个简单一点的 `update_in` 函数

```python
from typing import Dict, List, T

def update_in(data: Dict, ks: List, value: T):
    """Updates a value in a nested associative structure.

    Args:
        data (Dict): nested associative structure
        ks (List): sequence of keys
        value (Any): new value
    """
    if len(ks) == 0:
        return
    cursor = data
    for k in ks[:-1]:
        if k not in cursor:
            cursor[k] = {}
        elif not isinstance(cursor[k], dict):
            cursor[k] = {}
        cursor = cursor[k]
    cursor[ks[-1]] = value
```

> 这里有一个类型 `T` ，这是Python内置的任意类型，在 `typing` 模块可以看到定义 `T = TypeVar('T')`
>

## 对update_in函数解释一下

首先判断键序列是否为空，为空的话什么也不做。然后使用一个游标`cursor`来指定当前取得是字典所在的位置，然后不停的循环下去，拿到除最后一个键之外的值。在最后把值赋给找到的最后一个键所指定的值。

## 来看几个例子

来看几个例子，了解函数的作用。假设我有一个空字典 `data = dict()`

```python
update_in(data, ['a'], 1)
```

执行完成`update_in`函数后，data的值为 `{'a': 1}`

```python
update_in(data, ['a', 'b'], 2)
```

执行完后，data的值为 `{'a': {'b': 1}}`

```python
update_in(data, ['a', 'b', 'c', 'd'], 3)
```

执行完后，data的值为 `{'a': {'b': {'c': {'d': 3}}}}`

我们封装了一个通用函数，可以对任意的字典，任意的键序列进行值的修改。

再更进一步，如果不是单纯的给某个键序列进行赋值，而是要取出那个值，然后再进行一些变换呢？比如，我要取出 `['a', 'b']` 对应的值，进行一个平方计算，或者取个对数呢？我们增加一个参数，变成`update_in_fn`函数

# update_in_fn

```python
from typing import Dict, List, Callable, T

def update_in_fn(data: Dict, ks: List, fn: Callable, default: T):
    """Updates a value in a nested associative structure.

    Args:
        data (Dict): nested associative structure
        ks (List): sequence of keys
        fn (Callable): take the old value and return the new value
        default (Any): default value

    Examples:
        >>> a = {}
        >>> update_in_fn(a, ['a', 'b'], lambda x: x + 1, default=0)
        >>> assert a == {'a': {'b': 0}}
    """
    if len(ks) == 0:
        return
    cursor = data
    for k in ks[:-1]:
        if k not in cursor:
            cursor[k] = {}
        elif not isinstance(cursor[k], dict):
            cursor[k] = {}
        cursor = cursor[k]
    key = ks[-1]
    if key not in cursor:
        cursor[key] = default
    else:
        cursor[key] = fn(cursor[key])
```

我们对这个函数加以说明：

第一个参数是被修改的字典数据

第二个参数是键序列

第三个参数是一个可调用函数，传入一个参数，返回一个结果

第三个参数表明如果字典里面没有找到对应的值，使用default填充键序列指定的那个位置的值

还是使用几个例子来说明用法及结果

```python
data = dict()
update_in_fn(data, ['a'], lambda x: x**2, default=2)
# data: {'a': 2}
```

在一个空字典上执行函数调用时，会填充默认的值 `default`

```python
data = {'a': 3}
update_in_fn(data, ['a'], lambda x: x**2, default=0)
# data: {'a': 9}
```

对键 `'a'` 指定的值进行平方计算，计算后的结果为 `9`

```python
data = {'a': {'b': 4}}
update_in_fn(data, ['a', 'b'], lambda x: x**2, default=0)
# data: {'a': {'b': 16}}
```

对嵌套字典里面的值进行更新，4的平方16

# 结论

借助Python强大的函数式编程范式，我们可以编写非常通用的函数及模块，上面的两个函数代码还有优化的空间，比如用户自定义的字典类型，可以像get_in函数一样支持数组。