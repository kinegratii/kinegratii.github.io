---
title: 浅谈Python元类在Django中的Field.choices的应用
date: 2016-11-02 16:05:57
categories:
- 编程
tags:
- Python
---

  使用Python元类改写Django模型字段的choices，使其更加简单可读。

## Django Field.choices

choices是Django定义模型字段的可选参数，适用于所有的字段类型。

choices是一个可迭代的对象，其中每个元素包含两个值。每个元素的第一个元素是写入数据库的真实值，第二个值是可读的名称。类似 n x 2 的二维数组。比如

```
YEAR_IN_SCHOOL_CHOICES = (
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
)
```

当某个字段设置了这个参数，默认的表单组件将变成一个携带这些选项的选择框，而不是标准的文本输入框。

一般来说，choices变量应该在model内部定义，而且每个值都用一个常量赋值定义。

<!-- more -->

```
from django.db import models

class Student(models.Model):
    FRESHMAN = 'FR'
    SOPHOMORE = 'SO'
    JUNIOR = 'JR'
    SENIOR = 'SR'
    YEAR_IN_SCHOOL_CHOICES = (
        (FRESHMAN, 'Freshman'),
        (SOPHOMORE, 'Sophomore'),
        (JUNIOR, 'Junior'),
        (SENIOR, 'Senior'),
    )
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
    )

    def is_upperclass(self):
        return self.year_in_school in (self.JUNIOR, self.SENIOR)
```

使用方法

```
>>> s = Student(year_in_school=Student.JUNIOR)
>>> s.year_in_school
'JR'
>>> s.get_year_in_school_display()
'Junior'
>>> y = 'Et'
y in [c[0] for c in Student.YEAR_IN_SCHOOL_CHOICES]
True

```

## 存在的问题


- 在不考虑多变量同时赋值，每增加一个取值时，需要添加两行代码
- 由于常量变量（如`JUNIOR`和`SENIOR`）直接定义在Model中，当Model有两个字段同时定义了choices，很难区分哪些常量值是属于同一字段的不同取值。
- 判断取值是否有效时，每次都要构造值的列表 、 集合。

##  改进后的Field.choices

代码如下

```
class ChoicesMetaclass(type):
    def __new__(cls, name, bases, attrs):
        choices = []
        values = {}
        ks = []
        for k, v in six.iteritems(attrs):
            if k.isupper() and not k.startswith('_'):
                if isinstance(v, tuple) and len(v) == 2:
                    pass
                else:
                    v = v, v
                ks.append(k)
                choices.append(v)
                values[v[0]] = v[1]
                attrs[k] = v[0]
        attrs['choices'] = choices
        attrs['values'] = values
        return type.__new__(cls, name, bases, attrs)


class ConstChoices(six.with_metaclass(ChoicesMetaclass)):
    @classmethod
    def is_valid(cls, value):
        return value in cls.values

    @classmethod
    def get_value_display(cls, value):
        return cls.values.get(value)
```

要点

- 将同一个choices单独封装在一个类当中。
- 在定义常量过程中，自动计算choices和values。
- 每一个项取值都放在一行，如`FRESHMAN = 'FR', 'Freshman'`

使用方法

```
class Student(models.Model):
    class YearInShoolChoices(ConstChoices):
        FRESHMAN = 'FR', 'Freshman'
        SOPHOMORE = 'SO', 'Sophomore'
        JUNIOR = 'JR', 'Junior'
        SENIOR = 'SR', 'Senior'

    year_in_school = models.CharField(
        max_length=2,
        choices=YearInShoolChoices.choices,
        default=YearInShoolChoices.FRESHMAN,
    )

    def is_upperclass(self):
        return self.year_in_school in (YearInShoolChoices.JUNIOR, YearInShoolChoices.SENIOR)
```

使用方法

```
>>> s = Student(year_in_school=Student.JUNIOR)
>>> s.year_in_school
'JR'
>>> s.get_year_in_school_display()
'Junior'
>>> y = 'Et'
YearInShoolChoices.is_valid(y)
True
```
