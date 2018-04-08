---
title: LazyObject 简单实现
date: 2018-04-07 08:22:26
categories: 编程
---

基于 django.utils.functional.SimpleLazyObject 的延迟初始化技术。

<!-- more -->

```python
import six

def proxy_method(func):
    def inner(self, *args):
        if self._wrapped is None:
            self._setup()
        return func(self._wrapped, *args)

    return inner

class LazyObject(object):
    _wrapped = None

    def __init__(self, func):
        self.__dict__['_setupfunc'] = func

    __getattr__ = proxy_method(getattr)

    def __setattr__(self, key, value):
        if key == '_wrapped':
            self.__dict__['_wrapped'] = value
        else:
            if self._wrapped is None:
                self._setup()
            setattr(self._wrapped, key, value)

    def _setup(self):
        self._wrapped = self._setupfunc()

    __getitem__ = proxy_method(operator.getitem)

    if six.PY2:
        __str__ = proxy_method(str)
        __unicode__ = proxy_method(unicode)  # NOQA: unicode undefined on PY3
        __nonzero__ = proxy_method(bool)
    else:
        __bytes__ = proxy_method(bytes)
        __str__ = proxy_method(str)
        __bool__ = proxy_method(bool)

```
