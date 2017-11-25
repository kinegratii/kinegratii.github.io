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

### 持续开发

这里的持续性开发指的的公共API的稳定性，更为确切的说是废弃策略。随着项目的不断推进，新代码不断加入，旧代码不断淘汰。但由于开源项目的公开性和考虑其稳定性，无用的代码并总是立即被删除，而是经过一段时间后再删除，在这方面，个人 Django 项目做的比较好，将旧有代码按照淘汰进程分几个等级([链接](https://docs.djangoproject.com/en/1.11/internals/release-process/#deprecation-policy)) ，我自己在此基础上增加了一个等级：Not Recommend ，通常用于重大变更，涉及到核心代码

- 不再推荐使用(Not Recommend)：仅在更新日志和文档中表明
- 废弃(Deprecated)：使用 `warnings` 模块表明
- 移除(Removed)：删除相关代码

## 功能设计与实现

### js 内嵌引入的正则替换

主要指的是 `pyecharts.utils.freeze_js` 的原理是先渲染生成 html 文件字符串，再使用正则替换，这在之前是没有问题，引入自定义模板后，模板文件也有可能引用其他文件（如 bootstrap.min.js），这样的话，碰到该行直接出现错误。

改进的办法是在渲染的过程就根据配置决定是否替换，因此该函数也可移除。

### 和 Flask 整合问题

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

### json.dumps 方法

简而言之，将字典转化为json字符串时，python3 在每一对键值分割符“,”增加了一个空格，看下面的代码：

```shell
> python
Python 3.5.2 (v3.5.2:4def2a2901a5, Jun 25 2016, 22:18:55) [MSC v.1900 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import json
>>> c = {'date':'2017-01-01', 'a':'1'}
>>> len(json.dumps(c, indent=0))
34
>>>


zhenwei.yan@YZHW-PC C:\Users\zhenwei.yan
> python2
Python 2.7.13 (v2.7.13:a06454b1afa1, Dec 17 2016, 20:53:40) [MSC v.1500 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import json
>>> c = {'date':'2017-01-01', 'a':'1'}
>>> len(json.dumps(c, indent=0))
35
>>>
```

影响到的是最后测试的时候直接使用表达式结果作为 assert 语句的第一个参数，这是一个取巧的方法，因为目前没有引入 `six` 等兼容库，代码需要多写。

```python
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


按照测试原则，assert 语句的第一个应当是表征字面量。

```python
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