---
title: "Python变量可变性"
date: 2022-11-13T18:13:19+08:00
tags: [Python]
---

Python变量的可变与不可变

****Mutable and Immutable****

可变是一种奇特的说法，即对象的内部状态已更改/突变。 所以，最简单的定义是：内部状态可以改变的对象是可变的。 另一方面，不可变对象一旦创建就不允许对其进行任何更改。这两种状态都是 Python 数据结构的组成部分。

## 可变定义

可变是指某些东西是可变的或有能力改变的。 在 Python 中“可变”是对象改变其值的能力。这些通常是存储数据集合的对象。

## 不可变定义

不可变是指随着时间的推移不可能发生变化。 在 Python 中，如果一个对象的值不能随时间改变，那么它就是不可变的。一旦创建，这些对象的值是永久固定的，直到被回收。

## 可变和不可变对象列表

内置可变类型：

- 列表（List）
- 集合（Set）
- 字典（Dictionarie）
- 自定义类（User-Defined Classes 取决于用户的定义）

内置不可变类型：

- 数字 （Integer, Rational, Float, Decimal, Complex & Booleans）
- 字符串（String）
- 元组（Tuple）
- 冻结的集合（Frozen Set）
- 自定义类（User-Defined Classes 取决于用户的定义）

对象可变性是使 Python 成为动态类型语言的特征之一。尽管 Python 中的 Mutable 和 Immutable 是一个非常基本的概念，但由于不变性的不传递性，它有时会让人有些困惑。

## Python 中的对象

在 Python 中，一切都被视为对象。 每个对象都具有以下三个属性：

Identity — 这是指对象在计算机内存中引用的地址，使用[id](https://docs.python.org/3/library/functions.html?highlight=id#id)函数可以得到唯一标识。
Type — 这是指创建的对象的类型。 例如：整数、列表、字符串等。
Value — 这是指对象存储的值。 例如：List=[1,2,3] 将保存数字 1,2 和 3
虽然 ID 和 Type 一经创建就无法更改，但 Mutable 对象的值可以更改。

### Python中的可变对象

与其深入研究 Python 中可变和不可变的理论方面，不如用简单的代码来描述它在 Python 中的含义。

让我们逐步讨论以下代码：

```python
# 创建一个包含城市名称的列表
cities = ['Shanghai', 'Changsha', 'Guangzhou']

for city in cities:
    print(city, end=', ')

# 输出: Shanghai, Changsha, Guangzhou,
```

以十六进制格式打印内存地址中创建的对象的位置

```python
print(hex(id(cities)))

# 输出: 0x185bd166100
# 输出内容会根据不同的机器，不同的运行环境，结果也不相同
```

添加一个新的城市到列表当中

```python
cities.append('Shenzheng')
for city in cities:
    print(city, end=', ')

print(hex(id(cities)))

# 输出: Changsha, Guangzhou, Shenzheng,  0x185bd166100
```

可以看到，最后输出的十六进制的地址是没有改变的。

上面的例子告诉我们，我们可以通过再添加一个城市”Shenzheng”来改变对象“cities”的内部状态，但是对象的内存地址并没有改变。这证实了我们没有创建一个新对象，而是同一个对象被更改或变异了。因此，我们可以说具有引用变量名称“城市”的列表类型的对象是 Mutable object。

现在让我们讨论 Immutable一词。考虑到我们理解 mutable 的含义，很明显 immutable 的定义将包含“不”。这是不可变的最简单定义 —— 内部状态不能改变的对象是不可变的。

### Python中的不可变对象

同样的，一个简单的代码将是描述不可变代表什么的最佳方式。 因此，让我们逐步讨论以下代码：

```python
# 创建一个包含每周工作日名称的元组
weekdays = 'Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'

## 打印工作日名称元组
print(weekdays)

# 输出: ('Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday')

# 打印十六进制格式内存地址

print(hex(id(weekdays)))

# 输出: 0x241fa15fa60
```

元组是不可变的，因此不能添加新元素。

使用带有 `+=` 运算符的元组合并在元组“weekdays”中添加新的元素

```python
weekdays += ('Pythonday',)

print(weekdays)

print(hex(id(weekdays)))

# 输出：
# ('Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Pythonday')
# 0x22d51ba9a50
```

上面的例子表明我们能够使用相同的变量名来引用一个对象，该对象是一种包含七个元素的元组。 但是，新旧元组的 ID 或内存位置并不相同。 我们无法更改对象“weekdays”的内部状态。Python 程序管理器在内存地址中创建了一个新对象，变量名称“weekdays”开始引用包含八个元素的新对象。 因此，我们可以说具有引用变量名称“weekdays”的元组类型的对象是 Immutable object。

那么如何使用可变和不可变对象：

可变对象可用于您希望允许任何更新的地方。 例如，您有一个组织中的员工姓名列表，每次雇用新成员时都需要更新该列表。 您可以创建一个可变列表，并且可以轻松更新它。

不变性为我们在允许并行处理的以网络为中心的环境中执行的不同敏感任务提供了许多有用的应用程序。 通过创建不可变对象，您可以密封值并确保没有线程可以调用对数据的覆盖/更新。 这在您想要编写一段无法修改的代码的情况下也很有用。 例如，尝试查找不可变对象的值的调试代码。

注意不变性的非传递性：

好的！ 现在我们确实了解了 Python 中的可变对象和不可变对象是什么。 让我们继续讨论这两者的结合并探索可能性。 让我们讨论一下，如果您有一个包含可变对象的不可变对象，它将如何表现？ 或相反亦然？ 让我们再次使用代码来理解这种行为

创建一个元组（不可变对象），包含两个列表（可变对象）：

```python
# 列表表示名字，年龄和性别
persons = (['Aron', 5, 'Male'], ['Anusha', 8, 'Female'])
# 打印元组
print(persons)
# 打印十六进制内存地址
print(hex(id(persons)))

# 输出：
# (['Aron', 5, 'Male'], ['Anusha', 8, 'Female'])
# 0x16eb651cb40
```

更改第一个元素的年龄，使用索引 `[0]` 选择元组的第一个元素，然后使用索引 `[1]` 选择列表的第二个元素并将年龄的值赋值为4

```python
persons[0][1] = 4
# 打印更新后的元组
print(persons)
# 打印十六进制内存地址
print(hex(id(persons)))

# 输出：
# (['Aron', 4, 'Male'], ['Anusha', 8, 'Female'])
# 0x16eb651cb40
```

在上面的代码中，您可以看到对象“persons”是不可变的，内存地址没有改变，因为它是一种元组。但是，它有两个列表作为元素，我们可以更改列表的状态（列表是可变的）。所以，这里我们并没有改变 Tuple 内部的对象引用，而是被引用的对象发生了变异。

同样地，让我们探讨一下如果你有一个包含不可变对象的可变对象，它将如何表现？ 让我们再次使用代码来理解行为

```python
# 创建一个包含两个元组的列表对象
list1 = [(1, 2, 3), (4, 5, 6)]
# 打印
print(list1)
# 打印内存地址
print(hex(id(list1)), hex(id(list1[0])), hex(id(list1[1])))

# 输出：
# [(1, 2, 3), (4, 5, 6)]
# 0x1b355326080 0x1b35539b900 0x1b3555fc1c0
```

列表的元素是可以修改，但是列表里面的元组元素修改是不被允许的。

```python
# 修改列表的第一个元素值
list1[0] = (7, 8, 9)
# 打印列表
print(list1)
# 打印内存地址
print(hex(id(list1)), hex(id(list1[0])), hex(id(list1[1])))

# 输出：
# [(7, 8, 9), (4, 5, 6)]
# 0x1b355326080 0x1b3553261c0 0x1b3555fc1c0
```

```python
# 把列表的第二个元素，也就是第二个元组拼接上一个新的值
list1[1] += (3,)
# 打印列表
print(list1)
# 打印内存地址
print(hex(id(list1)), hex(id(list1[0])), hex(id(list1[1])))

# 输出：
# [(7, 8, 9), (4, 5, 6, 3)]
# 0x1b355326080 0x1b3553261c0 0x1b355513540
```

从上面两段程序的结果来看，list1的内存地址是没有改变的，改变列表里面的整个元素的值，也不会改变那个元素的内存地址，啥时内存里面的内容发生了改变。而对一个不可变元素进行扩展的时候，内存地址是发生了改变的。

**元组的不变性**

元组是不可变的，因此一旦在 Python 中创建，它们就不能有任何更改。 这是因为它们支持与字符串相同的序列操作。 我们都知道字符串是不可变的。 索引运算符将从元组中选择一个元素，就像在字符串中一样。 因此，它们是不可变的。
