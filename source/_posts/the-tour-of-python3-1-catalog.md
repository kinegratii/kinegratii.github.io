---
title: Python3之旅（一） —— 目录
date: 2017-02-04 09:55:27
categories: 编程
tags:
 - Python
comments: false
---

《Python3之旅》介绍了Python3的相关知识点及其应用。

<!-- more -->

## 1 概述

### 1.1 Python3介绍

- python学习术语
- Python3优点
- 解释 Python 2 和 Python 3 的版本之间差别 - Python - 伯乐在线
http://python.jobbole.com/87372/
- Python 2.7 Countdown
https://pythonclock.org/

### 1.2 参考资料

本书的内容由众多资源（问答、博客、书籍等）整理而成，以下是主要的参考资料：

- [PEP 增强方案 Python Enhancement Proposals](https://www.python.org/dev/peps/)
- [Dive Into Python 3](http://www.diveintopython3.net/)
- [Python部落](https://python.freelycode.com/)
- [优雅简单的Python] - 专题 - 看云
http://www.kancloud.cn/special/python
- Incompatibilities between Python 2 and Python 3 - Python Language - Stack Overflow
http://stackoverflow.com/documentation/python/809/incompatibilities-between-python-2-and-python-3

## 2 Python3的新语法

本章描述Python3.X的语法变化，整理自[What’s New In Python 3.0 — Python v3.0.1 documentation](https://docs.python.org/3.0/whatsnew/3.0.html)，并按照语法进行分类阐述。

### 2.1 数据类型

- Python3.X不再有long的数据类型，合并到int中，因此像 `34023932232L`和 `34023932232l` 将引发语法错误。
- `sys.maxint`常量被移除，因为整数没有任何限制。然而常量 `sys.maxsize`在列表索引等情况下可以被认为正无穷。
- 八进制数使用更加明确的0o前缀，代码`t = 070`将产生语法错误，`int('070')`结果为70。

### 2.2 列表、字典和集合

#### 2.2.1 迭代器和列表

一些常用的API不再返回列表

- 字典中  `dict.keys()`, `dict.items()` 和 `dict.values()` 等方法返回 “views” 。代码 `k = d.keys(); k.sort()`在3.X无法正常工作。应该使用内置的`sorted`函数 `k = sorted(d)`。
- `dict.iterkeys()`, `dict.iteritems()` 和 `dict.itervalues()`等方法不再支持。
- 内置函数 `map()` 和 `filter()` 返回迭代器，使用 `list(map())` 或者列表推导式返回数据列表。
- 3.X的`range` 函数同2.X的`xrange`一样，返回的是一个迭代器， `xrange` 在3.X被移除
- `zip`函数返回一个迭代器

### 2.3 运算符

#### 2.3.1 除法

| 操作符 | Python2 | Python3 | 备注 |
| ------ | ------ | ------ | |
| /  | 地板除法 | 精确除法 | |
| // | 地板除法 | 地板除法 | |
| /  | 5 / 2 = 2 | 5 / 2 = 2.5 | |
| | 5.0 / 2 = 2.5 | 5.0 / 2 = 2.5 | |
| | 4 / 2 = 2 | 4 / 2 = 2.0 | |
| | 4.0 / 2 = 2.0 | 4.0 / 2 = 2.0 | |
| // | 5 // 2 = 2 | 5 // 2 = 2 | |
| | 5.0 // 2 = 2.0 | 5.0 // 2 = 2.0 | |
| | 4 // 2 = 2 | 4 // 2 = 2 | |
| | 4.0 // 2 = 2.0 | 4.0 // 2 = 2.0 | |

**应用实践**

- Python3中/执行的是精确除法，无论操作数的类型，都会返回一个浮点数。

#### 2.3.2 比较

3.0 简化了比较规则：

- 当两个操作数类型不同，比较操作符（`> >= <= <`）将引发一个 `TypeError` 异常。因此像`1 < ''`、`0 > None` 、`len < len`、`None < None` 不再有效。同样的道理，不同类型组成的列表也不具有比较的意义。
- 当两个操作数属于不同类型时，`==`和`!=`操作符总是返回False和True
- `builtin.sort()`和`list.sorted()`不再支持cmp参数，应该使用key参数，注意的的是key和reverse参数必须以关键字方式传入。
- Python2中的 `builtin.cmp()`和类中的魔术方法`__cmp__`不推荐使用

### 2.4 语句

#### 2.4.1 print函数

在Python3.0中使用 `print()`函数代替`print`语句，并且提供一些关键字参数实现`print`语句的特别的语法。

```
Old: print "The answer is", 2*2
New: print("The answer is", 2*2)

Old: print x,           # Trailing comma suppresses newline
New: print(x, end=" ")  # Appends a space instead of a newline

Old: print              # Prints a newline
New: print()            # You must call the function!

Old: print >>sys.stderr, "fatal error"
New: print("fatal error", file=sys.stderr)

Old: print (x, y)       # prints repr((x, y))
New: print((x, y))      # Not the same as print(x, y)!
```
函数的完整定义为：`print(*objects, sep='', end='\n', file=sys.stdout)` ，关键字参数描述如下：
- sep：分隔符
- end：末尾字符串
- file：输出目标，如控制台、文件。

比如：
```
>>> print("There are <", 2**32, "> possibilities!", sep="")
There are <4294967296> possibilities!
```
另外注意的是，`print()`函数不支持“软空格”，示例如下：
```
# In Python 2.X
>>> print "A\n", "B"
A
B
# In Python 3.X
>>> print("A\n",  "B")
A
 B
```

#### 2.4.2 赋值

扩展迭代解包（Extended Iterable Unpacking），可以使用 `a, b, *reset = some_sequece`和 `*reset, a = some_sequece`的语法，执行后变量reset是一个列表，可以为空列表（[]）。

```
>>>a, *reset, b = range(5)
>>>a
0
>>>reset
[1, 2, 3]
>>>b
4
```

#### 2.4.3 闭包与nonlocal语句

nonlocal语句提供了一种解决嵌套函数引用外部变量的方法，在Python2.x中，闭包只能读取外部函数的变量，而不能改写它。

举例来说，这样是合法的。

```
def a():
  x = 0
  def b():
    print locals()
    y = x + 1
    print locals()
    print x, y
  return b

a()()
```
参考资料

- Python的闭包与nonlocal
http://www.keakon.net/2009/10/15/Python%e7%9a%84%e9%97%ad%e5%8c%85%e4%b8%8enonlocal
- Python中nonlocal关键字 - Python基础教程|Python教程|Python入门 - PythonTab中文网
http://www.pythontab.com/html/2013/pythonjichu_0407/338.html

#### 2.4.4 异常语句

异常语句变更，由 `except exc,var`变为 `except exc as var`，捕获多个异常时使用 `except (exc1, exc2) as var`。

### 2.5 函数定义与调用

#### 2.5.1 位置参数和关键字参数

`*args`和`**kwargs`分别表示位置参数和关键字参数，可用于函数定义、函数调用和变量赋值。使用`*args`和`**kwargs`可以加强函数的拓展性，方便日后的代码维护。

在使用过程中，需要注意的是位置参数必须在关键字参数之前。

以下是函数定义和函数调用的几个例子。

```python
# 使用*args以适应任意数目的数字求和
def my_sum(*args):
    s = 0
    for i in args:
        s += i
    return  s

my_sum(1, 2)  # 3
my_sum(1, 2, 3, 4) # 10
```
#### 2.5.2 关键字限定参数传递

在3.X中关键字限定参数传递（Keyword-Only Arguments）中，`**` 参数只能通过关键字方式传入，不能通过位置参数方式**。
```python
def sortwords(*wordlist, case_sensitive=False):
    pass
```
Python2.X的语法情况，如函数 `compare` 携带两个位置参数和一个关键字参数。
```python
def compare(a, b, key=None):
    pass

# 几种有效的调用形式
compare(2, 3)
compare(2, 3, True)
compare(2, 3, key=True)
compare(a=2, b=3, key=True)
compare(1, 2, *[True])
compare(1, 2, ** {'key': True})
```
在key参数前增加一个`*`，表示key参数必须以关键字形式传入。
```
def compare(a, b, *, key=None):
    pass

# 几种有效调用形式
compare(2, 3)
compare(2, 3, key=True)
compare(a=2, b=3, key=True)
compare(1, 2, ** {'key': True})

# 以下两种调用形式均引发 TypeError: compare() takes 2 positional arguments but 3 were given
compare(2, 3, True)
compare(1, 2, *[True])
```

链接 http://agiliq.com/blog/2012/06/understanding-args-and-kwargs/


### 2.6 内置空间变更

- `print`由语句变为内置函数，最明显的区别是3.x需要添加括号
- `reduce` 函数由内置命名空间移到 `functools.reduce`
- `xrange` 在3.X被移除， `range` 返回的是一个迭代器，如需具体数据使用`list(range(start, end, step=1))` 即可

### 2.7 类

#### 2.7.1 经典类与新式类

从2.2开始引入新式类。与经典类的区别：

- Python 2.X默认都是经典类，只有显式继承object的类才是新式类，而在Python3.X中默认都是新式类，无需显式继承object。
- 在多继承中，新式类采用广度优先搜索，而旧式类是采用深度优先搜索。
- 新式类对象可以直接通过__class__属性获取自身类型:type
- 新式类增加了__slots__内置属性, 可以把实例属性的种类锁定到__slots__规定的范围之中
- 新式类增加了__getattribute__方法

参考
- Python新式类和经典类的区别 - u010066807的博客 - 博客频道 - CSDN.NET
http://blog.csdn.net/u010066807/article/details/46896835
- https://wiki.python.org/moin/NewClassVsClassicClass

#### 2.7.2 类继承
#### 2.7.3 元类

元类的定义方式有所变更，由
```
class C:
    __metaclass__ = M
```
变为
```
class C(metaclass=M):
    pass
```

- Python 3 Object Oriented
https://www.tutorialspoint.com/python3/python_classes_objects.htm

### 2.8 模块整合

- 重命名：一些以大写字母表示的模块更为小写字母，比如`Tkinter/tkinter`、`Queue/queue`、`ConfigParse/configparser`等。
- 整合：如`urllib/urllib2`等HTTP请求库。


## 3 字符串

### 3.1 字节、字符和编码

由unicode字符组成的序列称为string，由0-255数字组成的序列称为bytes对象。string通过encode变成bytes，bytes通过decode变成string。


- Unicode HOWTO — Python 3.6.0 documentation
https://docs.python.org/3/howto/unicode.html
- 图灵社区 : 阅读 : Python 3的bytes/str之别
http://www.ituring.com.cn/article/1116

http://www.diveintopython3.net/strings.html#byte-arrays

### 3.2 unicode与文件读写
### 3.3 串口编程

## 4 模块

### 4.1 enum：枚举类型
自Python3.4开始，标准库中新增了枚举类型模块enum，在此之前可以通过第三方库引用。模块中有四种枚举类型。

- `enum.Enum`：基本类。
- `enum.IntEnum`：继承自Enum，但枚举项的值为整数。
- `enum.Flag`：Python3.6新增。
- `enum.IntFlag`：Python3.6新增。

简单的枚举类型定义如下：

```python
from enum import Enum

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

print(Color.RED.name)    # 'RED'
print(Color.RED.value)    # 1
print(type(Color.RED))     # <enum 'Color'>
print(isinstance(Color.RED, Color)) # True

for color in Color:
    print(color)

hex_dict = { Color.RED:'#FF0000', Color.GREEN: '#0000FF', Color.BLUE:'#00FF00' }
```

> 即使使用了`class`语法创建枚举类型，但是其和一般Python类还是有所区别，更多信息可查看[How are Enums different](https://docs.python.org/3/library/enum.html#how-are-enums-different) 。

`RED = 1` 定义了一个枚举项，使用`Color.RED`引用，具有以下几个特点：

- `Color.RED` 属于Color类
- 具有名称和值两个属性，值可以是任意类型的对象。分别使用`Color.RED.name`和 `Color.RED.value` 获取。
- 在Python3.6+，除了字面量外，枚举项的值还可以使用`enum.auto()`表示自动排序的整数值，第一个auto()表示1，第二个auto()表示2，依次类推，通常用于枚举项的值经常变化的情况。
- 枚举项是可哈希的，可以作为字典的键。

通过值访问枚举项：
```python
Color(1)  # <Color.RED: 1>
```
通过名称访问枚举项
```python
Color['RED']  # <Color.RED: 1>
```

### 4.2 StringIO
### 4.3 Mock测试框架

《Python高手之路》

## 5 异步并发编程

### 5.1 asyncio模块
### 5.2 aiohttp库
### 5.3 tkinter与异步

asyncio示例

Python asyncio Examples
http://www.programcreek.com/python/index/5526/asyncio

`asyncio.ensure_future`的使用
http://stackoverflow.com/questions/36342899/asyncio-ensure-future-vs-baseeventloop-create-task-vs-simple-coroutine


使用 `asyncio.ensure_future`和`await`的区别

```python
import asyncio

async def msg(text):
    await asyncio.sleep(0.1)
    print(text)

async def long_operation():
    print('long_operation started')
    await asyncio.sleep(3)
    print('long_operation finished')

async def test1():
    await msg('first')
    task = asyncio.ensure_future(long_operation())
    await msg('second')
    await task

async def test2():
    await msg('first')
    await long_operation()
    await  msg('second')


if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(test1())
    # loop.run_until_complete(test2())
```

输出

```
test1输出：

first
long_operation started
second
(3s)
long_operation finished

test2输出：

first
long_operation started
(3s)
long_operation finished
second
```

## 附录
### A 兼容库
