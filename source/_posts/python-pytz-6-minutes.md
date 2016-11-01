---
title: pytz东八区6分钟问题
date: 2015-11-12 21:47:00
categories:
 - 编程
tags:
 - pytz
---

本文介绍了pytz库中 `Asia/Shanghai`时区相差6分钟的问题。

<!-- more -->

之前也一直没有注意到这个问题，最近的项目需要同时显示DTU设备时间和服务器时间，才发现设备时间总是少6-7分钟。
在项目中，两者解析成datetime对象的使用方法不一样：

- 服务器用的是Django框架，使用了django.utils.timezone.now函数解析服务器当前时间
- DTU则是自己通过构造函数创建的，时区用的是pytz.timezone('Asia/Shanghai')

首先在一篇文章[《用datetime和pytz来转换时区》](http://www.keakon.net/2010/12/14/%E7%94%A8datetime%E5%92%8Cpytz%E6%9D%A5%E8%BD%AC%E6%8D%A2%E6%97%B6%E5%8C%BA)中说可以用台北时间（Asia/Taipei），试验下发现台北时间也有6分钟的问题。为了测试通过临时强制加上了6分钟，才勉强通过测试，然而这不是长久之计。

晚上下班时回家用“pytz 6分钟”搜索发现了[《python中pytz,东8区,6分钟问题 - 老楠老楠》](http://www.laonan.net/blog/G681_FUNEeWTK_79rf__Vw/)这篇文章。**根据文章的描述，用localize函数就可以了。**

使用datetime直接构造时间的时候，设置时区是没有北京时间的，一般来说习惯了linux的同志都会默认用上海时间来代替，这里却有一个问题，如果要进行时区转换，上海时间比北京时间差6分钟。。。

比如：

```
tz = pytz.timezone('Asia/Shanghai')
t = datetime.datetime(2015, 9, 5, 9, 0, 0, 0, tzinfo=tz)
```
这样打印出来得到的时间是：

```
2015-09-07 09:00:00+08:06
```

在django框架中，貌似from django.utils.timezone import localtime的这个localtime会修正那6分钟，这问题就来了，要自己在程序里构造时间，并且跟用这个localtime转化的时间对比的时候巨麻烦。
