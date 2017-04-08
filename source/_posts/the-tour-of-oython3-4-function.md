---
title: Python3之旅（四） —— 函数定义与调用
date: 2017-04-08 20:49:04
categories: 编程
tags:
- Python
---

Python3中的函数定义和调用。

<!-- more -->

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
## 2 关键字限定参数传递

在3.X中关键字限定参数传递（Keyword-Only Arguments）中，**参数只能通过关键字方式传入，不能通过位置参数方式**。
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

Understanding '*', '*args', '**' and '**kwargs' - Agiliq Blog | Django web app development
http://agiliq.com/blog/2012/06/understanding-args-and-kwargs/
