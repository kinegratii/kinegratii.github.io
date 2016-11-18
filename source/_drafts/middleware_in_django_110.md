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

`get_response`回调可能是Django中实际的view或者是下一个Middleware，但无需关注是那种情况。

Middleware可以存在任何可以import的地方。

## `__init__(get_response)`

Middleware工厂需要有`get_response`的参数，你可以初始化一些全局性的状态。

- Django初始化Middleware只能接收一个`get_response`参数，因此你不能在`__init__`定义其他参数。
- 和 `__call__`每次请求都被调用，`__init__`只会在Django项目启动时调用一次。

> Django 1.10变更
-  在之前版本中，`__init__`是在第一次请求时调用。
- 为了兼容之前版本`__init__`无参调用形式，可以使用 `__init__(get_response=None)`

## 禁用Middleware

在`__init__`中抛出`MiddlewareNotUsed`可以禁用这些Middleware，当`debug=True`时，Django在记录一个log消息。

## 激活Middleware

将Middleware加入settings文件的`MIDDLEWARE`变量中，可激活Middleware。

在 `MIDDLEWARE` 中每个Middleware中路径。在`django-admin startproject`命令创建的项目默认配置包含以下Middleware。

```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

一个Django项目不需要任何Middleware，但强烈建议包含`CommonMiddleware`。

Middleware中顺序有关系。例如，

## Middleware顺序

在请求处理过程中，Django按照MIDDLEWARE从上到下定义的顺序应用这些Middleware。

Middleware工作流程如同像洋葱。Middleware一层一层包裹在实际的视图中。请求依次通过Middleware进入view视图中，在返回的响应response通过相反的顺序返回。

## 其他Middleware钩子

### process_view()

`process_view(request, view_func, view_args, view_kwargs)`

request是一个`HttpRequest`对象，`view_func`是一个视图函数对象，`view_args`是传入`view_func`的位置参数，`view_kwargs`是传入`view_func`的可选参数。`view_args`和`view_kwargs`都不包含第一个参数`request`。

`process_view`仅在Django视图之前被调用。

`process_view`需要返回None或者`HttpResponse`实例。当返回None时，Django继续执行其他Middleware直到实际的视图。当返回一个`HttpResponse`时，Django不再继续往下执行其他Middleware，直接返回这个`HttpResponse`。

> 注意
在Middleware中访问`request.POST`，可能影响后续流程文件上传的处理，这种情况应该避免。

### process_exception()

`process_exception(request, exception)`

request是一个`HttpRequest`对象，`exception`是视图抛出的异常实例。当一个视图抛出异常时，Django会调用`process_exception`函数。当函数返回`HttpResponse`时，Django直接将返回到浏览器中。否则转入默认的异常处理机制处理。

再次提醒，在响应处理期间，中间件是按照倒序执行的，也包括了`process_exception`。如果一个异常处理的中间件返回了一个响应，在该Middleware都不会被调用。

### process_template_response()

`process_template_response(request, response)`

request是一个`HttpRequest`对象，`response`是一个由视图或者其他Middleware返回的`TemplateResponse`对象（或者等价对象）。

如果响应的实例有render()方法，`process_template_response()`在视图刚好执行完毕之后被调用，这表明了它是一个`TemplateResponse`对象（或等价的对象）。

这个方法必须返回一个实现了render方法的响应对象。它可以修改给定的response对象，通过修改 `response.template_name`和`response.context_data`或者它可以创建一个全新的 `TemplateResponse`或等价的对象。

你不需要显式渲染响应 —— 一旦所有的模板响应中间件被调用，响应会自动被渲染。

在一个响应的处理期间，中间件以相反的顺序运行，这包括`process_template_response()。
