---
title: Django 1.10的Middleware
date: 2016-11-04 21:30:45
categories:
- 编程
tags:
- Django
---

本文介绍了Django 1.10新风格的Middleware。

<!-- more -->

Middleware是Django中“请求/相应”处理流程的一组“钩子”，是一个可以全局修改输入输出的底层轻量级插件。

每个Middleware组件负责处理一个特定的功能。例如`AuthenticationMiddleware`通过session将请求request和登录用户结合起来。

这篇文章描述了以下内容：

- Middleware如何工作
- 如何激活Middleware
- 自定义Middleware

Django中自带了一些Middleware。

## 编写自己的Middleware

Middleware的工厂（factory）一个是一个回调函数。
- `get_response`参数。
- 返回一个Middleware

Middleware和view一样的回调函数。

- 以HTTP请求request
- 需要返回一个响应response


函数形式

```
def simple_middleware(get_response):
    # One-time configuration and initialization.

    def middleware(request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.

        response = get_response(request)

        # Code to be executed for each request/response after
        # the view is called.

        return response

    return middleware
```
类形式，
```
class SimpleMiddleware(object):
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time configuration and initialization.

    def __call__(self, request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.

        response = self.get_response(request)

        # Code to be executed for each request/response after
        # the view is called.

        return response
```
