---
title: 圈复杂度和McCabe
date: 2018-04-08 19:40:35
categories: 技术研究
tags:

- 软件架构
---

圈复杂度（Cyclomatic Complexity）是衡量计算机程序复杂程度的一种措施。它根据程序从开始到结束的线性独立路径的数量计算得来的。在 Python 中可以使用 mccabe 包测量程序的圈复杂度。它可以当作一个独立的模块，也可以当作程序的一个插件。

<!-- more -->

## 圈复杂度

对于没有任何分支的代码，它的圈复杂度为 1 ，意味着代码只有一条路径。例如下面的函数：

```python

def add(a, b):
    return a + b
```

对于有一条分支的代码，它的圈复杂度为 2 ，比如下面递归计算阶乘的代码：

```python
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n-1)
```

为了得到这个指标，程序控制流程图可以画成一个有向图，其中：

- 节点表示一个程序块
- 边表示从一个程序块到另一个程序块的控制流

根据程序的控制图，McCabe 复杂度可以表示为

```
M = E - N + 2P
```

其中：

- E：边的数量
- N：节点的数量
- P：连接组件的数量

## McCabe的使用

McCabe 支持 Python 各个版本。可以使用 pip 安装模块

```shell
pip install mccabe
```

### 作为命令行使用

和 `unittest` 、`flake8` 等工具一样。


```shell
$ python -m mccabe --min 5 mccabe.py
("185:1: 'PathGraphingAstVisitor.visitIf'", 5)
("71:1: 'PathGraph.to_dot'", 5)
("245:1: 'McCabeChecker.run'", 5)
("283:1: 'main'", 7)
("203:1: 'PathGraphingAstVisitor.visitTryExcept'", 5)
("257:1: 'get_code_complexity'", 5)
```

### 作为 flake8 插件

当 flake8 版本在 2.0 以上和 McCabe 已经安装的情况下，该插件可用。

```
$ flake8 --version
2.0 (pep8: 1.4.2, pyflakes: 0.6.1, mccabe: 0.2)
```

在命令中使用 `--max-complexity` 选项即可。

```
$ flake8 --max-complexity 10 coolproject
...
coolproject/mod.py:1204:1: C901 'CoolFactory.prepare' is too complex (14)
```
根据 McCabe 圈复杂度大于 10 ，就认为是 too complex ，需要进行重构以降低复杂度。
