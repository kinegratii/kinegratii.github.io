---
title: pyinstaller打包教程
date: 2018-09-12 21:19:21
categories: 技术研究
tags:
---

https://github.com/kingratii/pyinstaller-guide

## 项目准备

### 开发流程

[poetry](https://poetry.eustace.io) 是一个 Python 的 *依赖管理* 和 *打包构建* 的工具，可以看作是 pip 和 twine 的集成。 

- 为每个项目创建单独的环境
- 使用 *pyproject.toml* 和 *pyproject.lock* 文件，这两个文件均应当提交到版本库

在项目开发流程中， poetry 提供了以下命令：

- init ： 初始化项目基本信息，支持交互式填写
- add/remove/update ：添加、移除、更新一个或多个依赖库
- run ：使用独立的环境运行脚本



```
poetry init
poetry add pyecharts
poetry add -D pyinstaller
poetry run
```

添加 pyecharts 为依赖，

```
F:\ArgonApp>poetry add pyecharts
Creating virtualenv argonapp-py3.6 in C:\Users\zhenwei.yan\AppData\Local\pypoetry\Cache\virtualenvs
Using version ^0.5.11 for pyecharts

Updating dependencies
Resolving dependencies...


Package operations: 12 installs, 0 updates, 0 removals

Writing lock file

  - Installing dukpy (0.2.2)
  - Installing macropy3 (1.1.0b2)
  - Installing javascripthon (0.10)
  - Installing lml (0.0.2)
  - Installing markupsafe (1.0)
  - Installing pyecharts-jupyter-installer (0.0.3)
  - Installing future (0.16.0)
  - Installing jinja2 (2.10)
  - Installing jupyter-echarts-pypkg (0.1.2)
  - Installing pillow (5.2.0)
  - Installing pyecharts-javascripthon (0.0.6)
  - Installing pyecharts (0.5.11)
  
```





## 编写应用程序





基于 pyecharts + tkinter 

pyinstaller 3.4





## PyInstaller 打包



必须安装 pywin32-ctypes



```
poetry run pyinstaller app.py -D --clean
```





打包的几个难点：

- 第三方库打包，特别是含有 C 扩展的，比如 PIL 。



具体打包过程



- pyecharts 库是否使用 `sys._MES`
- argon 程序路径处理
- 是否文件夹模式