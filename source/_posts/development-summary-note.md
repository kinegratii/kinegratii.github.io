---
title: 2019年开发总结（一）
date: 2019-05-22 21:33:16
categories: 编程
tags: Python
---

2019年开发总结。

<!-- more -->

## raise的用法

### 直接抛出异常

```shell
>>> raise NameError('HiThere')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: HiThere
```

如果 raise 后是一个类对象，则相当于无参的对象。

```python
raise ValueError  # shorthand for 'raise ValueError()'
```

### 不处理，再次抛出

```shell
>>> try:
...     raise NameError('HiThere')
... except NameError:
...     print('An exception flew by!')
...     raise
...
An exception flew by!
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
NameError: HiThere
```

### 以其他异常方式抛异常

```shell
>>> try:
...     print(1 / 0)
... except:
...     raise RuntimeError("Something bad happened")
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
ZeroDivisionError: division by zero

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError: Something bad happened
```

### 连锁异常

```shell
>>> try:
...     print(1 / 0)
... except Exception as exc:
...     raise RuntimeError("Something bad happened") from exc
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
ZeroDivisionError: division by zero

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError: Something bad happened
```

### 参考

- [PEP3134](https://legacy.python.org/dev/peps/pep-3134/)
- [Python 中 raise 和 raise/from 的区别](https://blog.csdn.net/jpch89/article/details/84315444)



## 文档生成

在Python开发中， python-docx 和 docxtpl 是两个生成文档的第三方库。

- 这两个库不支持 doc 格式文件；
- 使用方式简单，类似于 jinja2模板



## 压缩文件处理

rar 压缩算法不公开。

`zipfile.write()` 参数说明



## postgresql数字布尔值转化

修改int为boolean类型的SQL代码：

```sql
ALTER TABLE <tablename> ALTER COLUMN <fieldname> DROP DEFAULT;
ALTER TABLE <tablename> ALTER <fieldname> TYPE bool USING <fieldname>::boolean;
ALTER TABLE <tablename> ALTER COLUMN <fieldname> SET DEFAULT FALSE;
```

例子：

```sql
ALTER TABLE spideroption ALTER COLUMN activate DROP DEFAULT;
ALTER TABLE spideroption ALTER activate TYPE bool USING activate::boolean;
ALTER TABLE spideroption ALTER COLUMN activate SET DEFAULT FALSE;
```

参考资料： https://stackoverflow.com/a/1740337