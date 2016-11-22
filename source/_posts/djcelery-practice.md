---
title: djcelery实践
date: 2015-10-22 10:17:40
categories:
- 编程
tags:
- Celery
---

Celery是一个异步任务队列/基于分布式消息传递的作业队列。Celery通过消息（message）进行通信，使用代理（broker）在客户端和工作执行者之间进行交互。当开始一个任务时，客户端发送消息到队列并由代理将其发往响应的工作执行者处。。djcelery是其和Django框架一个很方便使用的第三方包。

<!-- more -->

## 1 环境配置

### 1.1 安装ERLang

1. 首先是到ERLang官网去下载ERlang可执行文件  地址：http://www.erlang.org/download.html
2. 然后安装ERLang。
3. 然后设置ERLang的环境变量。
4. 在环境变量中加入 `ERL_HOME = erlang安装目录`
5. 在path中添加 `%ERL_HOME%\bin`

### 1.2 安装rabbitmq

从http://www.rabbitmq.com/releases/rabbitmq-server/v3.1.5/rabbitmq-server-3.1.5.exe下载rabbitmq-server 并安装。

 点击开始菜单中的 rabbitmq-start ，rabbitmq-server就启动了，在管理工具-服务中可以看到相关信息的。


### 1.3 安装Djcelery

和大多数Python第三方包一样，用 pip安装celery和djcelery两个包。djcelery依赖于djcelery，所以只要执行pip install djcelery命令即可。

## 2 配置Djcelery

主要步骤

- 在settings配置相关参数
- 定义任务
- 执行任务，可以在程序中调用执行，也可交给后台周期性执行

### 2.1 基本配置

下面是Djcelery的有关配置，定义在Django项目的settings模块内。

```python
#... ...

INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Uncomment the next line to enable the admin:
    'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    'djcelery',  #添加djcelery
     'mrs_app',  #自己的APP
)

# ... ....
# 配置djcelery相关参数，ResultStore默认存储在数据库可不必重写 ，
import djcelery
djcelery.setup_loader()
BROKER_URL = 'amqp://guest:guest@localhost:5672//'
#任务定义所在的模块
CELERY_IMPORTS = ('mrs_app.my_celery.tasks', )
# 使用和Django一样的时区
CELERY_TIMEZONE = TIME_ZONE

#以上为基本配置，以下为周期性任务定义，以celerybeat_开头的  

CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'

#CELERYBEAT_SCHEDULE = {
#    'add-every-3-minutes': {
#        'task': 'mrs_app.my_celery.tasks.monthly_reading_task',
#        'schedule': timedelta(minutes=3)
#    },
#}
```

## 2.2 定义任务

有两种格式

- 类定义：一个继承了celery.app.task的类并实现了run方法
- 函数定义：@task装饰的函数

通过实现task相关方法可以实现更多的逻辑，比如成功回调、错误处理、重试机制等，以下是最基本的定义方式。

```python
#mrs_app.my_celery.tasks.py

from celery import task

```python

#第一种，函数方式  
 @task(name='monthly_reading')
def monthly_reading_task():
    task_obj = MonthlyReading(debug=False)
    task_obj.start()

#第二种，类定义
class MonthlyReadingTask(Task):
    name='monthly_reading'
    def run(*args, **kwargs):
        task_obj = MonthlyReading(debug=False)
        task_obj.start()
```

### 2.3 启动

- 启动 `python manage.py celery worker -l info`
- 如果有定时任务的话，还需要启动心跳
 - 另开一个cmd窗口 `python manage.py celery beat`  （windows下-B选项不可用）

## 3 执行任务

### 3.1 直接调用

自己在代码中的调用，支持延迟/同步/异步调用，可参考task类定义，例子见参考资料的《使用django+celery+RabbitMQ实现异步执行》。

### 3.2 周期性调用

这种由djcelery调用，所以需要在`settings.CELERYBEAT_SCHEDULER`设置一个调度器，这里使用数据库。

djcelery提供了一些Model（定义在`djcelery/models.py`文件）

说明：

- 任务和定时任务的区别：定时任务 = 任务 + intervalschedule/crontabschedule 。两个定时任务可以执行同一个任务。
- 任务没有相应的Model，用字符串表示，即periodictask模型的task字段
- 定时任务有相应的Model即periodictask。
- djcelery在初始化中主要完成两件：

- 在`settings.CELERY_IMPORTS`定义下的模块搜索所有任务。这个对数据库没有任何改变，只是用Admin添加定时任务时periodictask.task字段变成选择框，列出了所有定义的任务。

- 从settings.CELERYBEAT_SCHEDULE创建定时任务，这个会创建数据记录，相当于`celery_models.PeriodicTask.objects.create(..)`语句。

### 3.3 创建定时任务

通过它提供的Model Query API来操作，同平常的数据库查询一样。

```python
from djcelery import models as celery_models

celery_models.PeriodicTask.objects.create(...)
celery_models.PeriodicTask.ojects.get(name='add')
....
```
djcelery提供了admin管理界面，访问`http://localhost:8000/admin/djcelery/` 即可，在这里可以对定时任务进行增删改查，具体和Django admin一样。

注：当我们修改任务的设置后，比如关闭、更改时间后不用重启celery服务等。
