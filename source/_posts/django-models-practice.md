---
title: Django实践：数据库查询
date: 2016-11-18 20:43:57
categories:
 - 编程
tags:
 - Django
---

本文总结了一些Django数据库查询的实践经验。

- 基本的增删改查
- 分类统计：`aggregate`和`annotate`的使用
- 实现按年/月/日统计
- Manager和QuerySet的混合使用
- 编写迁移文件

<!-- more -->


## 1概述

根据[Django官方文档](https://docs.djangoproject.com/en/1.10/)，本人整理出与数据库相关的话题列表，

- 基本的增删改查
- 外键访问 (Accessing related objects)
- 管理器和查询集 (Manger & QuerySet)
- 原生SQL (raw SQL)
- 事务 (Transactions)
- 统计、聚合和分组 (Aggregation)
- 搜索 (Search)
- 自定义字段 (Custom fields)
- 多数据库 (Multiple databases )
- 查询表达式和自定义查询表达式(Lookup expressions & Custom lookups)
- 条件表达式 (Conditional Expressions)
- 数据库函数 (Database Functions)
- 数据库优化 (Optimize database access)
- 数据库迁移 (Migrations)

其中一部分是在数据库有对应的内容，另外一部分则是Django框架自有的特性。涉及的代码主要包括以下三个包：

- `django.db.connections`: 底层数据库连接对象操作
- `django.db.migrations`:  迁移相关
- `django.db.models`: 模型定义、数据库查询

## 2 查询API

### 2.1 模型描述

以一个设备管理系统的简易系统为例，该项目包含了设备描述和报警记录。

- 设备以序列号唯一确定该设备，可默认为“主键”。
- `longitude`  `latitude`和`address`表示设备的地理位置，创建后可认为不可更改。
- 设备含有使用和报警两个状态标识变量。
- `Alarm.catalog`表示警报类型，定义在choices上。

```
class Device(models.Model):
    serial = models.CharField(verbose_name='序列号', max_length=100, unique=True)
    name = models.CharField(verbose_name='名称', max_length=100, null=True, blank=True)
    longitude = models.FloatField(verbose_name='经度', null=True,
                                  validators=[validators.MaxValueValidator(180), validators.MinValueValidator(-180)])
    latitude = models.FloatField(verbose_name='纬度', null=True,
                                 validators=[validators.MaxValueValidator(90), validators.MinValueValidator(-90)])
    address = models.CharField(verbose_name='地址', max_length=100, null=True)
    is_active = models.BooleanField(verbose_name='使用标识', default=True)
    is_alarm = models.BooleanField(verbose_name='报警标识', default=False)
    latest_alarm_time = models.DateTimeField(verbose_name='最新报警时间', null=True, blank=True)
    latest_alarm_remark = models.CharField(verbose_name='最新报警内容', max_length=200, null=True, blank=True)

    def __str__(self):
        return self.serial

class Alarm(models.Model):
    ALARM_CATALOG_CHOICES = (
          ('low_battery', '低电量'),
          ('fail_connection', '通信故障'),
          ('location_moved', '位置移动')
      )
    device = models.ForeignKey(Device, verbose_name='设备')
    create_time = models.DateTimeField(verbose_name='创建时间', default=timezone.now)
    catalog = models.CharField(max_length=30, null=True, choices=ALARM_CATALOG_CHOICES)
    content = models.CharField(verbose_name='内容', max_length=30, null=True, blank=True)
    read = models.BooleanField(verbose_name='已读', default=False)
```

### 2.2 查询一览表

#### 2.2.1 检索、过滤、外键查询、分页

```
# 查询mac地址为'0FFFFFFF561C4030'的设备
try:
    device = models.Device.objects.get(serial='0FFFFFFF561C4030')
except models.Device.DoesNotExist:
    device = None

<Device:0FFFFFFF561C4030>

# 查询地址包含“小区”的设备。
>>> device_list = models.Device.objects.filter(address__icontains='小区')
<QuerySet [<Device: 0FFFFFFF5BC91F87>, <Device: 0FFFFFFF561C4030>, ...]>

# 查询设备'0FFFFFFF5BC91F87'的所有报警记录
models.Alarm.objects.filter(device__serial='0FFFFFFF5BC91F87')

# 假设设备列表每页20项，查询第3页的数据。
device_list = models.Device.objects.all()[20:30]
```
#### 2.2.2 更新

```
# 单记录更新
try:
    device = models.Device.objects.get(serial='0FFFFFFF561C4030')
    device.is_active = False
    device.save()
except models.Device.DoesNotExist:
    pass

# 多记录更新
models.Device.objects.filter(serial__in=['0FFFFFFF561C4030', '0FFFFFFF56174CA0']).update(is_active=False)

# 去掉所有设备地址中“福建省”的前缀，比如“福建省厦门市Xxxx”改为"厦门市Xxxx"。
models.Device.objects.filter(address__isnull=False).update(address=F('address').strip('福建省'))
```

#### 2.2.3 删除

```
# 删除单条记录
try:
    device = models.Device.objects.get(serial='0FFFFFFF561C4030')
    device.delete()
except models.Device.DoesNotExist:
    pass

# 批量删除
models.Alarm.objects.filter(serial='0FFFFFFF561C4030').delete()
# 由于delete只是QuerySet的方法，并没有向Manager公开，需要先调用all方法
models.Alarm.objects.all().delete() # OK
models.Alarm.objects.delete() # Fail
```

#### 2.2.4 基础统计：数据、最值和平均值

```
# 计算设备0FFFFFFF561C4030所有的报警数目。
models.Alarm.objects.filter(device__serial='0FFFFFFF561C4030').count()
# 165

# 计算每个设备的报警数目。
>>> device_list = models.Device.objects.annotate(num_alarms=Count('alarm'))
>>> device_list
<QuerySet [<Device: 0FFFFFFF561C4021>, <Device: 0FFFFFFF561C4030>, ...]>
>>> device_list [0].num_alarms
34

# 查询2016年报警次数最多的前5个设备

models.Alarm.objects.filter(create_time__year=2016).values('serial').annotate(num_alarms=models.Count('serial')).order_by('-num_alarms')[:5]

[
    {'serial':'0FFFFFFF9FFC15F9', 'num_alarms':38},
    {'serial':'0FFFFFFF71281152', 'num_alarms':32},
    {'serial':'0FFFFFFF5992B723', 'num_alarms':27},
    {'serial':'0FFFFFFF05E20356', 'num_alarms':21},
    {'serial':'0FFFFFFF66DDF14D', 'num_alarms':12},
]

```

#### 2.2.5 分类统计

分类统计有以下两种方法。

- `aggregate` + 条件表达式Case，返回一个字典形式的结果，未出现的分类值默认为None，需要使用`Coalesce`函数设置默认值
- `annotate` + 分组GROUP BY，返回一个列表形式的结果，未出现的分类值不会出现在最后的结果中

下面是两种方式查询最近30天中每个报警类型的报警数目为例。

```
latest_week_qs = models.Alarm.objects.filter(create_time__gt=timezone.now()-timedelta(days=30).

# aggregate方式
latest_week_qs.aggregate(
    fail_connection=Coalesce(Sum(
        Case(When(catalog='fail_connection', then=1), output_field=models.IntegerField()),
    ), 0),
    low_battery=Coalesce(Sum(
        Case(When(catalog='low_battery', then=1), output_field=models.IntegerField()),
    ), 0),
    location_moved=Coalesce(Sum(
        Case(When(catalog='location_moved', then=1), output_field=models.IntegerField()),
    ), 0)
)
# 结果
{‘fail_connection’：12， 'low_battery'：34, 'location_moved': 0}

# annotate方式
latest_week_qs.values('catalog').annotate(count=Count('catalog'))
# 结果
[{'catalog':'low_battery', 'count':34},{'catalog':'fail_connection'， 'count': 12}]
```

#### 2.2.6 日期统计

实现按年、月、日统计通常有两种方法：

- 数据库函数 `django.db.connection.ops.date_trunc_sql`
- 对第一种的封装类DateExtra，仅Django 1.10+可用

以上两种结果中日期类型不一样，第一种返回时datetime对象，第二种只返回其中的分类字段，为整数类型。

```
# 查询mac地址为`0FFFFFFF561C4030`的设备最近一周每天报警次数。
models.Alarm.objects.filter(serial='0FFFFFFF561C4030', create_time__gt=timezone.now()-timedelta(days=7)).extra(
    select={'dt': connection.ops.date_trunc_sql('day', 'create_time')}
).values('dt').annotate(count=models.Count('create_time')).order_by('dt')

[
    {'count':4, 'dt':datetime.datetime(2016, 11, 08, 0, 0, 0,0)},
    {'count':2, 'dt':datetime.datetime(2016, 11, 11, 0, 0, 0,0)},
    {'count':1, 'dt':datetime.datetime(2016, 11, 12, 0, 0, 0,0)}
]
# 在Django 1.10+ 还可以使用`DateExtra`相关类
models.Alarm.objects.filter(serial='0FFFFFFF561C4030', create_time__gt=timezone.now()-timedelta(days=7))
    annotate(day=ExtractDay('create_time'))
# 结果
{
  'count':4, 'day': 8,
  'count':2, 'day': 11,
  'count':1, 'day': 12
}
```

### 2.3 数据库函数

- `django.db.models.Q`: 与、或、非条件组合查询
- `django.db.models.F`: `F()`表示数据库中相应字段的值，用于计数器更新等。
- `django.db.models.Functions.Coalesce`:接收一组参数，返回第一个不为None的数据，

更多函数可参考[Database Functions](https://docs.djangoproject.com/en/1.10/ref/models/database-functions/)。

## 3 管理器和查询集

### 3.1 管理器与模型的关系

管理器是Django数据库查询的接口。查询语法 `models.XxModels.objects.filter(*kwargs)` 。

- 一个模型可以拥有一个或多个管理器。
- 默认情况下，每个模型都有名为objects的管理器，默认返回数据表中所有记录。
- 管理器来源于默认管理器、外键管理器和自定义管理器。

### 3.2 自定义管理器

当一些查询逻辑复杂而且经常使用时，往往是在管理器上添加自定义函数封装相关查询逻辑，一方面减少重复代码，另一方面对view层透明，有利于MVC职责分工。

自定义管理器有三种方法

#### 3.2.1 继承 models.Manager

这是默认出现的方式，以下 `period_date`函数封装了日期时间段查询函数

```
class AlarmManager(models.Manager):
    def period_date(self, field, start_date=None, end_date=None, fmt='%Y-%m-%d'):
        """封装日期开始结束时间段查询"""
        def to_datetime(val):
            if isinstance(val, (datetime, date)):
                return val
            else:
                try:
                    return datetime.strptime(val, fmt)
                except (TypeError, ValueError):
                    pass

        kvs = {}
        start_date = to_datetime(start_date)
        end_date = to_datetime(end_date)
        if start_date:
            kvs[field + '__gte'] = start_date
        if end_date:
            kvs[field + '__lte'] = end_date + timedelta(days=1)  # 包含当前
        return self.filter(**kvs)    
    def has_location(self):
        return self.filter(longitude__isnull=False, latitude__isnull=False)
    def unread(self):
        return self.filter(read=False)

# views.py
# 返回未读的报警记录
alarm_list = models.Alarm.objects.unread()
# 返回2016年9月12日到21日的报警记录
alarm_list = models.Alarm.objects.period_date(field='create_time', start_time='2016-09-12', end_time='2016-09-21')
# 返回2016年9月26日以前的报警记录
alarm_list = models.Alarm.objects.period_date(field='create_time', end_time='2016-09-26')
#返回2016年9月26日以前的未读报警记录
alarm_list  = models.Device.objects.period_date(field='create_time', end_time='2016-09-26').unread()
AttributeError: '_QuerySet' object has no attribute 'unread'
```

在最后一个查询中出现异常，因为这两个方法定义在`models.Manager`上，返回的却是`models.QuerySet`实例。这时非常希望自定义的方法能够级联调用，下面的几种方法可以解决这个问题。

#### 3.2.2 使用QuerySet的方法

使用查询集上的 `as_manager()`函数创建新的管理器

将自定义的方法定义从`models.Manager`移到`models.QuerySet`

```
class AlarmQuerySet(models.QuerySet):
    def period_date(self, field, start_date=None, end_date=None, fmt='%Y-%m-%d'):
        # 省略具体代码
        pass
    def has_location(self):
        return self.filter(longitude__isnull=False, latitude__isnull=False)
    def unread(self):
        return self.filter(read=False)

class Alarm(models.Model):
    ...
   objects = AlarmQuerySet.as_manager()
```

这时代码`alarm_list  = models.Device.objects.period_date(field='create_time', end_time='2016-09-26').unread()`就能够返回正确的结果。

#### 3.2.3 继承Manager和QuerySet

使用管理器上的`from_queryset(queryset_class)`函数创建新的管理器
在使用django认证用户上一方面需要继承 `django.contrib.auth.models.BaseUserManager`，另一方面又希望能够自定义函数，这时可以使用这种方式。

```
class UserQuerySet(models.QuerySet):
    def no_login_in_days(self, days):
        start_time = timezone.now() - timedelta(days=days)
        return self.filter(last_login_time__ge=start_time)
    def no_activity_in_days(self, days):
        start_time = timezone.now() - timedelta(days=days)
        return self.filter(last_activity_time__ge=start_time)

MyUserManager = BaseUserManager.from_queryset(UserQuerySet)

class MyUser(AbstractBaseUser):
    objects = MyUserManager()
```
 当直接调用`django.db.models.Manager.from_queryset`方法，其等效于第二种方法，即以下两行等效。

```
# 使用as_manager函数
objects = AlarmQuerySet.as_manager()
# 使用from_queryset函数
objects = models.Manager.from_queryset(AlarmQuerySet)
```

### 3.3 managers模块实践

随着业务逻辑越来越复杂，需要编写更多的自定义管理器，通常的做法是单独创建一个名称为`managers`的模块，封装所有数据操作。

```
from django.db import models
from django.contrib.auth.models import BaseUserManager

__all__ = ['AxxManager', 'BxxManager', 'UserManager'] # 外部模块只能引用XxxManager类


class BaseQuerySet(models.QuerySet):
    def common_method_for_all_models(self):
        pass

class AxxQuerySet(BaseQuerySet):
    pass

AxxManager = models.Manager().from_queryset(AxxQuerySet)

class BxxQuerySet(BaseQuerySet):
    pass

BxxManager = models.Manager().from_queryset(BxxQuerySet)

class UserQuerySet(BaseQuerySet):
    pass

class UserManager(BaseUserManager.from_queryset(UserQuerySet)):
    def create_user(self, username, password=None, **kwargs):
        pass
```

为避免模块循环导入的问题

- 需要使用`django.apps.apps.get_model`函数获取模型类对象，不能直接使用 `from models import Xxxx`
- `managers`模块一般只能被`models`模块引用，其他模块应当不能引用

### 3.4 小提示

**覆盖get函数异常**

在获取对象函数会抛出`ObjectDoesNotExist`异常，在这种情况下我们需要使用`try-catch`捕捉异常，会出现大量重复的代码。这时我们可以用下的代码实现封装。

```
class BaseManager(models.Manager):
    def get_object(self, **kwargs):
        try:
            return self.get(**kwargs)
        except models.ObjectDoesNotExist:
            return None

# 以下方式访问
device = models.objects.get_object(serial='0FFFFFFF66DDF13F')
```

**访问request变量**

按照MVC分离的实践，不应该直接访问request，只能通过参数传递方式。


## 4 迁移

### 4.1 开发流程

迁移是将模型代码的变化应用到数据库，可以认为是一个数据库模式的版本管理系统。

> 在Django1.7之前的版本第三方库[South](http://south.aeracode.org/)提供了类似的功能。

迁移通常可以按照下列步骤循环进行。

- 1 编写 `models` 模块代码
- 2 模型迁移：执行 `python manage.py makemigrations`，在`APP.migrations`包生成迁移模块文件。
- 3 数据迁移：如果需要数据迁移，按照一定的格式编写迁移文件。
- 3 应用迁移：执行 `python manage.py migrate`，将2、3步迁移文件所实现的数据库变化应用到数据库。

Django Migration分为模式迁移（Schema migration）和数据迁移（Data Migration）。

- 模式迁移：包括表结构修改，对应于 SQL的 `CREATE TABLE` `ALTER TABLE`和 `DROP TABLE`，可以由Django自动生成。
- 数据迁移：包括数据记录修改，对应于SQL的 `INSERT TO` `DELETE`和 `UPDATE`等语句,需要开发者自己编写。

无论是模式迁移还是数据迁移，迁移模块都具有几个特点：

- 每个迁移文件是一个Python模块，位于应用目录migrations包下，代表了一次迁移
- 每个迁移包含一个名为Migration的迁移类，该类继承自 `django.db.migrations.Migration`
- `dependencies` 属性表示需要依赖的迁移模块名称
- `operations` 属性表示一系列依次进行的迁移操作，这些都定义在 `django.db.migrations.operations`模块中。

```
from django.db import migrations

class Migration(migrations.Migration):
    dependencies = []
    operations = []
```

### 4.2 数据迁移

下面的例子实现了将设备的`longitude` `latitude`和`address`三个字段复制到报警记录表中，以便查询位置时无需外接操作。

所有的操作被操作 `migrations.RunPython` 类中，要注意的是需要 `django.apps.get_model` 函数引用模型类。

```
from __future__ import unicode_literals

from django.db import migrations, models

def create_address_for_alarm(apps, scheme_editor):
    AlarmClass = apps.get_model('hdc', 'Alarm')
    for alarm in AlarmClass.objects.all():
        alarm.longitude = alarm.device.longitude
        alarm.latitude = alarm.device.latitude
        alarm.address = alarm.device.address
        alarm.save()

class Migration(migrations.Migration):
    dependencies = [
        ('hdc', '0002_alarm_address'),
    ]

    operations = [
         migrations.RunPython(create_address_for_alarm),
    ]
```
