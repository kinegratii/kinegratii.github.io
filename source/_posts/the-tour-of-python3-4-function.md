---
title: Python3之旅（四） —— 函数定义与调用
date: 2017-04-08 20:49:04
categories: 编程
tags:
- Python
---

## 1 位置参数和关键字参数

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

<!-- more -->

## 2 强制关键字参数定义和调用

> PEP链接
[PEP 3102 -- Keyword-Only Arguments | Python.org](https://www.python.org/dev/peps/pep-3102/)

在3.X中新增了强制关键字参数传递（Keyword-Only Arguments）中，**参数只能通过关键字方式传入，不能通过位置参数方式**。

定义的形式为使用 `*` 单独占用一个参数，表示之后的参数必须以关键字方式传入。

下表通过对比显示了一个简单的例子。

| 定义-调用 | `def compare(a, b, key=None):pass` |  `def compare(a, b, *, key=None):pass` |
| ------ | ------ | ------ |
| `compare(2, 3)` | 是 | 是 |
| `compare(2, 3, key=True)` | 是 | 是 |
| `compare(a=2, b=3, key=True)` | 是 | 是 |
| `compare(1, 2, ** {'key': True})` | 是 | 是 |
| `compare(2, 3, True)` | 是 | 引发TypeError |
| `compare(1, 2, *[True])` | 是 | 引发TypeError |

## 3 参考链接

- [Understanding '*', '*args', '**' and '**kwargs' - Agiliq Blog | Django web app development](http://agiliq.com/blog/2012/06/understanding-args-and-kwargs/)
- [Python 的 Keyword-Only Arguments (强制关键字参数) - Python 学习之旅 - SegmentFault](https://segmentfault.com/a/1190000005173136)
