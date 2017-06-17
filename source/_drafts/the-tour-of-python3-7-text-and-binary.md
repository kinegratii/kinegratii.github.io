---
title: Python3之旅（七） —— 文本和字符串
date: 2017-06-10 20:21:23
categories: 编程
tags:
- Python
---

主题词

- 文本
- 二进制
- 编码

相关模板

- struct
- binascii

文本和二进制数据是Python2和3的一个主要区别，对应的类表示如下表：

| - | 文本 | 二进制 |
| ------ | ------ | ------ |
| python2 | unicode | str |
| python3 | str | bytes |

编码和解码

文本数据 --(encode)--> 二进制数据 --(decode)--> 文本数据
