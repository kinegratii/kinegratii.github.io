---
title: 高德地图热力图显示
date: 2017-04-29 18:42:05
categories: 编程
tags:
- 数据可视化
- Javascript
- 地图
---

## 1 热力图

热力图以高亮形式显示数据密集程度。根据密集程度的不同，图上会呈现不同的颜色，以直观的形式展现数据密度。

[AMap.Heatmap](http://lbs.amap.com/api/javascript-api/reference/layer/#m_AMap.Heatmap) 是高德地图热力图插件，基于[heatmapjs](https://www.patrick-wied.at/static/heatmapjs/)。高德地图API引用了heatmap.js最新版本v2.0，v2.0基于新的渲染模型，具有更高的渲染效率和更强的性能。支持chrome、firefox、safari、ie9及以上浏览器。

<!-- more -->

## 2 报警热力图

采用热力图可以直观地显示哪些区域的设备具有很高的报警率，为监控决策提供了数据依据。


### 2.1 后端数据API

API数据范围一段时间内，每个设备的地理位置和报警数目，格式如下：

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



### 2.2 html布局

地图控件(div#id_map_container)使用Bootstrap Panel作为容器，并实现了折叠(collapse)效果。

时间选择，由于业务场景的关系，选择最近一年或者全部时间，时间段太短不具有很好的代表性。根据高德地图自定义控件的设置，将时间选择控件放置在地图控件上，并设置相关css样式。

```html
<div class="row">
    <div class="col-md-12">
        <div class="panel panel-default">
            <div class="panel-heading">
                <strong>报警区域分布图</strong>
            </div>
            <div class="panel-body">
                <div id="id_map_container" style="height: 655px;">
                    <div class="panel panel-default"
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

### 2.3 js实现

- 引入相关库文件。
-  `setDataSet` 参数设置将data设置为后端API返回的数据。
- 同时设置热力图最大最小值。
- 渲染完成后将地图移动到数值最大的点上。

```javascript
<script type="text/javascript"
        src="http://webapi.amap.com/maps?v=1.3&key=API-KEY"></script>
<script type="text/javascript">
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
</script>
```

### 2.4 示例

![amap-heatmap](/images/amap-heatmap.png)
