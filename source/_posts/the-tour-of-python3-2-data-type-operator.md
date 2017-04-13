---
title: Python3之旅（二）—— 数据类型与操作符
date: 2017-02-25 10:50:25
categories: 编程
tags:
- Python
---

## 1 整数

- Python3.X不再有long的数据类型，合并到int中，因此像 `34023932232L` 和 `34023932232l` 将引发语法错误。
- `sys.maxint`常量被移除，因为整数没有任何限制。
- 八进制数使用更加明确的0o前缀，代码 `t = 070` 将产生语法错误，`int('070')` 结果为70。
- 数字字面量支持下划线划分 (【Python3.6+】)

在数字字面量使用下划线：

```python
# grouping decimal numbers by thousands
amount = 10_000_000.0

# grouping hexadecimal addresses by words
addr = 0xCAFE_F00D

# grouping bits into nibbles in a binary literal
flags = 0b_0011_1111_0100_1110

# same, for string conversions
flags = int('0b_1111_0000', 2)
```

> PEP 链接
- [PEP 515 -- Underscores in Numeric Literals | Python.org](https://www.python.org/dev/peps/pep-0515/)

<!-- more -->

## 2 列表、字典和迭代器

一些常用的API不再返回列表

- 字典中  `dict.keys()`, `dict.items()` 和 `dict.values()` 等方法返回 “views” 。代码 `k = d.keys(); k.sort()`在3.X无法正常工作。应该使用内置的`sorted`函数 `k = sorted(d)`。
- `dict.iterkeys()`, `dict.iteritems()` 和 `dict.itervalues()`等方法不再支持。
- 内置函数 `map()` 和 `filter()` 返回迭代器，使用 `list(map())` 或者列表推导式返回数据列表。
- 3.X的`range` 函数同2.X的`xrange`一样，返回的是一个迭代器， `xrange` 在3.X被移除
- `zip`函数返回一个迭代器

## 3 除法

Python3中针对除法作了比较大的改变，`/`由地板除法改为了精确除法。

| 操作符 | Python2 | Python3 |
| ------ | ------ | ------ |
| /  | 地板除法 | 精确除法 |
| // | 地板除法 | 地板除法 |

代码示例如下表：

| 操作符 | Python2 | Python3 | 备注 |
| ------ | ------ | ------ | |
| /  | 5 / 2 = 2 | 5 / 2 = 2.5 | |
| | 5.0 / 2 = 2.5 | 5.0 / 2 = 2.5 | |
| | 4 / 2 = 2 | 4 / 2 = 2.0 | |
| | 4.0 / 2 = 2.0 | 4.0 / 2 = 2.0 | |
| // | 5 // 2 = 2 | 5 // 2 = 2 | |
| | 5.0 // 2 = 2.0 | 5.0 // 2 = 2.0 | |
| | 4 // 2 = 2 | 4 // 2 = 2 | |
| | 4.0 // 2 = 2.0 | 4.0 // 2 = 2.0 | |


> PEP 链接
- [PEP 238 -- Changing the Division Operator | Python.org](https://www.python.org/dev/peps/pep-0238/)

## 4 矩阵乘法

Python3.5增加了用于矩阵乘法的操作符 `@` 。但目前所有内置类型还不支持这个操作符，可以通过定义 `__matmul__`、`__rmatmul__` 和 `__lmatmul__` 三个魔术方法实现。

引入操作符后，将使得代码变得更加可读性。

@操作符

```python
S = (H @ beta - r).T @ inv(H @ V @ H.T) @ (H @ beta - r)
```

函数实现

```python
S = dot((dot(H, beta) - r).T,
        dot(inv(dot(dot(H, V), H.T)), dot(H, beta) - r))
```

NumPy 1.10已经增加了`@`操作符的支持。

```
>>> import numpy

>>> x = numpy.ones(3)
>>> x
array([ 1., 1., 1.])

>>> m = numpy.eye(3)
>>> m
array([[ 1., 0., 0.],
       [ 0., 1., 0.],
       [ 0., 0., 1.]])

>>> x @ m
array([ 1., 1., 1.])
```

## 5 比较

3.0 简化了比较规则：

- 当两个操作数类型不同，比较操作符（`> >= <= <`）将引发一个 `TypeError` 异常。因此像`1 < ''`、`0 > None` 、`len < len`、`None < None` 不再有效。同样的道理，不同类型组成的列表也不具有比较的意义。
- 当两个操作数属于不同类型时，`==`和`!=`操作符总是返回False和True
- `builtin.sort()`和`list.sorted()`不再支持cmp参数，应该使用key参数，注意的的是key和reverse参数必须以关键字方式传入。
- Python2中的 `builtin.cmp()`和类中的魔术方法`__cmp__`不推荐使用
