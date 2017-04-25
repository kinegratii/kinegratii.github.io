---
title: python项目持续集成与包发布
date: 2017-04-25 18:11:18
categories: 编程
tags:
- Python
- PyPI
- 测试
- 构建
- 持续集成
---

本文以 [ConfStruct](https://github.com/kinegratii/ConfStruct) 为例子描述了Python项目的持续集成和包发布等开发流程，并了解 Travis CI、wheel和twine等工具的使用。

## 1 准备工作

首先准备好项目代码和文本，包括：

- 源代码
- 完全通过的测试用例代码
- README文件
- 开源协议文件
- requirements.txt依赖文件
- .gitignore文件

<!-- more -->

## 2 运行本地单元测试

依据开发规范，测试代码放置在tests包下，每个测试文件以test_开头。测试框架这里使用的是Python自带的unittest（实际是后面pytest和Travis CI集成的出现了问题，所以改用unittest）。执行以下命令。

```
python -m unittest
```

结果如下，完全测试通过。

```
E:\projects\ConfStruct>python -m unittest
.......
----------------------------------------------------------------------
Ran 7 tests in 0.002s

OK
```

## 3 Travis持续集成

Travis是一个在线持续集成的平台，支持github登录。

第一步，在项目根目录下创建 .travis.yaml文件，写入相关配置。

```yaml
language: python
python:
  - "2.7"
  - "3.4"
  - "3.5"
  - "3.6"
install:
  - pip install -r requirements_dev.txt
script: python -m unittest
```

上述内容项目需在python2.7和python3.4+环境下使用unittest测试。

第二步，在github创建一个空项目，并使用github登录Travis,打开自动构建项目开关，每当有新的push或者PR时就会自动触发，并给出是否构建成功的消息。

第三步，使用git将本地代码上传到github，过一两分钟后可在Travis查看相关构建信息。下面是构建成功的结果。

![travis-build-success](/images/travis-build-success.png)


## 4 编写setup.py文件

setup.py文件描述了项目的一些基本信息，定义在 setuptools或 distutils.core 的 setup 函数参数。主要有

**name**

名称，PyPI的唯一标识，不能与已有的冲突。

**version**

版本字符串，可以使用常量或者从库的 `__VERSION__` 导入。

**author / author_email**

作者名称和邮箱

**url**

项目主页，可以使用github项目页面。

**py_modules**

项目代码模块

**install_requires**

安装依赖，格式与requirements.txt相同

**license**

开源协议类型，如 MIT

**lib_classifiers**

分类标签，设置Python版本支持、操作系统支持、面向开发者或使用者、软件分类等信息。

## 5 生成wheel安装包

wheel实际上是一个zip压缩包，是Python最新推荐的包格式。在生成之前需要安装相关包。

```
pip install wheel
```

成功后执行命令

```
python setup.py bdist_wheel --universal
```

在dist目录下多了个 ConfStruct-0.5.0-py2.py3-none-any.whl 安装包，使用pip install 命令即可安装成功。在生成命令中 `--universal` 表示python2和pytho3公用一个安装包，这和安装包后缀py2.py3-none-any相对应。当然其他还可以python2和3使用不同的安装包。

## 6 发布到PyPI

执行 `pip install twine` 安装twine。执行以下命令上传文件，在此过程中可能需要输入PyPI的用户名和密码。

```
twine upload dist/*
```

最新版本的twine无需注册这一步骤，执行命令 `twine register` 将显示 `HTTPError: 410 Client Error: This API is no longer supported, instead simply upload the file. for url: https://upload.pypi.org/legacy/`的错误。

## 7 添加徽章

在 https://badge.fury.io 中输入项目名称并查找，把markdown格式复制到README.md文件。

点击Travis控制台build pass 图片并复制图片链接到README.md。如

```
![travis](https://travis-ci.org/kinegratii/ConfStruct.svg?branch=master)
[![PyPI version](https://badge.fury.io/py/ConfStruct.svg)](https://badge.fury.io/py/ConfStruct)
```

效果图

![travis](https://travis-ci.org/kinegratii/ConfStruct.svg?branch=master)
![PyPI version](https://badge.fury.io/py/ConfStruct.svg)
