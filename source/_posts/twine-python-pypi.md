---
title: twine：python包发布工具
date: 2017-04-24 11:01:17
categories: 编程
tags:
- Python
- PyPI
- twine
---

Twine是一个PyPI相关的工具。主要特点：

- HTTPS安全连接
- 上传过程中不会执行 `setup.py`
- 在发布前支持测试分发功能
- 支持人任何包格式，包括了wheels


以 [ConstChoices](https://github.com/kinegratii/ConstChoices)为例，已经完成了 `setup.py` 文件。
1 安装

```
pip install twine
```

2 生成wheel包

```
python setup.py sdist bdist_wheel --universal
```

3 注册项目（第一次上传时必须）

```
twine register dist/ConstChoices-0.1.0-py2.py3-none-any.whl
```

4 上传

```
twine upload dist/*
```

5 成功！

> 参考文档 [Python Packaging User Guide — Python Packaging User Guide documentation](https://packaging.python.org/)
