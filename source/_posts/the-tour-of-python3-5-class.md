---
title: Python3之旅（五）——类与类继承
date: 2017-04-29 20:12:58
categories: 编程
tags:
- Python
---

## 1 经典类与新式类

从2.2开始引入新式类。与经典类的区别：

- Python 2.X默认都是经典类，只有显式继承object的类才是新式类，而在Python3.X中默认都是新式类，无需显式继承object。
- 在多继承中，新式类采用广度优先搜索，而旧式类是采用深度优先搜索。
- 新式类对象可以直接通过 `__class__` 属性获取自身类型:type
- 新式类增加了 `__slots__` 内置属性, 可以把实例属性的种类锁定到 `__slots__` 规定的范围之中
- 新式类增加了 `__getattribute__` 方法
- `super` 函数用法差异

```python

# Python2
class A(object):
    pass

# Python3
class A:
    pass
```

<!-- more -->

**super函数**

关于super函数定义在 [PEP3135](https://www.python.org/dev/peps/pep-3135/) ，super演变过程描述历经三个过程：

1 在Python2的旧式类中，用法是这样子：

```python
class MySubClass(MySuperClass):
     def __init__(self):
         MySuperClass.__init__(self)
```
2 为了使用代码更加抽象，Python2的新式类改成 `super(<class>, <instance>)`，上述 `__init__` 变成


```python
class MySubClassBetter(MySuperClass):
    def __init__(self):
        super(MySubClassBetter, self).__init__()
```

3 在Python3中，支持无参数super调用。

```python
class MySubClassBetter(MySuperClass):
    def __init__(self):
        super().__init__()
```

## 2 类继承

新式类采用广度优先搜索，而旧式类是采用深度优先搜索，在新式类中可以通过 `__mro__` 树形查看继承结构。

```
>>> class New(object): pass
>>> class N1(New): pass
>>> class N2(New): pass
>>> class ND(N1, N2): pass
>>> ND.__mro__
(<class '__main__.ND'>, <class '__main__.N1'>, <class '__main__.N2'>, <class '__main__.New'>, <type 'object'>)
```

## 3 混合(Mixin)

混合在Django CBV中运用的非常频繁。和类继承不同的是，单独的Mixin不可实例化和执行其方法，需要配合其他主体类执行。

## 4 元类

元类的定义方式有所变更，下面是Python2的定义形式。
```
class C:
    __metaclass__ = M
```
而Python3废弃了 `__metaclass__` 定义，改为 metaclass 参数。
```
class C(metaclass=M):
    pass
```

## 5 参考资料

- Python新式类和经典类的区别 - u010066807的博客 - 博客频道 - CSDN.NET
http://blog.csdn.net/u010066807/article/details/46896835
- https://wiki.python.org/moin/NewClassVsClassicClass
- Python 3 Object Oriented
https://www.tutorialspoint.com/python3/python_classes_objects.htm
- Python3 Tutorial: Inheritance
http://www.python-course.eu/python3_inheritance.php
