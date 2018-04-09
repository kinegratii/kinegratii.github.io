---
title: 圈复杂度和McCabe
date: 2018-04-08 19:40:35
categories: 技术研究
tags:

- 软件架构
---

圈复杂度（Cyclomatic Complexity）是衡量计算机程序复杂程度的一种措施。它根据程序从开始到结束的线性独立路径的数量计算得来的。在 Python 中可以使用 mccabe 包测量程序的圈复杂度。

<!-- more -->

## 1 圈复杂度

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

在 1976 年， Thomas J. McCabe 开发了使用有向图计算复杂的算法。为了得到这个指标，程序控制流程图可以画成一个有向图，其中：

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

## 2 McCabe的使用

在 Python 中可以使用 McCabe 包测量程序的圈复杂度。它可以当作一个独立的模块，也可以当作程序的一个插件。可以使用 pip 安装模块

```shell
pip install mccabe
```

### 2.1 作为命令行使用

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

### 2.2 作为 flake8 插件

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

## 3 程序控制图

### 3.1 生成程序控制图

以上述 `factorial` 函数代码为例子，将其保存为一个 *factorial.py* 文件中，如下：

```python
# coding=utf8

def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n-1)
```

第一步 使用 `python -m mccabe factorial.py -d`

输出 dot 文本图形描述语言的有向图。

```
graph {
subgraph {
node [shape=circle,label="If 4"] 2187726207240;
node [shape=circle,label="3:0: 'factorial'"] 2187726207016;
node [shape=circle,label="Stmt 5"] 2187726207408;
node [shape=point,label=""] 2187726207296;
node [shape=circle,label="Stmt 7"] 2187726207520;
2187726207240 -- 2187726207408;
2187726207240 -- 2187726207520;
2187726207016 -- 2187726207240;
2187726207408 -- 2187726207296;
2187726207520 -- 2187726207296;
}
}
```

第二步，Graphviz渲染图片

[Graphviz](http://www.graphviz.org/) 是 AT&T Labs Research开发的图形绘制工具软件.它使用一个特定的DSL(领域特定语言): dot作为脚本语言，然后使用布局引擎来解析此脚本，并完成自动布局。graphviz提供丰富的导出格式，如常用的图片格式，SVG，PDF格式等。

打开 Graphviz 编辑器，将上述文档保存为 factorial.gv 文档，生成程序控制图。



![factorial.gv](/images/factorial.png)

第三步，计算 McCabe 复杂度

根据公式，复杂度： M = 5 - 4 + 2 x 1 = 2

## 4 降低复杂度

使用字典替代复杂的 *if-else* 分支代码是 Python 中降低复杂度一个有效的方法。

比如可以将下面的分支代码：

```python

def f(x):
    if x == 'a':
        return 1
    elif x == 'b':
        return 2
    else:
        return 9
```

改为下面的字典映射代码

```python

def f(x):
    return {
        'a': 1,
        'b': 2
    }.get(x, 9)  
```

复杂度也从 3 降低到 1 。

## 5 参考资料

- [Software Architecture with Python](https://www.amazon.com/Software-Architecture-Python-Balachandran-Pillai/dp/1786468522)
- [The Architecture of Open Source Applications](http://www.aosabook.org/en/index.html)
