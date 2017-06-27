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

## 3 使用语法

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

泛型指的是一个变量可以是不同的类型，主要有两种方式：

第一，使用内置的容器类型的类

```python
from typing import Mapping, Set

def notify_by_email(employees: Set[Employee], overrides: Mapping[str, str]) -> None:
```
第二，使用 `TypeVar` 定义类型模板。

```python
from typing import Sequence, TypeVar

T = TypeVar('T')      # Declare type variable

def first(l: Sequence[T]) -> T:   # Generic function
    return l[0]
```

### 3.5 前置引用

当变量类型还未定义或者模块存在相互引用时可以使用类名字符串形式代替 class对象，一个典型的例子是在定义树形结构，一棵树的左子树还是一个树的类型。

```python
class Tree:
    def __init__(self, left: 'Tree', right: 'Tree'):
        self.left = left
        self.right = right
```
类型提示中使用 `'Tree'` 代替 `Tree`，这和Django中 models.ForeiginKey的model参数定义是一致的。

### 3.6 组合类型Union

变量可以是多种类型。


```python
_empty = object()

def func(x=_empty):
    if x is _empty:  # default argument value
        return 0
    elif x is None:  # argument was provided and it's None
        return 1
    else:
        return x * 2
```

### 3.7 任意类型Any

```python
from typing import Mapping

def use_map(m: Mapping) -> None:  # Same as Mapping[Any, Any]

def add(a:Any, b:Any) -> Any:
```
### 3.8 无返回值NoReturn

```python
import sys
from typing import NoReturn

  def f(x: int) -> NoReturn:  # Error, f(0) implicitly returns None
      if x != 0:
          sys.exit(1)
```
