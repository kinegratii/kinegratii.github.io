---
title: Python3之旅（六）—— 类型提示
date: 2017-05-20 20:31:53
categories: 编程
tags:
- Python
---

本文第《Python3之旅》的第六篇，主题是在Python3.5新增的类型提示（Type Hints），目前有关于这方面的中文文档相对较少，因此本文主要以 [PEP 484](https://www.python.org/dev/peps/pep-0484/) 为参考资料。

## 1 为什么要有类型提示

- 静态分析
- 重构
- 潜在的运行类型监察
- 基于类型信息的代码生成

在这些目标中，静态分析是最重要的。 这包括对离线类型检查器（如mypy）的支持，以及提供IDE可用于代码完成和重构的标准符号。

还应该强调的是， **Python将保持一种动态类型的语言，作者也不希望强制使用类型提示，即使依据惯例**。

<!-- more -->

## 2 在哪里添加类型提示

- 注释类型提示
- 特殊注释
- 单独的分发文件（相当于C语言的.h文件或Typescript的.d.ts文件）

一个使用注释类型提示的函数如下：

```python
def greeting(name: str) -> str:
    return 'Hello ' + name
```

这表示name参数的预期类型是str。类似地，预期的返回类型是str。

## 3 类型提示语法

### 3.1 类型提示符号

可用的类型提示主要有：

- 内置类型（包括标准库和第三方所定义的）
- 抽象基类
- `type` 模块中可用的类型
- 用户定义的类型（包括标准库和第三方所定义的）
- 特殊结构，如 `None` 、`Any` 、 `Union` 、`Callable`、类型变量和类型别名等。

### 3.2 类型别名

```python
Url = str

def retry(url: Url, retry_count: int) -> None:
    pass
```

### 3.3 回调对象

回调对象形式为 `Callable [[Arg1Type， Arg2Type]，ReturnType]`。

例子

```python
from typing import Callable

def feeder(get_next_item: Callable[[], str]) -> None:
    pass

def async_query(on_success: Callable[[int], None],
                on_error: Callable[[int, Exception], None]) -> None:
    pass
```

### 3.4 泛型

```python
from typing import Sequence, TypeVar

T = TypeVar('T')      # Declare type variable

def first(l: Sequence[T]) -> T:   # Generic function
    return l[0]
```
