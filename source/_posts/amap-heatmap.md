---
title: 高德地图热力图和设备监测
date: 2017-04-29 18:42:05
categories: 编程
tags:
- 数据可视化
- Javascript
- 地图
---

## A 热力图


热力图以高亮形式显示数据密集程度。根据密集程度的不同，图上会呈现不同的颜色，以直观的形式展现数据密度。

在设备检测领域，采用热力图可以直观地显示哪些区域的设备具有很高的报警率，为监控决策和提前介入提供了数据依据。

[AMap.Heatmap](http://lbs.amap.com/api/javascript-api/reference/layer/#m_AMap.Heatmap) 是高德地图热力图插件，基于[heatmapjs](https://www.patrick-wied.at/static/heatmapjs/)。高德地图API引用了heatmap.js最新版本v2.0，v2.0基于新的渲染模型，具有更高的渲染效率和更强的性能。支持chrome、firefox、safari、ie9及以上浏览器。

<!-- more -->

## B 后端数据API

根据热力图的文档，后端API需要返回的数据格式如下：

```json
[
    {
        "device_serial": "F023D02900010002",
        "lng": 119.368489,
        "lat": 25.729161,
        "address": "XXX",
        "count": 830
    },
    {
        "device_serial": "F023D02900010003",
        "lng": 119.53378,
        "lat": 26.206372,
        "address": "XXX",
        "count": 220
    }
]
```

总体为一个列表，每个元素表示一个设备，包括了设备序列号、设备经纬度、地址和报警数目。

## C 页面布局设计

页面布局包括两大部分：

- 地图控件(div#id_map_container)，放置地图的控件，必须设置其高度。
- 时间选择器， 使用 `position: absolute;z-index: 2;`等样式，将时间段选择控件(div#id_time_radio_panel)以绝对定位方式放置在地图控件的右上角。

最外层使用Bootstrap Panel作为容器，并实现了折叠(collapse)效果。

> 时间选择器只提供了 “最近一年”和“全部”两个时段，时间段太短数据量偏少，不具有很好的代表性。

```html
<div class="row">
    <div class="col-md-12">
        <div class="panel panel-default">
            <div class="panel-heading">
                <strong>报警区域分布图</strong>
            </div>
            <div class="panel-body">
                <div id="id_map_container" style="height: 655px;">
                    <div  id="id_time_radio_panel" class="panel panel-default"
                         style="position: absolute;width:23%;z-index: 2;top:5px;right: 5px;">
                        <div class="panel-heading">
                            <a data-toggle="collapse" href="#id_time_radio">时间段设置</a>
                        </div>
                        <div id="id_time_radio" class="collapse in">
                            <div class="panel-body">
                                <input type="radio" name="timeDelta" value="365d" checked/>最近一年&nbsp;
                                <input type="radio" name="timeDelta" value=""/>全部时间
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

## D js实现

主要步骤如下：

1. 首先引入高德地图js库文件，使用注册好的API KEY。
2. 通过 `isSupportCanvas` 判断是否支持canvas，否则提示相关信息。
3. 创建地图对象，并初始化工具插件。
4. 编写时间选择器切换响应函数，在函数中请求远程数据，并渲染热力图层。
5. `setDataSet` 除了数据链表外，还需要设置热力图数值最大最小值。
6. 渲染完成后将地图移动到数值最大的点上。

```javascript
    function loadHeatmapData() {
        var heatmap;
        $.get('/api/alarm/heatmap/?timeDelta=' + $("input[name=timeDelta]:checked").val(), function (data) {
            gMapObj.plugin(["AMap.Heatmap"], function () {
                //初始化heatmap对象
                heatmap = new AMap.Heatmap(gMapObj, {
                    radius: 20,
                    opacity: [0, 0.8]
                });
                var maxVal = 0, minVal = 10000;
                var cIndex = -1;
                for (var i = 0; i < data.length; i++) {
                    if (data[i].count > maxVal) {
                        maxVal = data[i].count;
                        cIndex = i;
                    }
                    if (data[i].count < minVal) {
                        minVal = data[i].count;
                    }
                }
                heatmap.setDataSet({
                    data: data,
                    max: maxVal,
                    min: minVal
                });
                if (cIndex > -1) {
                    gMapObj.setCenter(new AMap.LngLat(data[cIndex].lng, data[cIndex].lat));
                }
            });
        });
    }
    function isSupportCanvas() {
        var elem = document.createElement('canvas');
        return !!(elem.getContext && elem.getContext('2d'));
    }
    var gMapObj = new AMap.Map("id_map_container", {
        zoom: 15
    });
    AMap.plugin(['AMap.ToolBar', 'AMap.Scale'], function () {
        gMapObj.addControl(new AMap.ToolBar());
        gMapObj.addControl(new AMap.Scale());
        gMapObj.addControl(new AMap.OverView());
    });
    if (!isSupportCanvas()) {
        alert("'热力图仅对支持canvas的浏览器适用,您所使用的浏览器不能使用热力图功能,请换个浏览器试试");
    } else {
        loadHeatmapData();
        $(":radio").click(function () {
            loadHeatmapData();
        });
    }
```

## E 示例

这是系统经过一个月运行后生成的热力图，虽然数据量还是偏少，但设备之间还是有很好的区分度，比如国惠大酒店旁的设备报警次数就比其他多了几个等级。

![amap-heatmap](/images/amap-heatmap.png)
