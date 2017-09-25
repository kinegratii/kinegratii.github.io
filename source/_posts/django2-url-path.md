---
title: django2笔记:路由path语法
date: 2017-09-25 18:09:15
categories: 编程
tags:
- Django
---

9月23日Django发布了2.0a1版本，这是一个 feature freeze 版本，如果没有什么意外的话，2.0正式版不会再增加新的功能了。按照以往的规律，预计正式版将在12月发布。

2.0无疑是一个重要的版本，因为这是第一个只支持Python3.X的版本，和1.x是不兼容的。

今天我们来看看一个新语法：使用path方式表示路由路径，即新的 `django.urls.path` 函数，它允许使用一种更加简洁、可读的路由语法。比如：之前的版本的代码：

```python
url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
```

在新版本中也可以写为：

```python
path('articles/<int:year>/', views.year_archive),
```

新语法支持类型转化，因此 year_archive的year参数就变成整数而不是字符串。

如果你有接触过 Flask 框架，就会发现和 [Variable-Rules](http://flask.pocoo.org/docs/0.12/quickstart/#variable-rules) 的语法形式和功能都是相类似的。

<!-- more -->

## 问题引入

下面是Django1.X的一段代码：

```python
from django.urls import url

def year_archive(request, year):
    year = int(year) # convert str to int
    # Get articles from database

def detail_view(request, article_id):
    pass

def edit_view(request, article_id):
    pass

def delete_view(request, article_id):
    pass
  
urlpatterns = [
    url('articles/(?P<year>[0-9]{4})/', year_archive),
    url('article/(?P<article_id>[a-zA-Z0-9]+)/detail/', detail_view),
    url('articles/(?P<article_id>[a-zA-Z0-9]+)/edit/', edit_view),
    url('articles/(?P<article_id>[a-zA-Z0-9]+)/delete/', delete_view),
]
```

考虑下这样的两个问题：

第一个问题，函数 `year_archive` 中year参数是字符串类型的，因此需要先转化为整数类型的变量值，当然 `year=int(year)` 不会有诸如如TypeError或者ValueError的异常。那么有没有一种方法，在url中，使得这一转化步骤可以由Django自动完成？

第二个问题，三个路由中article_id都是同样的正则表达式，但是你需要写三遍，那么有没有一种方法，使得三个路由共用正则表达式，这样以后表达式变更后，只需修改一处？

在Django2.0中，可以使用 `path` 解决以上的两个问题。

## 基本示例

这是一个简单的例子：

```python
from django.urls import path

from . import views

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<slug>/', views.article_detail),
]
```

基本规则：

- 使用尖括号(`<>`)从url中捕获值。
- 捕获值中可以包含一个转化器类型（converter type），比如使用 `<int:name>` 捕获一个整数变量。若果没有转化器，将匹配任何字符串，当然也包括了 `/` 字符。
- 无需添加前导斜杠。

以下是一些示例及其匹配过程：

| 请求URL                                    | 匹配项  | 视图调用形式                                   |
| ---------------------------------------- | ---- | ---------------------------------------- |
| /articles/2005/03/                       | 第3个  | views.month_archive(request, year=2005, month=3) |
| /articles/2003/                          | 第1个  | views.special_case_2003(request)         |
| /articles/2003                           | 无    | -                                        |
| /articles/2003/03/building-a-django-site/ | 第4个  | views.article_detail(request, year=2003, month=3, slug="building-a-django-site") |

## path转化器

文档原文是Path converters，暂且翻译为转化器。Django默认支持以下5个转化器：

- str,匹配除了路径分隔符（`/`）之外的非空字符串，这是默认的形式
- int,匹配正整数，包含0。
- slug,匹配字母、数字以及横杠、下划线组成的字符串。
- uuid,匹配格式化的uuid，如 075194d3-6885-417e-a8a8-6c931e272f00。
- path,匹配任何非空字符串，包含了路径分隔符

##　注册自定义转化器

对于一些复杂或者复用的需要，可以定义自己的转化器。转化器是一个类或接口，它的要求有三点：

- `regex` 类属性，字符串类型

- `to_python(self, value)` 方法，value是由类属性 `regex` 所匹配到的字符串，返回具体的Python变量值，以供Django传递到对应的视图函数中。

- `to_url(self, value)` 方法，和 `to_python` 相反，value是一个具体的Python变量值，返回其字符串，通常用于url反向引用。

例子：

```python
class FourDigitYearConverter:
    regex = '[0-9]{4}'

    def to_python(self, value):
        return int(value)

    def to_url(self, value):
        return '%04d' % value
```

使用`register_converter` 将其注册到URL配置中：

```python
from django.urls import register_converter, path

from . import converters, views

register_converter(converters.FourDigitYearConverter, 'yyyy')

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<yyyy:year>/', views.year_archive),
    ...
]
```

##  使用正则表达式

如果上述的paths和converters还是无法满足需求，也可以使用正则表达式，这时应当使用 `django.urls.re_path` 函数。

在Python正则表达式中，命名式分组语法为 ` (?P<name>pattern)` ，其中name为名称， pattern为待匹配的模式。

之前的示例代码也可以写为：

```python
from django.urls import path, re_path

from . import views

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    re_path('articles/(?P<year>[0-9]{4})/', views.year_archive),
    re_path('articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/', views.month_archive),
    re_path('articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<slug>[^/]+)/', views.article_detail),
]
```

这段代码和之前的代码实现了基本的功能，但是还是有一些区别：

- 这里的代码匹配更加严格，比如year=10000在这里就无法匹配。
- 传递给视图函数的变量都是字符串类型，这点和 `url` 是一致的。

**无命名分组**

一般来说，不建议使用这种方式，因为有可能引入歧义，甚至错误。

##　Import变动

`django.urls.path` 可以看成是 `django.conf.urls.url` 的增强形式。

为了方便，其引用路径也有所变化。

| 1.X                      | 2.0                 | 备注         |
| ------------------------ | ------------------- | ---------- |
| -                        | django.urls.path    | 新增，url的增强版 |
| django.conf.urls.include | django.urls.include | 路径变更       |
| django.conf.urls.url     | django.urls.re_path | 异名同功能，url  |

## 总结

新的path语法：

- 类型自动转化
- 公用正则表达式

本文根据Django2.0官方文档整理。