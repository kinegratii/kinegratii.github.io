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

参考
- Python新式类和经典类的区别 - u010066807的博客 - 博客频道 - CSDN.NET
http://blog.csdn.net/u010066807/article/details/46896835
- https://wiki.python.org/moin/NewClassVsClassicClass

## 2 类继承和混合(Mixin)

新式类采用广度优先搜索，而旧式类是采用深度优先搜索

## 3 元类

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
- Python3 Tutorial: Inheritance
http://www.python-course.eu/python3_inheritance.php