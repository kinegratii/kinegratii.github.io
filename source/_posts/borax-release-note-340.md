---
title: Borax-3.4.0发布日志
date: 2020-11-08 14:14:14
categories: 技术研究
tags:
- 项目
---

2020年11月15日，也就是 v3.0.0 版本发布一周年之际，我们很高兴地宣布，Borax v3.4.0 正式发布。

<!-- more -->

# 1 项目开发SOP

## 1.1 全新的项目组织形式

Borax定位于一个由众多实用性功能组成的工具集合库。自v3.4开始，引入“话题/主题，Topic” 的概念，并使用“Borax.Foo” 的字符串标识每一个主题。

每个主题的代码形式可以是模块或包。目前已经被定义的主题如下表：

| 主题                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| Borax.Calendars      | 农历日期库。由 Borax.LunarDate 、Borax.Festivals 和 Borax.Birthday 三部分组成 |
| Borax.Choices        | 基于类定义的选项数据结构                                     |
| Borax.Datasets       | 记录型数据的操作库                                           |
| Borax.DataStructures | 常用的数据结构                                               |
| Borax.Numbers        | 数字处理库，包括中文数字、财务数字。                         |
| Borax.Pattern        | 基于Python实现的设计模式                                     |
| Borax.UI             | 基于tkinter的封装                                            |
| Borax.Utils          | 基础数据类型（字典、列表、对象）操作库。                     |

基于主题的项目组织形式，我们还做了下列的工作：

- 按照主题重新组织Borax文档的编排
- 按照主题的方式重新调整部分模块的归属

## 1.2 发布周期和API废弃策略

Borax 将继续遵循 “[语义化版本2.0](<https://semver.org/lang/zh-CN/>)” 的版本管理策略，即由“<主版本号>.<次版本号>.<修正版本号>”组成。并结合相关情况制定相应的发布策略：

- 其中 *次版本号* 不大于9
- 一年发布两个“次版本系列”，时间是5月和11月。以“2020年11月-v3.4” 为起始点。
- 使用 “主版本废弃策略”，延长废弃进程。即 3.x 被标记为废弃的接口将在 v4.0 移除。

Borax 预计的发布日程表如下：

| 版本   | 发布日期               |
| ------ | ---------------------- |
| v3.4.0 | 2020年11月15日         |
| v3.5.0 | 2021年5月15日（预计）  |
| v3.6.0 | 2021年11月15日（预计） |
| ...    | ...                    |

## 1.3 新的CI工具：Github Action

Borax 3.4开始使用 [Github Action](https://github.com/kinegratii/borax/actions) 代替原有的 [Travis CI]() ，主要原因在于：

- 跟 Github 结合的很好，使用 方便。
- 生态发展很好，支持 step 共享。

新增基于 Codecov 的代码覆盖率显示。

## 1.4 Python版本支持：新增python3.9支持

Borax 新增了对python3.9的构建支持。目前 Borax 支持以下的 python 版本：

```
python3.5 ~ 3.9
```

# 2 Borax.Numbers: 中文数字的四种形式

## 2.1 四种表示形式

根据大小写和使用场景，中文数字可分为不同的形式：

小写所使用的汉字如下：

```
一二三四五六七八九  十百千万亿
```

大写使用的汉字如下：

```
壹贰叁肆伍陆柒捌玖 拾佰仟万亿
```

使用场景指的是数字“0”的表示形式。根据《出版物上数字用法(GB T 15835-2011)》的规定，汉字 “零” 和 “〇” 是有严格的使用场景。

```
阿拉伯数字“0”有“零”和“〇”两种汉字书写形式。一个数字用作计量时，其中“0”的汉字书写形式为“零”，用作编号时，“0”的汉字书写形式为“〇”。

　示例：“3052（个）”的汉字数字形式为“三千零五十二”（不写为“三千〇五十二”）

“95.06”的汉字数字形式为“九十五点零六”（不写为“九十五点〇六”）

“公元2012（年）”的汉字数字形式为“二〇一二”（不写为“二零一二”）

---- 出版物上数字用法(GB T 15835-2011)
```

综上所述，数字204共有4种表示形式。


|               | 大写(Upper) | 小写(Lower) |
| ------------- | ----------- | ----------- |
| 计量(Measure) | 贰佰零肆    | 二百零四    |
| 编号(Order)   | 贰佰〇肆    | 二百〇四    |

财务金额使用的 大写，计量 的表示形式。法律条文中“第XXX条” 使用的是 小写，编号 的表示形式。

## 2.2 ChineseNumber类方法

Borax v3.4 引入了中文数字的计量和编号两种不同用法，对于数字 “0” 使用不同的中文汉字描述。

| 函数                                                         | 用法                                        |
| ------------------------------------------------------------ | ------------------------------------------- |
| ChineseNumbers.to_chinese_number(num:Union[int, str], upper:bool=False, order:bool=False)  -> str | 使用upper控制大小写，使用order控制计量/编号 |
| ChineseNumbers.measure_number(num:Union[int, str])  -> str   | 小写，计量                                  |
| ChineseNumbers.order_number(num:Union[int, str])  -> str     | 小写，编号                                  |

# 3 Borax.Choices: 整合Django.Choices

主要变化如下


- 整合ConstChoices和django choices



# 4 Borax.Calendars

## 4.1 农历日期格式化

strftime函数变化：

| 描述符 | 变更                                                     |
| ------ | -------------------------------------------------------- |
| %t     | 当节气不存在时显示为 `-` 而不是空字符串                  |
| %X     | [新增]月份的另一种形式，将“冬”、“腊”显示为“十一”、“十二” |
|        |                                                          |

描述符的文档，使用 年月日闰及其变种 的形式描述。

## 4.2 获取闰月的年份

函数签名


```python
def LCalendars.get_leap_years(month:int=0) -> tuple
```
函数返回 含有给定闰月的年份列表。当 month=0时，返回所有闰月的年份。当month大于12时，返回空元组。

## 4.3 节气日期

在Borax v3.3可使用 `LCalendars.create_solar_date` 获取公历年特定节气的日期。

```python
def LCalendars.create_solar_date(year: int, term_index: Optional[int] = None, term_name: Optional[str] = None) -> datetime.date
```

由于该方法的 year 参数和所在 `LCalendars` 类的意义有所冲突。Borax v3.4 该方法已废弃，新增 `SCalendars.term2date` ，签名如下：

```python
def SCalendars.term2date(year: int, term_name: Optional[str] = None) -> datetime.date
```

主要变化：

- 移除了 `term_index` 参数
- 参数 `term_name` 支持 `'ChunFeng'` 和 `'春分'` 两种形式

## 4.4 import路径

在Borax v3.4版本中，对开放的类提供了shortcut import path，完整的信息如下：

| 实际引用                             | Shortcut引用               |
| ------------------------------------ | -------------------------- |
| borax.calendars.lundardate.LunarDate | borax.calendars.LunarDate  |
| borax.calendars.lunardate.LCalendars | borax.calendars.LCalendars |
| borax.calendars.utils.SCalendars     | borax.calendars.SCalendars |



# 5 Borax.JSON: JSON序列化

## 5.1 模块组织

Borax V3.4版本将 `bjson` 和 `cjson` 两个模块整合合并，形成全新的 `borax.cjson` 模块。原有的 `bjson` 将被标记为 Deprecated ，并在V4.0中移除。

| V3.3.x                | V3.4.x      | 描述                        |
| --------------------- | ----------- | --------------------------- |
| borax.seriaize.bjson  |             | 实现基于类的 JSON Encoder   |
| borax.serialize.cjson | borax.cjson | 实现基于函数的 JSON Encoder |

> 最终保留 cjson 意味着完全采用基于函数的实现方式。

新的 `borax.serialize.cjson` 的变化如下：

- 默认使用目标类 `__json__` 的编码函数
- 新增 `datetime` / `date` 等常用类JSON编码
- 使用 `cjson.encoder.register` 添加新的编码函数



## 5.2 使用示例



```python
from datetime import date, datetime
from borax.calendars import LunarDate
from borax.serialize import cjson


@cjson.encoder.register(LunarDate)
def encode_ld(ld):
    return ld.strftime('%Y年%L%M月%D日')


data = {'current_time': datetime.now()}
print(cjson.dumps(data))

data2 = {
    'solar_day': date.today(),
    'lunar_day': LunarDate.today()
}
print(cjson.dumps(data2, ensure_ascii=False))
```

输出结果

```
{"current_time": "2020-10-10 10:42:25"}
{"solar_day": "2020-10-10", "lunar_day": "二〇二〇年八月廿四日"}
```

# 6 Borax.SerialNo 基于 Pool 的生成器

`borax.counters.serial_pool` 模块使用新的方式实现序列号生成器。包括以下函数和类：

- serial_no_generator 序列号生成器
- SerialElement 序列号实体类
- SerialNoPool 生成池



```python
from borax.counters.serial_pool import SerialNoPool


# 返回 'LC00000000' ~ 'LC99999999' 的迭代器
pool = SerialNoPool(label_fmt='LC{no}', dights=8)
data = pool.generate_labels(num=3)
print(data) # ['LC00000000', 'LC0000001', 'LC00000002']

pool.add_elements(data)
```
