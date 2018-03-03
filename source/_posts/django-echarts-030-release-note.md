---
title: django-echarts v0.3 发布日志
date: 2018-02-25 18:47:04
categories: 技术研究
tags:
- 数据可视化
- 项目
---

> 本文已收录于 [《pyecharts 开发专辑》](/pyecharts-project/) 。

django-echarts v0.3.0 正式发布。该版本将 **仅支持** Python3.5+ 以及 Django2.0+ 的环境，同时该增加了若干个功能特性：

- 移除对 Python2 的支持
- 新增计数模块 `datasets.section_counter`
- 部分函数增加 Key-Only Arguments ( [PEP 3102](https://www.python.org/dev/peps/pep-3102/) ) 限定
- 下载命令增加 `--fake` 选项，支持预览调试
- 整合单元测试
- 发布数据构建文档

<!-- more -->

## 1 Python3迁移

django-echarts 使用了更为激进的迁移策略，v0.3之后将仅支持  Python3.5+ 以及 Django2.0+ 的运行环境，不再支持 Python2 ，django-echarts v0.3 将在 Python2 环境中出现语法层面的错误。

具体来说就是  [PEP 3102](https://www.python.org/dev/peps/pep-3102/) 的应用，在3.X中新增了强制关键字参数传递（Keyword-Only Arguments）中，定义的形式为使用 * 单独占用一个参数，表示之后的参数必须以关键字方式传入参数，否则将引发TypeError异常。

例子：

django_echarts.plugins.host.HostStore

```python
class HostStore(object):
    HOST_DICT = {}

    def __init__(self, *, context=None, default_host=None):
        pass
```

django_echarts.plugins.store.SettingsStore

```python
class SettingsStore(object):
    def __init__(self, *, echarts_settings=None, extra_settings=None, **kwargs):
        pass
```

是否使用 Keyword-Only Arguments ，自己根据实际情况总结了一些应用场景。

- 函数有两个以上的可选参数（提供了默认参数）
- 这些参数的功能意义是平等的，通常可任意调换位置

比如 django_echarts.datasets.section_counter.BSectionIndex 类的 `__init__` 就没有使用这个特性，因为 [lower, upper] 更符合实际表达形式。

```python
class BSectionIndex(BIndex):
    def __init__(self, lower=None, upper=None):
        pass
```



## 2 计数模块 section_counter

该模块针对常见的数据计数业务场景进行的封装，该模块基于内置 `collections.Counter` 模块，并基于此进行了一些扩展。

BSectionCounter 库用于计算符合一系列条件的数目计数类。

先看一个例子

```python
data_list = list(df['stars'])
labels = ['00~00', '01~10', '11~50', '51~100', '101~500', '501~1000', '>1000']
sizes = []
sizes.append(len([pp for pp in data_list if pp == 0]))
sizes.append(len([pp for pp in data_list if pp >= 1 and pp <= 10]))
sizes.append(len([pp for pp in data_list if pp >= 11 and pp <= 50]))
sizes.append(len([pp for pp in data_list if pp >= 51 and pp <= 100]))
sizes.append(len([pp for pp in data_list if pp >= 101 and pp <= 500]))
sizes.append(len([pp for pp in data_list if pp >= 501 and pp <= 1000]))
sizes.append(len([pp for pp in data_list if pp >= 1001]))
stargazer_bar = Bar("stars", "stars hist graph of users", width=CHART_WIDTH)
stargazer_bar.add("", labels, sizes, is_label_show=True, mark_line=["average"])

```

使用 BSelectionCounter 后，简化为

```python
data_list = list(df['stars'])
rc1 = BSectionCounter(
    BValueIndex(0),
    BSectionIndex(1, 10),
    BSectionIndex(11, 50),
    BSectionIndex(51, 100),
    BSectionIndex(101, 500),
    BSectionIndex(501, 1000),
    BSectionIndex(1001)
)
labels, sizes = rc1.feed_as_axises(data_list)
stargazer_bar = Bar("stars", "stars hist graph of users", width=CHART_WIDTH)
stargazer_bar.add("", labels, sizes, is_label_show=True, mark_line=["average"])
```

## 3 下载命令支持预览调试

增加 `--fake` 后，命令将仅打印出对应文件的下载路径、引用路径、存储位置，而 **不会进行任何实际的下载操作** ，可用于预览调试。

例子：

```
>>>python manage.py download_echarts_js echarts.min china --fake
[Info] Download Meta for [echarts.min]
        Remote Url: https://cdn.bootcss.com/echarts/3.7.0/echarts.min.js
        Local  Url: /static/echarts/echarts.min.js
        Local Path: E:\projects\django-echarts\example\static\echarts\echarts.min.js
[Info] Download Meta for [china]
        Remote Url: http://echarts.baidu.com/asset/map/js/china.js
        Local  Url: /static/echarts/china.js
        Local Path: E:\projects\django-echarts\example\static\echarts\china.js
```

fake 命名灵感来自于 [migrate命令](https://docs.djangoproject.com/en/2.0/ref/django-admin/#cmdoption-migrate-fake) 。

## 4 发布数据构建文档

v0.3.0 开始，有关数据构建的文档将独立出来。一方面，从功能上来说，数据构建模块仅是数据创建和渲染过程中可能使用到的工具性代码，并不是核心功能。

另一方面，由于该模块的工具特性使其具有更为一般的通用性，因此在后续开发中，有考虑将其纳入 正在编写的 "pyecharts-contrib计划''之中。

> pyecharts-contrib 计划旨在于构建通用、简单的脚手架模板，和提供解决数据可视化领域中一些常见问题的工具集合。使用者可以迅速地基于 contrib 开始新的项目。pyecharts-contrib 将追求遵循[“batteries included” philosophy](https://docs.python.org/3/tutorial/stdlib.html#tut-batteries-included) 。pyecharts-contrib 命名的灵感来自于 `django.contrib` 。
>
> 目前，该计划正在紧张有序的进行中。

## 5 其他功能改进

其他部分功能改进。

- `FieldValuesQuerySet.fetch_values` 类和方法重名为 `AxisValuesQuerySet.as_axis_values` 更加符合实际意义
- 整合测试样例
