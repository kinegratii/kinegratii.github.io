---
title: python-wiki
date: 2017-04-28 20:23:57
---

## 在线电子书

> 本节搜集了那些有关于Python方面的电子书，你可以直接点击链接进行在线阅读。

- Python 入门指南 — Python tutorial 3.6.0 documentation
  http://www.pythondoc.com/pythontutorial3/index.html

- python-3-patterns-idioms
  https://bitbucket.org/MatToufoutu/python-3-patterns-idioms

- Python 3 Module of the Week — PyMOTW 3
  https://pymotw.com/3/

- Python Cookbook 3rd Edition Documentation — python3-cookbook 2.0.0 文档
  http://python3-cookbook.readthedocs.io/zh_CN/latest/index.html

- The Hitchhiker’s Guide to Python!
  http://docs.python-guide.org/en/latest/

- Full Stack Python
  https://www.fullstackpython.com/table-of-contents.html

- 神经网络与深度学习

  https://nndl.github.io/

- Python 和数据科学
  http://bookdata.readthedocs.io/en/latest/

- Python机器学习
  https://ljalphabeta.gitbooks.io/python-/content/

## Python库列表

> 这些都是我自己用过或者觉得非常实用的一些Python库，包括标准库和第三方库。本列表基于 [awesome-python](https://github.com/vinta/awesome-python)。

- [Six](https://pythonhosted.org/six/) - Python2和3的兼容库。
- [Django](https://www.djangoproject.com) - 全栈式web开发框架，大而全，什么都有。
- [Django Rest Framework](https://github.com/encode/django-rest-framework) - Django的Restful框架。
- [django-tenant-schemas](https://github.com/bernardopires/django-tenant-schemas) - 通过PostgreSQL Schemas在Django中实现多租户特性。当初用的时候Django1.7刚刚发布，因为Migrations的缘故，这个库还不支持，只好乖乖的使用1.6了，系统一直运行良好，也就没有必要更新了。
- [django-crispy-forms](https://github.com/django-crispy-forms/django-crispy-forms) - 渲染表单用的，支持Bootstrap2/3/4、Foundation、uni-form等UI框架，非常好用基本上一个模板tags或者filter就可实现，不用再在Python代码中写什么 `{'class':'form-control'}` 了。Django 1.11 也引入了类似 crispy-form 的[基于模板的控件渲染](https://docs.djangoproject.com/en/1.11/ref/forms/renderers/)  - 目前API还没有稳定下来，估计要等到2.0了。
- [django-simple-captcha](https://github.com/mbi/django-simple-captcha) - 非常简单的表单验证码字段，支持随机字符串、算术题和自定义等不同验证方式。
- [pyserial](https://github.com/pyserial/pyserial) - Python的串口访问库。
- [PyModus](https://github.com/riptideio/pymodbus) Modbus通信协议的Python实现。
- [Construct](http://construct.readthedocs.io/en/latest/) - Python对象和二进制数据之间的构建和解析，基于声明式的代码，支持的特性包括了能碰到的情况。代码书写方式类型于ORM但好像又不完全是，有些时候代码感觉有些奇怪。
- [NetworkX](https://networkx.github.io/) - Python的图论及其算法的实现库，比如最短路算法、旅行商问题和背包问题，同时支持通过matplotlib显示图片。最近使用NetworkX实现了Echarts Graph的后台数据渲染，因为二者数据结构是相似的，实现起来就比较简单了。
- [qrcode](https://github.com/lincolnloop/python-qrcode) - Python的生成二维码库，包括命令行工具和代码API。
- [Pelican](http://blog.getpelican.com/) - 静态站点生成器，之前博客就是用Pelican构建的，由于在windows上比较麻烦，改成Hexo了。
- [lxml](http://lxml.de/) - xml和html解析处理库。
- [python-gsmmodem](https://github.com/faucamp/python-gsmmodem) - Python的GSM库，包括收发短信、电话处理等功能，支持常见型号的芯片。
- [tablib](http://docs.python-tablib.org/en/latest/) 数据格式转换，支持JSON/XLS/XLSX/YAML等文件格式。
- [python-lunardate](https://github.com/lidaobing/python-lunardate) - 纯Python实现的中国农历库。
