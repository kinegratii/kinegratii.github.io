---
title: typescript农历库
date: 2017-01-28 20:27:01
categories: 编程
tags: Typescript
---

本文描述了自己写了一个 Typescript 日历库。

<!-- more -->

## 1 数据结构

一个日期使用 `CalendarDate` 类表示，其属性如下：

```javascript
CalendarDate {
  lYear: 2016,
  lMonth: 11,
  lDay: 8,
  animal: '猴',
  lMonthCn: '冬月',
  lDayCn: '初八',
  sYear: 2016,
  sMonth: 12,
  sDay: 6,
  gzYear: '丙申',
  gzMonth: '己亥',
  gzDay: '壬戌',
  isToday: false,
  isLeap: false,
  nWeek: 2,
  nWeekCn: '星期二',
  isTerm: false,
  term: null,
  astro: '射手座' }
```

## 2 创建日期对象

```javascript
import { CalendarDate,Calendars } from 'calendars';
```

### 2.1 公历转农历

使用 `Calendars.solar2lunar()` 或者 `Calendars.fromSolarDate()` 从农历日期创建 `CalendarDate` 对象。

```javascript
let date:CalendarDate = Calendars.solar2lunar();
date.format('{sYear}年{sMonth}月{sDay}日') // '2016年12月5日'
```

### 2.2 农历转公历

使用 `Calendars.lunar2solar()` 或者 `Calendars.fromLunarDate()` 从农历日期创建 `CalendarDate`对象。

```javascript
let date2:CalendarDate = Calendars.lunar2solar(2017, 6, 1, true); // 2017年农历闰六月初一
date.format('{sYear}年{sMonth}月{sDay}日') // '2017年7月23日'
```

### 2.3 从通用字符串创建

```javascript
let date3:CalendarDate = Calendars.fromDateString('2017060111'); // 2017年农历闰六月初一
date3.format('{sYear}年{sMonth}月{sDay}日') // '2017年7月23日'
```

## 3 工具类

### 3.1 日期间隔

```javascript
// 230，距离今天还有230天
date2.delta()
date2.delta(date)
date2.delta({sYear:2017,sMonth:7,sDay:23})
Calendars.delta(date, date2);
Calendars.delta({sYear:2016,sMonth:12,sDay:5}, {sYear:2017,sMonth:7,sDay:23})
```

### 3.2 日期推算

```javascript
//两天之后

date2.offset(2)
//或
date2.after(2)

//两天之前

date2.offset(-2)
//或
date2.before(2)
```
