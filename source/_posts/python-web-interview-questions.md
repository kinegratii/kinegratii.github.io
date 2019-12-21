---
title: Python开放工程师笔试题（web方向）
date: 2019-12-20 20:28:25
categories: 技术研究
tags:
- Python
- Django
---

Python开放工程师（web方向）

<!-- more -->

## 基础语法

★ 列出5个常用Python标准库？

★ 在单元测试库unittest中，函数 `unittest.TestCase.subTest` 和 装饰器 `unittest.skipIf` 的作用分别是什么？

★ Django数据库操作中，函数 `bulk_create` 和 循环 save 在用法上有什么区别？

★ 什么是“N+1”查询的问题。Django如何解决这一问题。

★ WSGI和ASGI有什么相同点和不同点。

★ 在web开发中，表单传递参数和JSON传递参数有什么不同，各自的应用场景是什么？

★ 下面下划线的作用是什么？

```python
_, user = User.objects.get_or_create(username='admin', default={'password':'123456', 'last_updated': '2019-10-23 08:00:00'})
```

## 程序阅读题

★ 阅读下面的python2程序，回答第1-2题。
```python
# coding=utf8

 a = range(1, 11)
result = reduce(lambda x, y:x + y, a[7:] + a[-4::2] + a[-5:5] + a[3:-2:4])  / 10
```
（1） 变量 result的值是多少？

（2） 以上代码在python3下是否可以正常运行，如果不能，需要怎样修改才能正常运行且结果不变。（3点）


★ 阅读下列程序片段，根据运行结果补充函数 `inject` 的内容。

```python
class App:
    def __init__(self, app_id):
        self.app_id = app_id
    def inject(self. **kwargs):
        pass

app = App(2)

@app.inject(app_id=6)
def build_context(context):
    context = context or {}
    context.update({'method': 'build_context'})
    return context

if __name__ == '__main__':
    print(build_context({'name':'hello'})) # 结果为 {'name':'hello', 'app_id': 6, 'method': 'build_context'}
```
