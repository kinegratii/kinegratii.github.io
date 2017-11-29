---
title: pyecharts 开发笔记
date: 2017-11-23 20:53:09
categories:
 - 技术研究
tags:
 - 数据可视化
 - Python
---

本文记录了在 pyecharts 开发过程中的一些想法思路和具体问题分析解决的方案。写作本文的目的主要有两个：一是工作总结，每完成一项工作需要静下心来总结工作得失，这就是一种进步和成长；二是技术分享，对于同一个知识和技术，每个人的理解和思考都是不同的，博客提供了一个很好的分享平台。

本文基于 [PR 自定义模板](https://github.com/chenjiandongx/pyecharts/pull/240) 整理。

<!-- more -->

## 总体思想

### 最初的想法

大约七八月份的时候就尝试了写了 [django-echarts](https://github.com/kinegratii/django-echarts) 这个项目，发现一些细节性问题处理起来不太方便，更为主要的是一个整体的架构没有完整的建立起来。直到十月份的时候才腾下时间思考这个问题。

因为本人是主要从事 Django 开发的，因此代码风格和思想难免受到 Django 的影响，比如基于类的代码、Mixin模式，还有一些变量命名等等。

### 从 OPP 到 OOP

面向过程和面向对象是两种不同的设计和编码方法。在我看来，虽然二者互有优劣，但并不是排他的。在前期开发过程中，使用面向过程的方法更有助于我们描述功能，把场景活动翻译成程序语言，因为我们自己就是一个过程性的思维，“先做什么，再做什么”。

随着开发不断推进，我们逐渐了解其内在的联系，抽象出对象、动作、接口等概念，进而能够应用继承、多态等面向对象的思想。

Python 之所以称之为万能胶水，我觉得一个原因是 Python 在面向过程和面向对象之间切换自由。描述同一语言 Python 不像 Java 那些，一上来就各种类，各种继承。 

### 规则、公开、API

这个主要是和 Python 语言特点有关，Python 是比较灵活的。比如 Python 对于属性权限限制是“约定俗称”的。下面的 `js_dependencies` 属性应当被看成是私有的。

```python
class Chart(object):
    def __init__(self):
        self._js_dependencies = {}
    @property
    def js_dependencies(self):
        return self._js_dependencies
```

但是，你也可以直接使用 `chart._js_dependencies` 来访问，只不过：

- IDE 可能会发出警告（warnings）
- 变更无法预料，从开发者的角度，无需为此语句有效性提供任何保障

使用 `@property` 语法公开了该类的一个访问接口。

当然，何时公开、怎么公开又是另外一个问题了。

### 持续开发与废弃策略

这里的持续性开发指的的公共API的稳定性，更为确切的说是废弃策略。随着项目的不断推进，新代码不断加入，旧代码不断淘汰。但由于开源项目的公开性和考虑其稳定性，无用的代码并总是立即被删除，而是经过一段时间后再删除，在这方面，个人 Django 项目做的比较好，将旧有代码按照淘汰进程分几个等级([链接](https://docs.djangoproject.com/en/1.11/internals/release-process/#deprecation-policy)) ，我自己在此基础上增加了一个等级：Not Recommend ，通常用于重大变更，涉及到核心代码

- 不再推荐使用(Not Recommend)：仅在更新日志和文档中表明
- 废弃(Deprecated)：使用 `warnings` 模块表明
- 移除(Removed)：删除相关代码

## 功能设计与实现

### html转义与Python实现

转义字符串（Escape Sequence）也称字符实体(Character Entity)。在HTML中，定义转义字符串的原因有两个：第一个原因是像“<”和“>”这类符号已经用来表示HTML标签，因此就不能直接当作文本中的符号来使用。为了在HTML文档中使用这些符号，就需要定义它的转义字符串。当解释程序遇到这类字符串时就把它解释为真实的字符。在输入转义字符串时，要严格遵守字母大小写的规则。第二个原因是，有些字符在ASCII字符集中没有定义，因此需要使用转义字符串来表示。

字符串安全

如果启用了手动转义，按需转义变量就是 **你的** 责任。要转义什么？如果你有 一个 *可能*包含 `>` 、 `<` 、 `&` 或 `"` 字符的变量，你必须转义 它，除非变量中的 HTML 有可信的良好格式。转义通过用管道传递到过滤器 `|e` 来实现: `{{ user.username|e }}` 。

当启用了自动转移，默认会转移一切，除非值被显式地标记为安全的。可以在应用中 标记，也可以在模板中使用 |safe 过滤器标记。这种方法的主要问题是 Python 本 身没有被污染的值的概念，所以一个值是否安全的信息会丢失。如果这个信息丢失， 会继续转义，你最后会得到一个转义了两次的内容。

显示地标记值安全的有两种方式：

- 在模板中使用 `safe` 过滤器
- 传递给模板的值用 `Markup` 类包裹下

根据 [官方文档](http://jinja.pocoo.org/docs/2.10/api/#jinja2.Markup) ，`Markup` 可以无需转义即可标记一个字符串为安全的。这是通过实现 `__html__` 接口来实现的。`Markup` 是 `unicode` 的直接子类，拥有其众多的方法和属性。核心代码如下：

```python
class Markup(text_type):
    def __html__(self):
        return self
```



其使用方式如下：

```shell
>>> Markup("Hello <em>World</em>!")
Markup(u'Hello <em>World</em>!')
>>> class Foo(object):
...  def __html__(self):
...   return '<a href="#">foo</a>'
...
>>> Markup(Foo())
Markup(u'<a href="#">foo</a>')
```

> Jinja2 的 [`Markup`](http://docs.jinkan.org/docs/jinja2/api.html#jinja2.Markup) 类至少与 Pylons 和 Genshi 兼容。预计不久更多模板 引擎和框架会采用 `__html__` 的概念。

Django 目前也支持 `__html__` 接口协议。其数据实体定义在 `django.utils.safestring.SafeData` 。源代码如下：

```python
class SafeData(object):
    def __html__(self):
        """
        Returns the html representation of a string for interoperability.

        This allows other template engines to understand Django's SafeData.
        """
        return self
```



### js 内嵌引入的正则替换

主要指的是 `pyecharts.utils.freeze_js` 的原理是先渲染生成 html 文件字符串，再使用正则替换，这在之前是没有问题，引入自定义模板后，模板文件也有可能引用其他文件（如 bootstrap.min.js），这样的话，碰到该行直接出现错误。

改进的办法是在渲染的过程就根据配置决定是否替换，因此该函数也可移除。

### 和 Flask 整合问题

> 此种方式是整合过程中产生一个代码版本，后来发现会破坏 Flask 原有的功能，因此改写为下一节的代码版本。但此种整合方式也是思考的一个过程，因此没有将此删除。

这是上周末刚刚完成的内容，解决在 Flask 框架中使用模板函数的问题。主要代码摘抄如下：


```python
from jinjia2 import Environment as BaseEnvironment

class Environment(BaseEnvironment):
    """Works like a regular Jinja2 environment but has some additional
    knowledge of how Flask's blueprint works so that it can prepend the
    name of the blueprint to referenced templates if necessary.
    """

    def __init__(self, app, **options):
        if 'loader' not in options:
            options['loader'] = app.create_global_jinja_loader()
        BaseEnvironment.__init__(self, **options)
        self.app = app
```

pyecharts 模板引擎

```python
from jinjia2 import Environment

class EchartsEnvironment(Environment):
    """Built-in jinja2 template engine for pyecharts

    """

    def __init__(self, pyecharts_config=None, *args, **kwargs):
        self._pyecharts_config = pyecharts_config or PyEchartsConfig()
        loader = kwargs.pop('loader', None)
        if loader is None:
            loader = FileSystemLoader(
                self._pyecharts_config.echarts_template_dir)
        super(EchartsEnvironment, self).__init__(
            keep_trailing_newline=True,
            trim_blocks=True,
            lstrip_blocks=True,
            loader=loader,
            *args,
            **kwargs)

        # Add PyEChartsConfig
        self.globals.update({
            'echarts_js_dependencies': echarts_js_dependencies,
            'echarts_js_dependencies_embed': echarts_js_dependencies_embed,
            'echarts_container': echarts_container,
            'echarts_js_content': echarts_js_content,
            'echarts_js_content_wrap': echarts_js_content_wrap
        })
```

代码解析要点如下：

- `Flask.Environment` 新增了两点扩展：
    - 增加了一个必要的 app 成员变量，这是一个 Flask 实例
    - 同时提供了默认的 loader 。
- `pyecharts.engine.EchartsEnvironment` 也有两点扩展：
    - 增加了一个可选 pyecharts_config 成员变量
    - 同时提供了默认的 loader。

整合的目标是实现一个类，使得同时具有以上四个特点。

主要整合方式：

第一种：Mixin 方式。这种方式是实现最为简单，但是在此种情况下却无法使用，这是因为二者都重写了 `__init__` ，都涉及到对象的创建过程，不建议使用。

第二种是代码混合方式：让一个直接继承 `jinja2.Environment` ，将另外一个的代码搬入。因为 `Flask.Environment ` 的代码比较少，继承 `EchartsEnvironment` 是更为优化的。

下面是使用第二种方式整合的最终代码及其使用方法：

```python
# ----- Adapter ---------
class FlaskEchartsEnvironment(EchartsEnvironment):
    def __init__(self, app, **kwargs):
        EchartsEnvironment.__init__(self, **kwargs)
        self.app = app


# ---User Code ----

class MyFlask(Flask):
    jinja_environment = FlaskEchartsEnvironment
    jinja_options = {'pyecharts_config': PyEchartsConfig(
        jshost='https://cdn.bootcss.com/echarts/3.7.2',
        echarts_template_dir='templates'
    )}


app = MyFlask(__name__)
```

因为 EchartsEnvironment 显式传入了 loader 参数，抵消了 Environment 类 loader 的重写逻辑。

目前该代码放在 demo 内，没有整合为 pyecharts 一部分。

### web框架整合优化

上一节实现有个重大问题，表面上看会覆盖原有模板目录功能，导致必须现实设置 `echarts_template_dir` 。因此必须继承  `flask.templating.Environment` 以保全全部功能。

```python
from __future__ import unicode_literals

import random
import datetime

from flask import Flask, render_template
from flask.templating import Environment

from pyecharts import HeatMap
from pyecharts.engine import PyEchartsConfigMixin, ECHAERTS_TEMPLATE_FUNCTIONS
from pyecharts.conf import PyEchartsConfig

class FlaskEchartsEnvironment(Environment, PyEchartsConfigMixin):
    pyecharts_config = PyEchartsConfig(
        jshost='https://cdn.bootcss.com/echarts/3.7.2'
    )

    def __init__(self, *args, **kwargs):
        super(FlaskEchartsEnvironment, self).__init__(*args, **kwargs)
        self.globals.update(ECHAERTS_TEMPLATE_FUNCTIONS)

class MyFlask(Flask):
    jinja_environment = FlaskEchartsEnvironment

app = MyFlask(__name__)
```

之后和标准的 Flask 项目一样使用。

### 命名借鉴

比如 `Page.from_charts` 借鉴了 `django.db.models.Manager.from_queryset` 。
又比如类 Mixin 模式变量方法的命名。

```python

clas DemoMixin(object):
    foo1 = None
    foo2 = None
    def get_foo1(self):
        return self.foo1
    def get_foo2(self):
        return self.foo2
```

## Python 2/3

### json.dumps 输出结果

```python
import json
c = {'date':'2017-01-01', 'a':'1'}
data = json.dumps(c, indent=0)
print(len(data))
print('*'.join(data))
```

上述代码在 2 和 3 环境下运行结果是不同的，结果如下：

环境：Python 3.6.3 (v3.6.3:2c5fed8, Oct  3 2017, 18:11:49) [MSC v.1900 64 bit (AMD64)]

```
34
{*
*"*d*a*t*e*"*:* *"*2*0*1*7*-*0*1*-*0*1*"*,*
*"*a*"*:* *"*1*"*
*}
```

环境：2.7.14 (v2.7.14:84471935ed, Sep 16 2017, 20:25:58) [MSC v.1500 64 bit (AMD64)]

```
35
{*
*"*d*a*t*e*"*:* *"*2*0*1*7*-*0*1*-*0*1*"*,* *
*"*a*"*:* *"*1*"*
*}
```

简而言之，将字典转化为json字符串时，python2 在每一对键值分割符“,”增加了一个空格

下面是测试 `pyecharts.utils.json_dumps` 功能的测试代码（使用 nosetests 框架）。影响到的是最后测试的时候直接使用表达式结果作为 assert 语句的第一个参数，这是一个取巧的方法，因为目前没有引入 `six` 等兼容库，代码需要多写。

```python
import json
from datetime import date, datetime

class UnknownTypeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, (datetime.datetime, datetime.date)):
            return obj.isoformat()
        else:
            try:
                return obj.astype(float).tolist()
            except Exception:
                try:
                    return obj.astype(str).tolist()
                except Exception:
                    return json.JSONEncoder.default(self, obj)


def json_dumps(data, indent=0):
    return json.dumps(data, indent=indent, cls=UnknownTypeEncoder)


def test_json_encoder():
    """
    Test json encoder.
    :return:
    """
    data = date(2017, 1, 1)
    eq_(json.dumps({'date': '2017-01-01', 'a': '1'}, indent=0), json_dumps({'date': data, 'a': '1'}))

    data2 = {'np_list': np.array(['a', 'b', 'c'])}
    data2_e = {'np_list': ['a', 'b', 'c']}
    eq_(json.dumps(data2_e, indent=0), json_dumps(data2))
```

上述测试代码是一个不好的实践方法，把测试目标改变了，上面测试的是 `data2_e` 和 `data2` 的 json 输出是否一致，而不是  `data2`  的 json 是否符合预期的 json 格式，这二者是截然不同的，显然我们要测试的是后者。

假设 `json.dumps` 输出不是符合标准的 json 数据，上述测试案例可以通过，但在之后的功能测试是不能通过的。

上述的测试代码已经蕴含了 `json.dumps` 一定能输出标准的 json 数据，这当然是。

按照测试原则，assert 语句的第一个应当是表征字面量，下面就是一个简单的对比。

```python
DEFAULT_HOST = 'https://chfw.github.io/jupyter-echarts/echarts'

def test_pyecharts_remote_jshost():
    target_config = PyEchartsConfig(jshost=DEFAULT_HOST)
    eq_('https://chfw.github.io/jupyter-echarts/echarts', target_config.jshost) # 良好的实践
    eq_(DEFAULT_HOST, target_config.jshost) # 糟糕的实践
```

### 函数不定参数定义与调用

在 Python3 中，函数定义时允许常规变量(regular argument)出现在一个不定参数(varargs argument)之后，如下面的函数。
```python
def sortwords(*wordlist, case_sensitive=False):
    ...
```
需要注意的是，调用的时候 case_sensitive 必须以关键字形式传入，类似 `sortwords('Apple', 'Orange', case_sensitive=True)` 。

更多的资料可以参考 [PEP 3102](https://www.python.org/dev/peps/pep-3102/) 。

之前在考虑 `page.from_charts(cls, *charts)` 是否添加 jshost 相关参数的时候碰到这个问题，最后考虑不添加这个特性，主要基于下面两个原因：

1 如果添加这个参数，会再调用时引起歧义，有以下两种种定义形式：

第一种： `Page.from_charts(jshost=None, *args)`

这种方式有个问题，就是即使 jshost 无意设置，也需使用 None 占位。

`Page.from_charts(chart1, chart2)` 调用从字面上是将两个图表合并，实际上只有一个，调用时会把 chart1 传给 jshost

第二种：`Page.from_charts(*args, jshost=None)`

这个将可选的参数放置在最后，可以解决 `Page.from_charts(chart1, chart2)` 字面和实际效果一致，但是仅Python3.5+支持

2 从功能上来看，该方法只是 `__init__` 方法的补充，不一定非要和其等价。

## 致谢

非常感谢 [@chenjiandongx](https://github.com/chenjiandongx) 和 [@chfw](https://github.com/chfw) 两位提供问题讨论和代码复查方面的经验。