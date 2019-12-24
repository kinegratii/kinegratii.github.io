---
title: 编译Python脚本
date: 2019-04-25 20:37:10
categories: 技术研究
tags:
- 构建
---

本文描述了将 python 源代码文件编译成相应的库文件。

<!-- more -->

## 1 文件类型

python开发中文件类型如下：

| 序号 | 文件类型 | 描述 |
| ------ | ------ | ------ |
| 1 | .py | 源代码文件 |
| 2 | .pyw | 源代码文件|
| 3 | .pyi | 存根文件，用于类、函数的声明；参见PEP 484 |
| 4 | .pyc | 字节码文档 |
| 5 | .pyo | 和pyc类似，优化后的字节码文件 |
| 6 | .pyd | 库文件，和dll 和 so 文件类似 |


### 和dll的区别

关于 dll 和 pyd 的区别可以参见 官方文档 [Is a *.pyd file the same as a DLL?](https://docs.python.org/3/faq/windows.html#id6)。

### 编译pyd文件

[Cython](https://cython.org/) 是一个静态编译器，也是 python语言的超集。 Cpython 将python代码转化为 c 语言代码，这个过程称为 **Cythonizing** 。和 原来的 python 代码相比，其执行速度将有较大的提高。

第一步，在windows下编译需要两个工具：

- 微软python编译器，在 https://visualstudio.microsoft.com/downloads/ 下载记可
- cpython 包，使用 `pip install cpython`

第二步，提供源代码。以 *hello.py* 文件为例子，内容如下：

```python
#!/usr/bin/env python


def hello():
    print("Hello world!")
```

第三步，编写一个 setup.py 文件，基本代码如下：

```python
#!/usr/bin/env python
from setuptools import setup
from Cython.Build import cythonize

setup(
    ext_modules=cythonize('module.py')
)
```

第四步，使用以下命令开始编译

```shell
python setup.py build_ext --inplace
```

会在同一个目录下生成 hell.pyd 文件，这个文件可以像正常的模块进行 import 。
