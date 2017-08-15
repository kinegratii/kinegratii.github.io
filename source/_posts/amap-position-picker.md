---
title: 【项目】高德地图位置选择器
date: 2017-08-09 21:08:12
categories: 技术研究
tags:
- 项目
- 高德地图
---

基于高德地图的位置选择jQuery插件。该项目结合了个人相关开发经验，分离功能独立的构件，严格遵守javascript开发规范。现已收录于开源中国中，主页为 https://www.oschina.net/p/amappositionpicker 。

<!-- more -->

> 从v0.9.0起，项目名称由 bootstrap.AMapPositionPicker 更改为 AMapPositionPicker。


主要特性有：

- AMD & CMD 引入
- `data-*` 属性配置
- 初始位置数据
- 浏览器定位
- 字段显示格式、验证
- 数据控件绑定
- 支持地理逆编码
- POI搜索
- 工具：显示点标记


## 基本使用


1 依次引入高德地图JS、jQuery、Bootstrap和bootstrap.AMapPositionPicker.min.js文件。

```html
<script type="text/javascript" src="http://webapi.amap.com/maps?v=1.3&key=您申请的key值"></script>
<script type="text/javascript" src="http://cdn.bootcss.com/jquery/1.11.1/jquery.min.js"></script>
<script type="text/javascript" src="http://cdn.bootcss.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
<script type="text/javascript" src="./dist/bootstrap.AMapPositionPicker.min.js"></script>
```

2 在目标输入框初始化选项。

html代码

```html
<input type="text" id="id_address_input" name="address"/>
```

JS代码

```javascript
$("#id_address_input").AMapPositionPicker();
```

更多示例可查看 [文档&示例](http://kinegratii.oschina.io/bootstrap-amappositionpicker/index.html)。

## 项目开发

在开发过程中，参考了[Eonasdan / bootstrap-datetimepicker](https://github.com/Eonasdan/bootstrap-datetimepicker)等项目的模块结构。遵循标准通用的代码结构，以适应于多种环境。

### 构建

项目使用gulp工具构建。

生成 release 文件

```
gulp release
```
