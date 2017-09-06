---
title: Django-Echarts系列：js依赖文件管理
date: 2017-09-06 11:35:47
categories: 技术研究
tags:
- 数据可视化
---

[django-echarts](https://github.com/kinegratii/django-echarts) 是本人正在开发的一个开源项目，该项目旨在将 [pyecharts](https://github.com/chenjiandongx/pyecharts) 库整合到Django web框架中，从而形成echarts-python-django 大整合的项目。

继之前简单的一个使用示例之后，最近加班加点完成了其中的一个部分：js依赖文件管理。

<!-- more -->

## 引入

要完成一个Echarts图表，从最后渲染完成的HTML结构来看，至少应当包含以下三个部分：

- 图表容器控件，比如`<div></div>`
- js依赖文件，比如 `<script src='/static/echarts/echarts.min.js'></script>`
- 图表初始化代码,代码中 `myCharts.setOptions(Foo)` 所在的script标签。

需要说明以上排序按照一般出现的顺序，将所有 script 标签放置在body标签的最后，有利于页面的加载。

### 目标

三者之前没有太大的耦合性，是可以单独拿出来讨论其设计思想和实现方式的。js 依赖文件管理最终的目标是**构建js文件路径字符串**。

```html
<script src='/static/echarts/echarts.min.js'></script>
<script src='/static/echarts/map/china.js'></script>
```

>  虽然最后生成的是一个或多个script标签，但依据 Django MTV 原则，标签的构建应当由模板系统负责。

核心的src属性由路径和js文件名两部分组成。路径的意义在于，对于同一个 echarts.min.js 可以由不同的地方提供，这称之为repository。而文件名是由用户输入提供的。使用代码表示如下：

```python
def generte_js_link(js_name):
    host = '' # TODO Where to pick a host according the settings
    return '{host}/{js_name}.js'.format(host=host, js_name=js_name)
```

### 问题

综上所述，js 依赖文件管理解决的问题：

- 需要管理哪些依赖文件
- 哪些repository可以提供js文件，其中哪些可以实现对其的支持。
- 如何在不同repository之间尽可能平稳的切换，即它们之间必须提供统一的API
- 需要对外提供哪些API，即在哪些情况下可能使用到这个功能

## 仓库(repository)与文件

按照正常逻辑，文件名由用户根据实际功能需求指定，其有效性应当交由用户确保。在实际过程不同repository可提供的文件是不一样。js文件分为核心库文件和地图数据文件两种。pyecharts能够提供本地和远程两种类型的repository，加上Django整合时，项目静态文件也可以作为一种repository存在，因此共有三种。

将repository和文件类型进行交叉分析，可整理出以下的一张表格：

| 仓库          | 核心库文件 | 地图数据文件 | 核心库版本支持 | 远程/ 本地 |
| ----------- | ----- | ------ | ------- | ------ |
| pyecharts本地 | 可提供   | 可提供    | 无，不可控   | 本地     |
| pyecharts远程 | 可提供   | 可提供    | 无，不可控   | 远程     |
| 官方CDN       | 不提供   | 提供     | -       | 远程     |
| 公共CDN       | 可提供   | 不提供    | 支持      | 远程     |
| 项目静态目录      | 可自定义  | 可自定义   | 可自定义    | 本地     |

分析如下：

- 首先的是pyecharts本地作为存储仓库是不太合适，同为本地文件存储，Django项目静态库显然是一个更为合适的选择。
- pyecharts远程库，路径为 https://chfw.github.io/jupyter-echarts/echarts， 该库的优势在于提供了一些列的自定义地图，但弱势是核心库文件没有版本管理。
- 官方CDN：其实指的是地图文件数据下载的源地址。
- 公共CDN：优势在于支持版本管理，缺点是不提供地图数据文件。公共CDN仅选择[Echarts官方教程](http://echarts.baidu.com/tutorial.html)提及的三个CDN，其余的已经很久没有更新了。
- 项目静态目录：通常由 `setttings.STATIC`指定。 这是开发者自己从零开始构建的，因此自由度最大。为了方便，可开发从其他远程仓库下载文件的功能。

一个通常的使用场景如下：

- 试验django-echarts，仅使用远程仓库，这样不必配置静态文件设置等。
- 如果可用，通过下载工具下载到本地，并进行一系列开发。
- 部署上线时，根据需要切换到CDN。

## 配置

### 基本配置

配置是项目初始化需要使用的。默认的配置如下：

```python
DEFAULT_SETTINGS = {
    'echarts_version': '3.7.0',
    'lib_js_host': 'bootcdn',
    'map_js_host': 'echarts',
    'local_host': None
}
```

在实际运行之前，会将用户自定义配置和默认配置进行合并，并向外提供统一的模块变量用于访问。

> 关于这一部分可以期待之后的《django-echarts系列：配置模块》一文。

由于核心库文件和地图数据文件需要分开托管，因此需要使用两个变量分别指定和设置。二者都有自己有效的可选值。

### 远程/本地切换

 `local_host`的作用有两点：

- 提供公用变量，当 `lib_js_host` 和 `map_js_host` 同时制定本地仓库时，可以借助该变量
- 下载工具的目标目录。

## 运行分析

### 分开托管和文件识别

由于不同仓库提供的文件不同，通常分为可提供核心库文件和地图数据文件，因此需要分别两个查询表（在Python使用一个dict表示即可）。

这就带来了一个问题：用户输入的是 js_name，只有这个参数，而且由于自定义地图文件，理论上可以是任何一个有效的文件名字符串，如何识别为核心库文件还是地图数据文件，即从哪个字典查询，成了一个待解决的问题。

这个没有一个百分百正确的答案。目前采用一个简单办法：由于公共CDN提供的核心库文件是一定的，可提供一个核心库文件列表，判断js_name是否在其中即可。

### 输出URL

当确定完某一个仓库后，之后的url构建就比较简单了，本质上来说是python string format的一些封装。仓库路径具体和一些因素有关，这些因素都需要在项目初始化就已经确定了，放在 settings 模块是最为合适了。目前支持以下字段：

- echarts_version：版本字符串，一些公共CDN需要指定版本号。
- STATIC_URL：静态文件目录，通常用于项目本地仓库，该值等于`settings.STATIC_URL`.

在实现过程中，也有两点问题需要注意：

- 字段的大小写问题，为了和`settings.STATIC_URL`一致，也采用了大写变量
- 目录后缀`/` ,依据Django规范 STATIC_URL是带有/的，而pyecharts的远程路径没有带有/，以及自己设置的第三方CDN中，这个问题需要作统一处理。目前是按照pyecharts没有带有/，不排除之后会依照Django规范，但是对外部使用是没有任何影响的。

基本代码

```python

class Host(object):
    HOST_LOOKUP = {}

    def __init__(self, name_or_host, context=None, host_lookup=None, **kwargs):
        context = context or {}
        host_lookup = host_lookup or self.HOST_LOOKUP
        host = host_lookup.get(name_or_host, name_or_host)
        try:
            self._host = host.format(**context).rstrip('/')
        except KeyError as e:
            self._host = None
            raise KeyError('The "{0}" value is not applied for the host.'.format(*e.args))

    @property
    def host_url(self):
        return self._host

    def generate_js_link(self, js_name):
        return '{0}/{1}.js'.format(self._host, js_name)
```

### 渲染html

这一过程是Django模板系统负责的，为了方面可以自定义一个模板标签echarts_js_dependencies解决这个问题。

以下是基本代码

```python
@register.simple_tag(takes_context=True)
def echarts_js_dependencies(context, *args):
    links = []
    for option_or_name in args:
        if isinstance(option_or_name, Base):
            for js_name in option_or_name.get_js_dependencies():
                if js_name not in links:
                    links.append(js_name)
        elif isinstance(option_or_name, six.text_type):
            if option_or_name not in links:
                links.append(option_or_name)
    links = map(DJANGO_ECHARTS_SETTING.host_store.generate_js_link, links)

    return template.Template('<br/>'.join(['<script src="{link}"></script>'.format(link=l) for l in links])).render(
        context)
```

有几个注意点：

- 标签是支持多个script渲染的。
- 为了方便，列表的每一项支持 文件名或者 `pyecharts.base.Base`对象。
- 在多个标签输出时，需要去掉那些重复的文件。
- 因为html结构简单，所以使用`register.simple_tag` 就可以了。

## 文件下载功能

下载工具提供将远程的js文件同步到本地静态文件目录中。该功能为manage命令。在配置方面（源目录、目标目录）仅支持 `DJANGO_ECHARTS` ，暂时还不支持命令行参数传入。这是下一阶段的重点内容。

## 展望

 本项目是基于 pyecharts 而发展的。[pyecharts](https://github.com/chenjiandongx/pyecharts)是一个非常棒的项目，解决在Python中使用echarts的问题，加强了Python在数据可视化方面的应用。

另一方面pyecharts刚刚面世两三个，目前着重于Echarts实例创建这一问题上，对于外围环境的问题涉及有所不足。

本项目的另外一个目的就是借由django-echarts的开发推动pyecharts的蓬勃发展。