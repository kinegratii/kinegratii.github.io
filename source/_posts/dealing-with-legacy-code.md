---
title: 处理和迁移遗留的代码
date: 2018-06-27 20:23:56
categories: 读书笔记
tags:
- Django
- 软件工程与实践
---

- **【书名】**：Django Design Patterns and Best Practices
- **【主题】**：Dealing with Legacy Code (处理遗留的代码)
- **【摘要】**：本文以 Django项目为例子，叙述在处理遗留代码过程中所使用的多项技术。阅读代码库通常是容易被低估的一个基本技能。在实践过程中，我们应当尽可能使用原有的可以运行良好的代码，而不是重复地发明轮子。另外测试案例也是代码编程一个非常重要的组成部分。

<!-- more -->

## A 概述

参加一个项目的开发是一个令人激动人心的事情，在此过程中你可以了解和使用到一些强大的工具和前沿的技术。然而常常接收的却是一个已经存在的，甚至远古时期的项目。当然，Django 发行仅有十几年（2005年），和其他语言框架相比，算是比较年轻的项目。由于技术的迅速的发展，使用早期 Django 项目是一个巨大的挑战，仅仅使用源码和文档可能是不够的。

和项目维护相比，项目迁移更多涉及到是系统设计层面的内容，比如：

- 使用新版本的语言，比如 从 Python2 迁移到 Python3
- 使用新版本的库以具有更好的性能表现
- 使用 CBV（类视图） 改写项目中的 CBV （函数视图）
- 使用 django-rest-framework 实现项目的 Restful API

## B Django 版本

确定项目使用了哪个版本的 Django 是一个重要的基础步骤。随着 Django 不断发展和完善，一些概念、规范和规则可能有所变化，比如  *默认目录结构* 和 *最佳实践规则* 。

### 依赖文件

一般来说，每个项目的根目录会有一个 *requirements.txt* 或者 *setup.py* 的文件，表明了这个项目使用了哪些第三方库，它们的版本是多少，通常是类似下面格式的配置文件：

```
django=1.11.2
```

使用 “=” 描述的依赖，是一个非常精确的版本。与此相对应的是类似 `django>=1.8` 的形式 ，表示是一个范围， pip 安装时总是以符合该范围的最新版本为准。

> **延伸** 固定依赖库的版本是一个最佳实践，这样会减少一些意想不到的错误，使得构建过程具有确定性。[pipenv](https://docs.pipenv.org/) 是一个 Python 项目开发构建工具，为项目提供了单独的虚拟环境，使用  Pipfile.lock 文件产生确定性的构建过程。

在现实的项目则往往是没有 *requirements.txt* 文件或 *setup.py* 文件，这时候你可能需要尝试其他方式来确定它的 Django 版本。

### 虚拟环境

在大多情况下，Django 项目可能部署在一个虚拟环境或者携带有构建环境，事实上，这种做法也是一个最佳实践额规则。当你位于一个虚拟环境时，可以按照下面的步骤查询 Django 版本。

第一步，激活该环境，在 Linux 中可以使用以下命令：

```shell
$ source venv_path/bin/activate
```

第二步，使用下面的 Python 片段，查询版本。

```shell
>>> import django
>>> django.get_version()
'1.11.1'
```

当然除了上述的方法外，也可以直接在 Django 源码查看版本。

```shell
$ cd envs/foo_env/lib/python2.7/site-packages/django
$ cat __init__.py
VERSION = (1, 11, 1, 'final', 0)
```

### 配置模块与Django版本特性

项目的配置模块是一个项目的重要的模块，它包含了该项目的基本信息和所使用的一些组件、构件、外部资源等。一般来说，配置模块名称为 *settings.py*。有时根据不同的环境，使用不同的配置，比如一个符合最佳实践的文件布局可能如下：

```
Fooo/
    Foo/
        __init__.py
        settings/
            __init__.py
            base.py
            dev.py
            prod.py
            test.py
        urls.py
        wsgi.py
```

使用 `django-admin startproject` 命令生成的项目，在 `settings.py` 的顶部含有类似下面的注释文字，清楚地记载了该项目所使用的具体版本。

```
Django settings for bws project.

Generated by 'django-admin startproject' using Django 1.8.3.
```

当然这只是表明项目创建时的版本，如果在开发过程中，项目有所更新，也可以通过 git 日志来获取版本的变更情况。

另外的，如果上述的注释被删除的话，需要根据项目的一些特性并结合版本发布日志可以大致确定版本范围。

下面是一些版本中比较大的变化：

| 版本 | 特性更新 |
| ------- | ------ |
| 1.4 LTS | 新的默认项目文件布局 |
| 1.5 | `AUTH_PROFILE_MODULE` 被废弃，使用 `AUTH_USER_MODEL` |
| 1.7 | 支持数据库迁移，可以使用 `migrate` 命令 |
| 1.8 LTS | 开始支持 Jinja2 模板引擎。|
| 2.0 | 不再支持 Python2 |

Django 的版本发布采用语义化的规范，特性发布以 *X.Y* 的格式，具体可查看 [The Release proccess](https://docs.djangoproject.com/en/2.0/internals/release-process/)。


## C 阅读代码

### 源文件

和 PHP 不同的是， Python 项目源代码文件不是位于 web 服务器的文档根目录，比如 *wwwroot* 或 *public_html* 。将源代码和配置数据放置在公共访问目录之外，web 爬虫程序就无法访问这些文件，提高了安全性。

### 入口点 —— 路由

通常来说，Python 程序的入口点类似于下面的代码段：

```python

if __name__ == '__main__':
    pass
```

Django 使用的是 “模型 - 模板 - 视图” 的架构模式，简称为 MTV ，是 MVC 的一种变形。在项目中，路由 是一个很好的入口点，包含了从每个请求到响应的映射。代码上可能为一个模块文件(Module)，复杂的情况下也可能是一个包(Package)。但无论何种情况，总是存在 *根路由* ，并且定义在项目配置模块中。

```python
ROOT_URLCONF = 'projectname.urls'
```

在 Linux 中，可以使用下面的命令寻找和定位 根路由：

```shell
$ find . -iname settings.py -exec grep -H 'ROOT_URLCONF' {} \;
./projectname/settings.py:ROOT_URLCONF = 'projectname.urls'
$ ls projectname/urls.py
projectname/urls.py
```

### 文档

**代码本身以及测试案例是了解代码的最好的方法。** 对于遗留的代码，旧有的文档可能会与之相匹配，但有时也无法覆盖未来的问题。

Django 的官方文档位于 https://www.djangoproject.com 。在右下角可以切换到不同的版本。

### 应用逻辑与可视化

图表是了解一个了解程序最好的方法。

[django-command-extensions](https://github.com/django-extensions/django-extensions) 是一个工具扩展包，提供了 *graph_models* 命令，可以用图表的方式显示所有数据模型及其之间的关系。

```shell
$ python manage.py graph_models -a -o myapp_models.png
```

在 PyCharm IDE 中也有类似的功能。右键选择模型文件 *models.py*，依次选择下面的菜单项。

```
Diagrams -> Show Diagram -> Django Model Dependency Diagram
```

需要注意的是，图中的 `str` 是 `settings.AUTH_USER_MODEL` 指向的模型类。

显示的图表如下：

![bws-diagram](/images/bws-diagram.png)

你还可以导出 uml 和 png 等不同的文件格式。

## D 增量变化 OR 全新重写

> 代码是一个债务，而不是一个资产，更少的代码具有更好的可维护性。

在最好的情况下，遗留的代码，总是我们所期望的：

- 拥有测试代码
- 良好的文档
- 平稳地迁移新环境

在最坏的情况下，可能更希望放弃现有的代码，并且开始重新开始新的编码工作。

在具体工作的过程中，应当根据具体的情况，比如项目进度、时间要求、业务逻辑等不同因素采取不同的策略。

有时候，在重写的过程中，应用（业务）领域的复杂度将是一个巨大的障碍。这是因为在编写旧有代码过程中所学习的大量的知识可能会遗失。特别的是，代码逻辑设计差以至于很难从中了解具体的业务逻辑。

最差的重写是转换，比如将原来的 PHP 项目使用 Python 语言重写，在此过程中很可能会丢失原有 PHP 语言的一些最佳实践规则。

## E 特征测试

> 编写测试案例应当在作任何改变之前。

使用测试，我们可以非常容易和正确地修改已有代码的行为。没有测试，则不可能知道我们的改变让代码变得更好还是更差。

特征测试描述了系统当前的实际行为。这是使用的一些步骤：

1. 在测试用具中使用目标代码快
2. 编写一个你会知道失败的断言
3. 从失败的断言中知道代码的行为
4. 修改测试，让它预期目标代码块的实际行为
5. 重复上述步骤

当准备在遗留系统中使用一个方法之前，请查看一下是否已有针对它的测试，没有的话就自己写一个，始终保持这一习惯，你的测试就能起到信息传递媒介的作用。别人只要一看到你的测试就能够知道对于某方法他们该期望什么而不该期望什么，试图使一个类变得可测试这一行为本身往往能够改善代码的质量，于是人们能够发现什么是可行的，以及是如何可行的，他们可以进行修改，更正BUG，然后继续前进。

## F 数据库


在 Django 中，可以使用下面的命令导入旧有的数据库结构。

```shell
$ python manage.py inspectdb > models.py
```

以上命令会根据配置模块的配置内容生成 Python 代码，这些代码通常位于 *models.py* 文件。

这里有一些最佳实践规则：

- 了解 Django ORM 的一些限制，比如不支持多字段主键、NoSQL数据库等。
- 清理代码，比如移除不必要的 ID 字段，Django 将会自动创建。
- 外键处理，在某些数据库中，原始生成的外键可能是使用整数类型定义的。
- 模块划分，将其分散在不同的 APP 以符合具体的业务逻辑
- 在迁移的过程中，会创建额外的数据表，比如 django_* 等数据表

## G 参考资料

- [Upgrading Django to a newer version](https://docs.djangoproject.com/en/2.0/howto/upgrade-version/)
- [Recommended Django Project Layout](https://www.revsys.com/tidbits/recommended-django-project-layout/)
- [Working Effectively With Characterization Tests](https://www.artima.com/weblogs/viewpost.jsp?thread=198296)
- 《修改代码的艺术》