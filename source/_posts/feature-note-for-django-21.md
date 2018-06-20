---
title: Django2.1新功能笔记
date: 2018-06-19 20:25:50
categories: 技术研究
tags:
- Django
---

2018年6月18日，Django 发布 2.1Beta1，这是 2.1 系列的第一个公测版本，正式版预计于8月初发布。

本文仅列出一些比较重要的改变，具体可详见 https://docs.djangoproject.com/en/dev/releases/2.1/ 。

<!-- more -->


## 1 查看(view)权限

一直以来，Django 的模型默认有三种权限：

- 增加（Add）
- 删除（Delete）
- 修改（Change）

和 数据库原子操作 “CURD” 相比，Django 没有单独的查看权限。

Django 2.1 新增了 **查看(View)** 权限。在 Admin 中新增了函数  `ModelAdmin.has_view_permission() ` ，控制用户是否具有可读权限。

作为向后兼容，若一个用户具有修改权限，自然也具查看权限。

## 2 Cookie 的 SameSite 配置

SameSite-cookies是一种机制，用于定义cookie如何跨域发送。SameSite-cookies的目的是尝试阻止CSRF（Cross-site request forgery 跨站请求伪造）以及XSSI（Cross Site Script Inclusion (XSSI) 跨站脚本包含）攻击。

配置模块新增 *SESSION_COOKIE_SAMESITE* 和 *CSRF_COOKIE_SAMESITE* 项，如下：

```python
SESSION_COOKIE_SAMESITE = 'Lax'
# SESSION_COOKIE_SAMESITE = 'Strict'
# SESSION_COOKIE_SAMESITE = None

CSRF_COOKIE_SAMESITE = 'Lax'
```

可选的值有三种：

(1) "Strict"： 表明这个 cookie 在任何情况下都不可能作为第三方 cookie，绝无例外。

举个例子，如果 Github 网站设置了这个值之后，已登录的用户点击协作邮件讨论中私有项目的链接，浏览器将不会接受 Cookie ，用户也将不能访问该项目主页。

(2) "Lax"：在安全性和使用性之间作了一个平衡。属性只会在使用危险HTTP方法发送跨域cookie的时候进行阻止，例如POST方式。

(3) None：禁用该特性。

## 3 用户 - 权限 - 动作

![user-permission-action](/images/user-permission-action.png)

文档： https://docs.djangoproject.com/en/2.1/ref/contrib/admin/actions/#setting-permissions-for-actions

和模型一样，动作也拥有权限配置了，可以限制只有某些权限的用户使用该动作。例子：

```python
def make_published(modeladmin, request, queryset):
    queryset.update(status='p')
make_published.allowed_permissions = ('change',)
```

`make_published()` 仅对那些通过了 `ModelAdmin.has_change_permission()` 检测的用户可用。

在具体表现形式上，如果一个用户没有 change 权限，在对象列表页面的动作下拉菜单不会显示 "make_published" 项目。

`allowed_permissions` 中的多个权限是 “或” 的关系，只要通过任何一个即可，用户就可以使用该 动作（Action）。

## 4 模型初始化

模型类支持 ` __init_subclass__` 重写，相关内容 参见 [PEP 487](https://www.python.org/dev/peps/pep-0487/)。

一般来说，这个特性提供了 metaclass 处理复杂类继承的另一种解决方案，很少会用得到。

> Metaclasses are a powerful tool to customize class creation. They have, however, the problem that there is no automatic way to combine metaclasses. If one wants to use two metaclasses for a class, a new metaclass combining those two needs to be created, typically manually.

## 5 验证类视图

文档： https://docs.djangoproject.com/en/dev/topics/auth/default/#all-authentication-views

用户验证登录模块的函数视图被移除，可以使用对应的类视图，比如 `contrib.auth.views.LoginView` 类。

## 6 模型字段

`BooleanField` 允许 `null=True` 的设置，用以替代 ，`NullBooleanField` 的功能，这也意味着后者可能在未来的版本中移除。

## 7 其他

- 基于内存的缓存使用 LRU 选择算法，而不是之前的 伪随机算法。

## 8 参考文档

- [SameSite Cookie，防止 CSRF 攻击](http://www.cnblogs.com/ziyunfei/p/5637945.html)
