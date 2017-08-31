---
title: Django项目全栈笔记
date: 2017-07-31 10:55:06
categories: 编程
tags:
- 项目
- Django
---

## 概述

BWS项目是自己一年来在开发的项目。本文就项目中一些技术选型、功能实现、项目流程做一些简单的总结。

<!-- more -->

## python3

虽然Python3发布已经10年之久，但是2和3之争直到今天依旧存在，对于如何学习，每个人都有自己的理解和学习策略。我个人看法：

- 果断学习面向未来的3；
- 3和2的差别对于学习过程没有太多的影响，学好了3自然也能够很快上手2了；
- 常见的第三方库大多数（80%以上）是支持Python3的；
- 不过在系统安装的时候总是23共存的，可以自由切换；

今年4月份重新开发的时候做了一个比较激进的做法：**完全抛弃对Python2的支持**。当时的考量，把这个项目作为Django持续学习的一个示范项目，毕竟 Django的下一个大版本2.0（预计2017年12月发布），也已经要求最低版本为3.5了。

其实在项目中使用几个Python2不支持的语法和没有的标准库，就可以达到以上目的，具体来说，在代码中使用了以下几个语法：

- 字典合并创建语法（[PEP448](https://www.python.org/dev/peps/pep-0448/)）
- 强制关键字传参（[PEP3102](https://www.python.org/dev/peps/pep-3102/)）

根据 PEP448，在Python3.5中可以使用更加简洁明了的代码实现合并多个字典。

```python
# 3.5+
combination = {**first_dictionary, "x": 1, "y": 2}

# 3.5以下
combination = first_dictionary.copy()
combination.update({"x": 1, "y": 2})
```

另外Python3在文本和二进制方面作了比较大的改变，这对文件导入导出功能开发提供了便利，不用再纠结2的编码问题，可以集中解决业务层面的问题。

## Django

我也算是Django的忠实用户了，从1.4到1.11都有用到，不断看到Django的成长和壮大。1.4/1.8/1.11是LTS版本，项目使用的是1.10。这些年来Django比较大的变更有：

- 自定义用户类型：这个是1.5就有的功能了，之前只能使用内置的用户类，连使用邮箱作为用户名也不能直接支持；
- 数据库迁移：1.7借鉴 [South](https://bitbucket.org/andrewgodwin/south/) 实现的，这个是开发的利器，修改用户模型时候可以使用命令一键将修改同步到数据库，而忽略具体的数据库类型；
- 自定义过滤查询：1.7，这个主要用于封装一些业务数据库查询。
- 多模板支持：1.8引入的，Django自己的模板引擎效率历来为人们所诟病，现在可以在Django中使用Jinja2这样的模板了。
- 表单控件支持模板渲染：Django表单其实是着重于后端验证，前端相对薄弱，导致定制起来没有那么顺手。最新的1.11引入的可以通过模板文件定制控件样式等等。

### CBV

**强烈建议使用 Class-Based-View 组织视图处理函数。**

Class-Based-View 是相对于Function-Based-View而言，主要支持封装，减少重复的代码编写工作，逻辑流程清晰，经过测试过的。在具体编写代码还是一定要查看源代码，才能理解其中的功能实现。

CBV的核心是Mixin模式。

> Mixin是一种将多个类中的功能单元的进行组合的利用的方式，这听起来就像是有类的继承机制就可以实现，然而这与传统的类继承有所不同。通常mixin并不作为任何类的基类，也不关心与什么类一起使用，而是在运行时动态的同其他零散的类一起组合使用。

> 使用mixin机制有如下好处：可以在不修改任何源代码的情况下，对已有类进行扩展；可以保证组件的划分；可以根据需要，使用已有的功能进行组合，来实现“新”类；很好的避免了类继承的局限性，因为新的业务需要可能就需要创建新的子类。

现在也基本上不写视图函数了，项目上能见到的也就是 `django.contrib.auth.login` 等几个函数了，不过现在也要改成视图类形式了。

### 是否启用admin


虽然admin是Django的主要优势所在，但是它的使用场景有限，主要由于整合许多功能，比如分页、过滤、搜索、增删改查和批量操作等等，相互之间具有非常高的耦合度。在没有提供公开的API下去实现一些定制往往是“牵一发而动全身”，最后基本上也改的是不成样子。

由于项目中没有使用内置的admin组件，增删改查的页面就需要多花一点时间自己去适配。

日志模型也要自己去设计，项目中我自己添加了ip这个字段，这个是原来所没有的。

## 后端数据API - DRF

后端数据采用的是 [Django Rest Framework](http://www.django-rest-framework.org/) 这个框架，覆盖了大多数需求，包括：

- 搜索/分页
- 访问权限
- 请求限制（频率、IP）
- 表单验证

## 前端 Amaze UI

前端UI用的是 [Amaze UI](http://amazeui.org/)这个框架。不过从后来的发展形势来看，这是最为错误的决定了，主要原因在于无法和后端比较平稳地整合。

Django表单中有一个比较大的问题，如何需要定制控件样式，需要在Python代码中修改，而且需要应用的每一处都需要更改，灵活度不够。目前主要有两种解决方式：

- 使用Django1.11版本的[模板功能](https://docs.djangoproject.com/en/1.11/ref/forms/renderers/#)，这个功能刚刚推出，文档也只有一页的内容，不太建议使用。
- 使用 [django-crispy-forms](https://github.com/django-crispy-forms/django-crispy-forms) 第三方库，这个库的思路也是使用模板html文件渲染控件，已经有一定的使用规模，但是支持 Bootstrap这样常见的UI框架，不支持 Amaze UI。

## 导入导出

实现导入导出功能主要使用的是 [tablib](https://github.com/kennethreitz/tablib) 和 [django-import-export](https://github.com/django-import-export/django-import-export) 这两个库，其中后者依赖前者。

### 导出

编写 `Resource`， 几点值得注意的地方：

- 需要设置表头，不仅需要指定字段`Meta.fields`，同时也要显示指定 `Meta.export_order` 的值，通常和 `Meta.fields`一样即可。
- `Meta.fields` 里的元素必须是模型的数据库字段，不能是自定义的 property，这一点和 `ModelAdmin.list_display` 不一样。
- 可以使用 `dehydrate_FOO` 函数重写导出内容

```python
class BillResource(resources.ModelResource):
    def get_export_headers(self):
        return ['流水号','月份', '类型','单价','用量', '金额']

    def dehydrate_price(self, obj):
        return obj.get_price_display

    class Meta:
        model = models.Bill
        fields = ('id', 'month', 'resource_type','price', 'amout', 'total')
        export_order = fields
```

### 导入

django-import-export也提供了几个Mixin，但问题这些和admin组件耦合很高，不利于一些自定义操作，所以直接使用tablib库比较好。根据官方文档，可以使用以下代码实现文件导入

```python
imported_data = Dataset().load(open('data.csv').read())
```

但其实load还有几个比较重要的参数：
- format：文件格式，如果不写，使用自动识别，但有出错的机率，之前试验过一个xlsx文件在不同环境下识别为json文件的，因此建议这个参数也留给用户输入
- headers：表示第一行是否是表头，这个参数在文档中没有表明，需要自行查看源代码获取相关信息。

所以最后就写成下面这个样子
```python

class BillUploadForm(form.Form):
    import_file = forms.FileField()
    format = forms.ChoiceField(choices=(('xlsx', 'xlsx'), ('xls', 'xls')))

# 导入

tablib.Dataset().load(form.cleaned_data['import_file'].read(), format=form.cleaned_data['format'], headers=False)
```

## 数据库查询优化

### select_related函数

select_related是 `django.db.models.QuerySet` 类的一个方法，它解决了 ORM中常见的N+1查询效率问题，关于这一部分可以参考我之前写过的一篇文章[《select_related函数性能基本测试》](/2017/07/11/django-select-related-performance/)。

### 更新记录

在更新记录时可以使用 update_fields 参数指定只需更新的字段列表。这个参数在只更新一两个字段的时候特别有用。

```python
product.name = 'Name changed again'
product.save(update_fields=['name'])
```

如果不指定参数的值，将更新所有字段。

## 测试和部署

### 分离配置文件

Django使用 `settings` 模块配置相关参数，这使得其很好的区分开发/测试/生产。

```
- BillWorkingSystem
    - BillWorkingSystem
        - __init__.py
        - settings.py
        - test_settings.py
        - urls.py
        - wsgi.py
```

一个简单的test_settings.py如下，可以配置一些仅用于测试的项目，如数据目录 FIXTURE_DIRS 。

```python
from BillWorkingSystem.settings import *


class DisableMigrations(object):
    def __contains__(self, item):
        return True
    def __getitem__(self, item):
        return "notmigrations"

MIGRATION_MODULES = DisableMigrations()

FIXTURE_DIRS = (
    os.path.join(BASE_DIR, 'fixtures').replace('\\', '/'),
)

UPLOAD_TEST_DATA_DIR = os.path.join(BASE_DIR, 'fixtures', 'test_data').replace('\\', '/')
```

`MIGRATION_MODULES ` 设置表示是否运行数据迁移脚本。上述例子设置为空，表示测试无需运行这些迁移脚本。

### 单元测试

单元测试主要测试那些返回为实际数据（如json/yaml）的视图。

测试采用标准的 `django.test.TestCase`，按照文档所描述的步骤，一步一步的编写。

```python
class BillCreateTestCase(TestCaseBase):
    url = '/api/bill/create/'

    def test_success(self):
        data = {
            'enterprise': 1,
            'year': 2017,
            'month': 1,
            'amount': 4000,
            'create_name': 'Test'
        }
        rsp = self.client.post(self.url, data)
        self.assertEqual(201, rsp.status_code)
        # 其他assert语句

    def test_with_error_enterprise(self):
        data = { } # 参数
        rsp = self.client.post(self.url, data)
        self.assertEqual(400, rsp.status_code)
        # 其他assert语句
```
一个基本模式，
- 一个TestCase表示一个主要功能，如一个POST请求
- 每个 `test_FOO` 函数表示一种情况，包括正确和无效参数
- 依次对响应对象的状态码和内容、数据变更进行断言(Assert)测试

### docker部署

之前采用的是daocloud这个平台的工具。具体可参考[《使用DaoCloud部署Django项目》](https://kinegratii.github.io/2016/07/23/daocloud-django-deploy/)这篇文章。由于对docker这方面没有一个完整的学习，加上daocloud.io更新到3版本，作了一些比较大的改变，后来就决定搬迁到阿里云服务器上，这样相对比较容易把握。

## 后记

### 轮子

什么是轮子 wheel，写多了代码就会发现一些代码具有共同之处，将其抽象并提取，慢慢地就形成了一个库，可以和别人分享。本质上来说，Django也是一个轮子。

### 持续开发

由于是个人项目，因此一些版本升级方面就比较随意，基本上新版本出来就完全废弃旧有版本。

### 第三方库

稳定才是真，不要为追求标新立异而盲目升级。
