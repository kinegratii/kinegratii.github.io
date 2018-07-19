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

## 1 Python 版本支持

Python 版本要求 3.5+ ，不再支持 3.4 。

## 2 查看(view)权限

Django 2.1 新增了 **查看(View)** 权限。至此 Django 的模型默认有四种权限：

| 权限 | Code |
| ------ | ------ |
| 增加（Add） | `<app_label>.add_<modelname>` |
| 删除（Delete） | `<app_label>.delete_<modelname>` |
| 修改（Change） | `<app_label>.change_<modelname>` |
| 查看（View） | `<app_label>.view_<modelname>` |

正好和数据库原子操作 “CURD” 一一对应。需要注意的是，**一个用户具有修改权限，自然也具查看权限。** 这既符合实际逻辑，又保证了旧有版本的兼容性。

### has_view_permission函数

在 Admin 中新增了函数  `ModelAdmin.has_view_permission() ` ，控制用户是否具有可读权限。函数签名为：

```python
ModelAdmin.has_view_permission(request:HttpRequest, obj:Model=None) -> bool
```

该函数应当返回一个布尔值，表明 查看某个具体的模型对象 obj 的操作是否被允许。如果 obj 为 None ，则表示 该模型类型的所有对象的查看操作是否被允许。

默认实现上，如果一个用户具有 “Change” 或 "View" 权限，则返回 True 。

### 兼容性

因为新的查看权限的权限码（code）使用了 `<app_label>.view_<modelname>` 的格式，因此如果之前你将该权限码用于自定义的权限并且和查看权限有冲突时，应当更新自己定义的权限的权限码。

另外，如果之前使用了 [django-admin-view-permission](https://github.com/ctxis/django-admin-view-permission) 库，可能需要花费一定的时间完成整合工作。

## 3 Cookie 的 SameSite 配置

[SameSite-cookies](https://www.owasp.org/index.php/SameSite) 是一种安全机制，用于定义cookie如何跨域发送。SameSite-cookies的目的是尝试阻止CSRF（Cross-site request forgery 跨站请求伪造）以及XSSI（Cross Site Script Inclusion (XSSI) 跨站脚本包含）攻击。

配置模块新增 *SESSION_COOKIE_SAMESITE* 和 *CSRF_COOKIE_SAMESITE* 项，如下：

```python
SESSION_COOKIE_SAMESITE = 'Lax'
# SESSION_COOKIE_SAMESITE = 'Strict'
# SESSION_COOKIE_SAMESITE = None

CSRF_COOKIE_SAMESITE = 'Lax'
```

可选的值有三种（请注意下首字母大写）：

(1) "Strict"： 表明这个 cookie 在任何情况下都不可能作为第三方 cookie，绝无例外。

举个例子，如果 Github 网站设置了这个值之后，已登录的用户点击协作邮件讨论中私有项目的链接，浏览器将不会接受 Cookie ，用户也将不能访问该项目主页。

(2) "Lax"：在安全性和使用性之间作了一个平衡。属性只会在使用危险HTTP方法发送跨域cookie的时候进行阻止，例如POST方式。

(3) None：禁用该特性。

## 4 用户 - 权限 - 动作

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

## 5 模型初始化

模型类支持 ` __init_subclass__` 函数重写。

```
>>> class QuestBase:
...    # this is implicitly a @classmethod (see below for motivation)
...    def __init_subclass__(cls, swallow, **kwargs):
...        cls.swallow = swallow
...        super().__init_subclass__(**kwargs)

>>> class Quest(QuestBase, swallow="african"):
...    pass

>>> Quest.swallow
'african'
```

这个特性提供了 metaclass 处理复杂类继承的另一种解决方案。当一个类需要使用两个以上的元类（Metaclass）时，需要手动创建一个新的 元类 以合并这几个元类。现在可以重写 `__init_subclass__` 来自定义类的创建流程，而无需创建用于合并的元类。

一般来说，这个特性很少会用得到。，相关内容 参见 [PEP 487](https://www.python.org/dev/peps/pep-0487/)。

## 6 验证类视图

文档： https://docs.djangoproject.com/en/dev/topics/auth/default/#all-authentication-views

`contrib.auth` （用户验证登录模块）的函数视图 *被移除* (Removed)，可以使用对应的类视图，比如 `contrib.auth.views.LoginView` 类。在更新到 Django 2.1 时，必须完成这一个改变。

比如函数视图：

```python

from django.urls import re_path
from django.contrib.auth import views as auth_views

urlpatterns = [
    re_path(r'^login/$', auth_views.login, {'template_name': 'accounts/login.html'}, name='account_login'),
    re_path(r'^logout/$', auth_views.logout, {'template_name': 'accounts/logout.html'}, name='account_logout')
]
```

可以改写为：

```python

from django.urls import re_path
from django.contrib.auth import views as auth_views

urlpatterns = [
    re_path(r'^login/$', auth_views.LoginView.as_view(), {'template_name': 'accounts/login.html'}, name='account_login'),
    re_path(r'^logout/$', auth_views.LogoutView.as_view(), {'template_name': 'accounts/logout.html'}, name='account_logout')
]
```

## 7 模型字段

### BooleanField

`BooleanField` 允许 `null=True` 的设置，用以替代 `NullBooleanField` 的功能，这也意味着后者可能在未来的版本中移除。

### JSONField

`django.contrib.postgres.fields.JSONField` 查询严格区分以下两种 “空值” 形式：

- 键 key 的值为 null ： 使用 `Q(key=None)`
- 缺少键 key ：使用 `Q(key__isnull=True)`

> 这个特性和之前有所变化，并且是第一次完整的列在文档中，因此需要按照最新的文档来检视现有的代码。

## 8 数据库查询高级特性

Django 数据库查询的高级特性通常包括：

- 自定义 Lookup
- 查询表达式
- 条件表达式
- 数据库函数

自 1.8 以来，Django 在数据库查询支持方面越来越完善，每个版本都会有新的类和函数被添加进来，使用这些特性的优点在于：

- 将复杂的数据处理放在数据库层，符合 web 开发的最佳实践
- 可以适应于不同的数据库类型
- 避免多线程下的 *竞态条件* 问题

这些特性通常是业务相关的，可以使用 `extra` 和 `raw` 这两个原始 SQL 调用函数。 

一般来说，如果现有代码可以正常工作，可以不必立即使用这些新特性。

## 9 其他

- 基于内存的缓存使用 LRU 选择算法，而不是之前的 伪随机算法。
- 默认的 jQuery 版本从 2.3.3 更新至 3.3.1

## 10 参考文档

- [SameSite Cookie，防止 CSRF 攻击](http://www.cnblogs.com/ziyunfei/p/5637945.html)
