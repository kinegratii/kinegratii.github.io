---
title:  Python3之旅（三）—— 语句
date: 2017-03-18 12:26:15
categories: 编程
tags:
- Python
---

Python3的语句。

<!-- more -->

## 1 print函数

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

## 2 赋值

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

## 3 闭包与nonlocal语句

nonlocal语句提供了一种解决嵌套函数引用外部变量的方法，在Python2.x中，闭包只能读取外部函数的变量，而不能改写它。

举例来说，这样是合法的。

```python
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

## 4 异常语句

异常语句变更，由 `except exc,var`变为 `except exc as var`，捕获多个异常时使用 `except (exc1, exc2) as var`。
