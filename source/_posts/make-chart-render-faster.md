---
title: pyecharts：改善渲染效率
categories: 技术研究
tags:
  - 数据可视化
date: 2018-03-03 23:34:08
---

> 本文已收录于 [《pyecharts 开发专辑》](/pyecharts-project/) 。

本文所述的基于特性 v0.3.3 版本，预计于 v0.3.4 / v0.4.0 加入。

<!-- more -->

## 问题

一直以来， pyecharts 都是推荐使用下面的方式多次渲染图表。

*代码 1*

```python
from pyecharts import Bar, Line

bar = Bar("直方图示例")
bar.add()
bar.render(path='bar.html')

line = Line("测试图表")
line.add()
bar.render(path='line.html')
```

但历经多次版本迭代后，该代码效率有所降低。 `chart.render` 函数调用了 `engine.render` 函数，该函数源码如下：

*代码 2*

```python
def render(template_file, notebook=False, **context):
    config = conf.CURRENT_CONFIG
    echarts_env = EchartsEnvironment(
        pyecharts_config=config,
        loader=FileSystemLoader(
            [config.echarts_template_dir, conf.DEFAULT_TEMPLATE_DIR])
    )
    template = echarts_env.get_template(template_file)
    return template.render(**context)
```

可以看出，每调用一次都会重新创建一个 `EchartsEnvironment` 实例，问题在于 `EchartsEnvironment` 引擎对象并不是一个简单的数据存储型对象，每次对象的创建都需要花费大量时间，特别是寻找和编译模板文件，而且在对象创建后就无法修改。

在上述使用示例中，二者的所使用的引擎对象配置是一样的但需要创建两次，无形中降低了执行效率。

## 使用底层API

基于第一节所述的问题，改成只创建一次引擎对象，基本代码如下：

*代码 3*

```python
from pyecharts import Bar, Line
from pyecharts.conf import PyEchartsConfig
from pyecharts.engine import EchartsEnvironment
from pyecharts.utils import write_utf8_html_file

bar = Bar("直方图示例")
bar.add()

line = Line("测试图表")
line.add()

config = PyEchartsConfig()
env = EchartsEnvironment(pyecharts_config=config)
tpl = env.get_template('simple_chart.html')

html = tpl.render(chart=bar)
write_utf8_html_file('bar.html', html)

html = tpl.render(chart=line)
write_utf8_html_file('line.html', html)
```

显然和第一节相比，该代码使用了更为底层的 API，虽然符合一般的渲染过程 （参见 [文档-图表api](http://pyecharts.org/#/zh-cn/api?id=%e6%80%bb%e4%bd%93%e6%b5%81%e7%a8%8b)），但是使用者需要了解一些更为底层的内部逻辑，方便性有所下降。

- 比如使用了`write_utf8_html_file` 这个工具方法
- 需要使用者显式指定 `simple_chart.html` 模板文件，之前是作为默认参数

## 改进方案（一）

第一种改进是基于第一份代码，直接使用  “Lazy Object” 方式，实现只有一个 `EchartsEnvironment`  的单例对象，关于具体细节可以查看之前的 [PR 240](https://github.com/pyecharts/pyecharts/pull/240)  。

## 改进方案（二）

第二种方案是基于第二份代码，并对此作一些封装。实现要点在于：

-  `EchartsEnvironment`   增加 `render_chart` 方法，参数与 现有的 `chart.render` 基本上相同
- `Chart.render` 直接调用 `EchartsEnvironment.render_chart` 方法
- 原有的 `engine.render` 和 `engine.render_notebook` 合并，等到 notebook 整合后直接移除，实际上已经向下移到 `EchartsEnvironment` 类了

实例代码

*代码 4*

```python
class EchartsEnvironment(BaseEnvironment):
    """
    Built-in jinja2 template engine for pyecharts
    """

    def __init__(self, pyecharts_config=None, *args, **kwargs):
        pyecharts_config = pyecharts_config or conf.PyEchartsConfig()
        loader = kwargs.pop('loader', None)
        if loader is None:
            loader = FileSystemLoader(pyecharts_config.echarts_template_dir)
        super(EchartsEnvironment, self).__init__(
            pyecharts_config=pyecharts_config,
            keep_trailing_newline=True,
            trim_blocks=True,
            lstrip_blocks=True,
            loader=loader,
            *args,
            **kwargs)
    def render_chart_to_file(
        self,
        chart,
        object_name='chart',
        path='render.html',
        template_name='simple_chart.html',
        extra_context=None
    ):
        context = {object_name: chart}
        context.update(extra_context or {})
        tpl = self.get_template(template_name)
        html = tpl.render(**context)
        write_utf8_html_file(path, html)

    def render_chart_to_notebook(self):
        pass

```

单个图表实现方式，`Base.render` 直接调用 `env.render_chart_to_file` 函数。

*代码 5*

```python
class Base:
    def render(self,
               path='render.html',
               template_name='simple_chart.html',
               object_name='chart',
               extra_context=None):
        config = conf.CURRENT_CONFIG
        echarts_env = EchartsEnvironment(
            pyecharts_config=config,
            loader=FileSystemLoader(
                [config.echarts_template_dir, conf.DEFAULT_TEMPLATE_DIR])
        )
        echarts_env.render_chart(
            chart=self,
            object_name=object_name,
            path=path,
            template_name=template_name,
            extra_context=extra_context
        )
```

多次渲染图表可以使用以下的代码，变得更为简洁，明朗。

*代码 6*

```python
from pyecharts import Bar, Line
from pyecharts.conf import PyEchartsConfig
from pyecharts.engine import EchartsEnvironment
from pyecharts.utils import write_utf8_html_file

bar = Bar("直方图示例")
bar.add()

line = Line("测试图表")
line.add()

config = PyEchartsConfig()
env = EchartsEnvironment(pyecharts_config=config)

env.render_chart_to_file(bar, path='bar.html')
env.render_chart_to_file(line, path='line.html')
```

这种方案最大的优点，**将 `render` 渲染这个操作变成了是引擎对象提供的方法**，这一点符合面向对象的实际情形，也和 jinja2 ，django 等模板系统保持一致，而原来的 `chart.render` 就变成了一个 shortcut 方法，继续可以使用。

## 与 Notebook 渲染的整合

pyecharts 的 issue [#412](https://github.com/pyecharts/pyecharts/issues/412) 也提出了关于 Notebook 使用场景下，引擎对象对此创建的问题，和其他一些问题。

主要解决方案如下：

第一步，整合模板，将所有的模板代码使用 *notebook.html* 存储，至于其中一些代码片段，借助已经实现的模板函数，将一些数据逻辑处理移到模板函数内执行，这样就能和本地渲染共用一些代码。

第二步，减少模板文件的 context 字典，将图表变量（chart 或 page）使用固定参数，其他使用 `**kwargs` 方式传入。例外的是 require_config ，该内部实现相对比较复杂，暂时没有进行改动，只是传入形式由：`**require_config` 改为更加明确的 `config_items=config_items,libraries=libraries` 。

为了提供方面，engine 提供了一个创建基本引擎对象的方法，纯 Python 和 notebook 两种场景下均可使用。

*代码 7*

```python
def create_default_environment():
    """
    Create environment object with pyecharts default single PyEchartsConfig.
    :return:
    """
    config = conf.CURRENT_CONFIG
    echarts_env = EchartsEnvironment(
        pyecharts_config=config,
        loader=FileSystemLoader(
            [config.echarts_template_dir, conf.DEFAULT_TEMPLATE_DIR])
    )
    return echarts_env
```

针对上述，继续使用 `create_default_environment` 优化为以下的代码：

*代码 8*

```python
from pyecharts import Bar, Line
from pyecharts.engine import create_default_environment

bar = Bar("直方图示例")
bar.add()

line = Line("测试图表")
line.add()

env = create_default_environment()

env.render_chart_to_file(bar, path='bar.html')
env.render_chart_to_file(line, path='line.html')
```

改进过程：代码1/ 代码2 -> 代码6 -> 代码8

## 更新日志

### 面向开发者

- 整合模板存储，html 文件只保留完整可用的模板代码，小片段代码全部使用 Python 代码实现，参见 `engine` 模块
- 移除 `engine.render` 和 `engine.render_notebook`  方法
- Notebook 相关测试案例移到新模块

### 面向使用者

- 引擎类增加快捷的渲染方法 `render_chart_to_file`
- 增加创建默认引擎类方法 `pyecharts.engine.create_default_environment`
- 重构渲染内部实现，改善效率
- 修正图表 width/height 在 Noteboo 渲染的 bug

## 总结

如果多次渲染图表，应当使用更为底层的 `render_chart` 方法， 这是需要坚持的一点。使用 `chart.render` 方式依旧无法解决多次创建引擎对象的问题，关于这一点之前已有多次讨论。
