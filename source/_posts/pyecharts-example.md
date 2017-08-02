---
title: pyecharts使用示例
date: 2017-08-02 09:26:37
categories: 编程
---

pyechart是一个用于生成 Echarts 图表的类库。实际上就是 Echarts 与 Python 的对接。

<!-- more -->

> [Echarts](https://github.com/ecomfe/echarts) 是百度开源的一个数据可视化 JS 库。看了官方的介绍文档，觉得很不错，就想看看有没有人实现了 Python 库可以直接调用的。Github 上找到了一个 [echarts-python](https://github.com/yufeiminds/echarts-python) 不过这个项目已经很久没更新且也没什么介绍文档。借鉴了该项目，就自己动手实现一个，于是就有了 pyecharts。API 接口是从另外一个图表库 [pygal](https://github.com/Kozea/pygal) 中模仿的。

目前项目还处于快速开发阶段，一些API还没有完全稳定下来。

## 与 tablib整合

```python
import tablib

from pyecharts import Bar

ds = tablib.Dataset()
ds.headers = ['Name', 'height']
ds.append(['Tim', 167])
ds.append(['John', 170])
ds.append(['Tus', 159])
ds.append(['Bob', 159])

bar = Bar('The height')
bar.add('Height', ds.get_col(0), ds.get_col(1))
bar.render('height_bar.html')
```

显示示例

![device_graph](/images/device_graph.jpg)


## 与networkx整合

```python
import networkx as nx
from networkx.readwrite import json_graph

from pyecharts import Graph

g = nx.Graph()

g.add_node('N1', name='Node 1')
g.add_node('N2', name='Node 2')
g.add_node('N3', name='Node 3')
g.add_edge('N1', 'N2')
g.add_edge('N1', 'N3')

g_data = json_graph.node_link_data(g)

print(g_data)

eg = Graph('设备最新拓扑图')
eg.add('Devices', nodes=g_data['nodes'], links=g_data['links'])
# eg.show_config()
eg.render()
```

显示示例

![height_bar_demo](/images/height_bar_demo.jpg)
