---
title: python小记
date: 2017-06-08 08:59:54
categories: 编程
---

## 连接二进制数据

<!-- more -->

使用 sum 方法

```
F:\kinegratii.github.io>python3
Python 3.6.0 (v3.6.0:41df79263a11, Dec 23 2016, 07:18:10) [MSC v.1900 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> t = [b'\x01', b'\x01\x01']
>>> sum(t)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'bytes'
>>> ^C
F:\kinegratii.github.io>python
Python 2.7.11 (v2.7.11:6d1b6a68f775, Dec  5 2015, 20:32:19) [MSC v.1500 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> t = [b'\x01', b'\x01\x01']
>>> sum(t)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'str'
>>> ^C
```

使用 join 方法

```
F:\kinegratii.github.io>python
Python 2.7.11 (v2.7.11:6d1b6a68f775, Dec  5 2015, 20:32:19) [MSC v.1500 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> t = [b'\x01', b'\x01\x01']
>>> b''.join(t)
'\x01\x01\x01'
>>> ^C
F:\kinegratii.github.io>python3
Python 3.6.0 (v3.6.0:41df79263a11, Dec 23 2016, 07:18:10) [MSC v.1900 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> t = [b'\x01', b'\x01\x01']
>>> b''.join(t)
b'\x01\x01\x01'
>>> ^C
```
使用reduce函数

```
F:\kinegratii.github.io>python
Python 2.7.11 (v2.7.11:6d1b6a68f775, Dec  5 2015, 20:32:19) [MSC v.1500 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import operator
>>> t = [b'\x01',b'\x01\x01']
>>> reduce(operator.add,t,b'')
'\x01\x01\x01'
>>> ^C
F:\kinegratii.github.io>python3
Python 3.6.0 (v3.6.0:41df79263a11, Dec 23 2016, 07:18:10) [MSC v.1900 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import operator
>>> from functools import reduce
>>> t = [b'\x01',b'\x01\x01']
>>> reduce(operator.add,t,b'')
b'\x01\x01\x01'
>>> ^C
```

连接二进制数据应该使用 `b''.join(t)` 或者 `reduce(operator.add,t,b'')`，`sum`函数应该只支持数字类型的。
