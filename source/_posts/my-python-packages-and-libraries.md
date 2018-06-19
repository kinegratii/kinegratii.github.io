---
title: 我的 Python 第三方库
date: 2018-06-10 20:37:33
categories: 编程
tags:
- Python
---

自己常用的 Python 第三方库。

<!-- more -->

## six

> 2 x 3 = 6

- 地址： https://pythonhosted.org/six/

Python2和3的兼容库。

## wheel / twine

第三方库 **构建 - 分发** 工具。

使用姿势： github + travis CI + wheel + twine ，参考  [python项目持续集成与包发布](https://kinegratii.github.io/2017/04/25/python-project-ci-publish/) 。

## Django

地址：https://www.djangoproject.com/

Python web 领域两大框架之一，遵循 “大而全”、“开箱即用”的原则和规范。Django 源代码内部耦合度高、文档齐全、使用者众多。

在实际业务开发方面，有非常多的第三方库可以使用，具体可参见 [Django Packages](https://djangopackages.org/) 和 [Awesome Django](http://awesome-django.com/)。

使用姿势： Python 3.6 + Django 2.0

## Graphviz

地址：http://graphviz.org/

图形可视化软件，支持 dot 格式文件。

## Sphinx

文档生成工具，使用 reStructuredText 作为标记语言，同时具有以下优点（引用自 [中文官方网站](http://sphinx-doc-zh.readthedocs.io/en/latest/)）：

- 丰富的输出格式: HTML (包括M$帮助), LaTeX (为PDF输出), manual pages(man), 纯文本
- 完备的交叉引用: 语义化的标签,并对 函式,类,引文,术语以及类似片段消息可以自动化链接
- 明晰的分层结构: 轻松定义文档树,并自动化链接同级/父级/下级文章
- 美观的自动索引: 可自动生成美观的模块索引
- 精确的语法高亮: 基于 Pygments 自动生成语法高亮
- 开放的扩展: 支持代码块的自动测试,自动包含Python 的模块自述文档,等等

使用姿势：  github + sphinx + readthedocs

## click

地址： https://github.com/pallets/click

命令行工具，Pallets 出品。最大的特点是使用 *装饰器* 实现命名格式，和 argparse 相比，更加地 “所见即所用”。

示例

```python
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()
```

## construct

地址：http://construct.readthedocs.io/en/latest/

一个声明式(declarative)、对称式（symmetrical）二进制解析和构建工具，适用于建立大型复杂应用程序的通信协议。

第一，功能强大。能够解决的该领域的大多数问题，并且性能表现良好。

第二，通常应用于项目级，在总体设计、技术选型的步骤。如果只有一两个功能点使用到，就不要使用该库，“杀鸡焉用牛刀”。

第三，入门/学习比较难，它引入很多的自有概念，并且一系高级功能需要通读源码。

第四，在代码方面使用了很多 Python 技巧，比如 元类、魔术方法、设计模式等，源码非常值得学习。

## PyInstaller

![PyInstaller](/images/pyinstaller-draft1c-header-trans.png)

地址：http://www.pyinstaller.org/

之前使用过几个打包工具（py2exe、cx_freeze），还是 PyInstaller 比较好用。程序打包主要难点在于：

- 含有C扩展的第三方库的打包，比如 Pillow、lxml 。
- 数据文件的路径处理
- 图标文件

可以参考 [python打包工具](https://kinegratii.github.io/2016/04/23/python-package/) 这篇文章。

## tablib

地址： http://docs.python-tablib.org/en/latest/

表格型数据的导入/导出/处理库。支持 json/xls/xlsx/yaml/dbf 等多种文件格式。

```
>>> data = tablib.Dataset(headers=['First Name', 'Last Name', 'Age'])
>>> map(data.append, [('Kenneth', 'Reitz', 22), ('Bessie', 'Monke', 21)])

>>> print data.json
[{"Last Name": "Reitz", "First Name": "Kenneth", "Age": 22}, {"Last Name": "Monke", "First Name": "Bessie", "Age": 21}]

>>> print data.yaml
- {Age: 22, First Name: Kenneth, Last Name: Reitz}
- {Age: 21, First Name: Bessie, Last Name: Monke}

>>> data.xlsx
<censored binary data>
```

## networkx

地址： http://networkx.github.io/

网络图的处理库，包含了：

- 良好的数据结构
- 常用的图算法
- 可以与 [matplotlib](https://matplotlib.org/)、[ECharts](http://echarts.baidu.com/) 等配合良好

## 其他


- [Django Rest Framework](https://github.com/encode/django-rest-framework) - Django的Restful框架。
- [django-tenant-schemas](https://github.com/bernardopires/django-tenant-schemas) - 通过PostgreSQL Schemas在Django中实现多租户特性。当初用的时候Django1.7刚刚发布，因为Migrations的缘故，这个库还不支持，只好乖乖的使用1.6了，系统一直运行良好，也就没有必要更新了。
- [django-crispy-forms](https://github.com/django-crispy-forms/django-crispy-forms) - 渲染表单用的，支持Bootstrap2/3/4、Foundation、uni-form等UI框架，非常好用基本上一个模板tags或者filter就可实现，不用再在Python代码中写什么 `{'class':'form-control'}` 了。Django 1.11 也引入了类似 crispy-form 的[基于模板的控件渲染](https://docs.djangoproject.com/en/1.11/ref/forms/renderers/)  - 目前API还没有稳定下来，估计要等到2.0了。
- [django-simple-captcha](https://github.com/mbi/django-simple-captcha) - 非常简单的表单验证码字段，支持随机字符串、算术题和自定义等不同验证方式。
- [pyserial](https://github.com/pyserial/pyserial) - Python的串口访问库。
- [PyModus](https://github.com/riptideio/pymodbus) Modbus通信协议的Python实现。
- [qrcode](https://github.com/lincolnloop/python-qrcode) - Python的生成二维码库，包括命令行工具和代码API。
- [Pelican](http://blog.getpelican.com/) - 静态站点生成器，之前博客就是用Pelican构建的，由于在windows上比较麻烦，改成Hexo了。
- [lxml](http://lxml.de/) - xml和html解析处理库。
- [python-gsmmodem](https://github.com/faucamp/python-gsmmodem) - Python的GSM库，包括收发短信、电话处理等功能，支持常见型号的芯片。
- [python-lunardate](https://github.com/lidaobing/python-lunardate) - 纯Python实现的中国农历库。
