---
title: 高德地图热力图显示
date: 2017-04-29 18:42:05
categories: 编程
tags:
- Javascript
- 地图
---


使用 AMap.Heatmap 插件实现热力图显示。

<!-- more -->

## html代码

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
                            <a data-toggle="collapse" href="#id_map_body">时间段设置</a>
                        </div>
                        <div id="id_map_body" class="collapse in">
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

## js代码

```javascript
<script type="text/javascript"
        src="http://webapi.amap.com/maps?v=1.3&key=ed1fafa0307bb4991da41f54d8a88b46"></script>
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

## 示例

![amap-heatmap](/images/amap-heatmap.png)
