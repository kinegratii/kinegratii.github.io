---
title: Construct：二进制数据解析器和构建器
date: 2017-04-07 09:30:11
categories: 技术研究
tags:
- Python
- ORM
- 软件
---

Construct是一个强大的二进制数据解析和构建Python库，适用于建立大型复杂应用程序的通信协议，类似于二进制数据的ORM库。


## 1 标准库struct

struct是用于二进制数据的解析和构建，处理Python对象和二进制数据之间的转化。它的API非常简单：

- `stricut.pack(fmt, v1, v2, *)` 打包，Python对象到二进制数据
- `struct.unpack(fmt, buffer)` 解包，二进制数据到Python对象
- `struct.calcsize(fmt)` 计算格式字符串数据大小

<!-- more -->

## 2 Construct库

在开发大型应用程序中，比如实现NMS协议，直接使用 `struct` 模块需要写大量的代码。和数据库访问库相类比， `struct` 相当于 `mysql-python` 底层连接的角色。

| - | 二进制数据 | 数据库访问 |
| ----- | ------ | ------ |
| ORM | ? | Django ORM / sqlalchemy |
| Connection | struct | mysql-python / pysycopg2 |

在 [Pypi](https://pypi.python.org/pypi) 使用 “struct + binary” 搜索相关Python库，比较后，Construct是比较合适：

- github star数目为287
- 开发活跃，最新发布版本是2.8.11，时间2017-04-05。
- 支持Python3.6。
- 文档完备。

按照文档要求下载、安装、测试，基本上符合大部分需求。

**声明式定义**

使用 Struct 类定义数据结构。

```
>>> format = Struct(
...     "signature" / Const(b"BMP"),
...     "width" / Int8ub,
...     "height" / Int8ub,
...     "pixels" / Array(this.width * this.height, Byte),
... )
>>> format.build(dict(width=3,height=2,pixels=[7,8,9,11,12,13]))
b'BMP\x03\x02\x07\x08\t\x0b\x0c\r'
>>> format.parse(b'BMP\x03\x02\x07\x08\t\x0b\x0c\r')
Container(signature=b'BMP')(width=3)(height=2)(pixels=[7, 8, 9, 11, 12, 13])
```

**组合和继承**

Construct 支持原子结构和自定义结构的组合。使用 Adapter 自定义，下面为4字节存储IPv4数据的解决方案。

```
>>> class IpAddressAdapter(Adapter):
...     def _encode(self, obj, context):
...         return list(map(int, obj.split(".")))
...     def _decode(self, obj, context):
...         return "{0}.{1}.{2}.{3}".format(*obj)
...
>>> IpAddress = IpAddressAdapter(Byte[4])
```

使用方法

```
>>> IpAddress.parse(b"\x01\x02\x03\x04")
'1.2.3.4'
>>> IpAddress.build("192.168.2.3")
b'\xc0\xa8\x02\x03'
```
