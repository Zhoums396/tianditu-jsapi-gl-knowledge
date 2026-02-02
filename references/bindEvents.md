# 地图事件

地图和图层的事件监听与处理。

## 事件监听

### 添加事件监听

```javascript
map.on(eventType, callback)
map.on(eventType, layerId, callback)  // 指定图层
```

### 移除事件监听

```javascript
map.off(eventType, callback)
map.off(eventType, layerId, callback)  // 指定图层
```

### 一次性事件

```javascript
map.once(eventType, callback)
```

## 地图事件类型

| 事件 | 说明 |
|------|------|
| load | 地图加载完成 |
| click | 鼠标点击 |
| dblclick | 鼠标双击 |
| contextmenu | 鼠标右键 |
| mousedown | 鼠标按下 |
| mouseup | 鼠标抬起 |
| mousemove | 鼠标移动 |
| mouseenter | 鼠标进入地图 |
| mouseleave | 鼠标离开地图 |
| wheel | 鼠标滚轮 |
| move | 地图移动 |
| movestart | 地图移动开始 |
| moveend | 地图移动结束 |
| zoom | 地图缩放 |
| zoomstart | 地图缩放开始 |
| zoomend | 地图缩放结束 |
| rotate | 地图旋转 |
| rotatestart | 地图旋转开始 |
| rotateend | 地图旋转结束 |
| pitch | 地图倾斜 |
| pitchstart | 地图倾斜开始 |
| pitchend | 地图倾斜结束 |
| drag | 地图拖拽 |
| dragstart | 地图拖拽开始 |
| dragend | 地图拖拽结束 |
| resize | 地图容器尺寸变化 |

## 事件对象

### 鼠标事件对象

```javascript
{
    lngLat: { lng: 116.40, lat: 39.90 },  // 经纬度坐标
    point: { x: 100, y: 150 },             // 像素坐标
    originalEvent: MouseEvent,              // 原生事件
    features: [...]                         // 图层事件时的要素列表
}
```

## 常见场景

### 地图加载完成

```javascript
map.on("load", function() {
    console.log("地图加载完成");
    // 在这里添加数据源和图层
});
```

### 点击地图获取坐标

```javascript
map.on("click", function(e) {
    console.log("经度:", e.lngLat.lng);
    console.log("纬度:", e.lngLat.lat);
    console.log("像素坐标:", e.point.x, e.point.y);
});
```

### 右键菜单

```javascript
map.on("contextmenu", function(e) {
    console.log("右键位置:", e.lngLat.lng, e.lngLat.lat);
    // 显示自定义右键菜单
});
```

### 监听地图移动

```javascript
map.on("moveend", function() {
    var center = map.getCenter();
    var zoom = map.getZoom();
    console.log("当前中心:", center.lng, center.lat);
    console.log("当前级别:", zoom);
});
```

### 监听缩放

```javascript
map.on("zoomend", function() {
    var zoom = map.getZoom();
    console.log("当前级别:", zoom);
});
```

## 图层事件

### 点击图层要素

```javascript
// 添加图层
map.addLayer({
    id: "point-layer",
    type: "circle",
    source: "my-source",
    paint: {
        "circle-radius": 10,
        "circle-color": "#ff0000"
    }
});

// 监听图层点击
map.on("click", "point-layer", function(e) {
    var feature = e.features[0];
    console.log("点击的要素:", feature);
    console.log("属性:", feature.properties);
    
    // 显示弹窗
    new TMapGL.Popup()
        .setLngLat(e.lngLat)
        .setHTML("<h3>" + feature.properties.name + "</h3>")
        .addTo(map);
});
```

### 鼠标悬停效果

```javascript
// 鼠标进入图层
map.on("mouseenter", "point-layer", function() {
    map.getCanvas().style.cursor = "pointer";
});

// 鼠标离开图层
map.on("mouseleave", "point-layer", function() {
    map.getCanvas().style.cursor = "";
});
```

### 高亮悬停要素

```javascript
var hoveredId = null;

map.on("mousemove", "point-layer", function(e) {
    if (e.features.length > 0) {
        // 取消之前的高亮
        if (hoveredId !== null) {
            map.setFeatureState(
                { source: "my-source", id: hoveredId },
                { hover: false }
            );
        }
        
        // 高亮当前要素
        hoveredId = e.features[0].id;
        map.setFeatureState(
            { source: "my-source", id: hoveredId },
            { hover: true }
        );
    }
});

map.on("mouseleave", "point-layer", function() {
    if (hoveredId !== null) {
        map.setFeatureState(
            { source: "my-source", id: hoveredId },
            { hover: false }
        );
    }
    hoveredId = null;
});

// 在图层样式中使用 feature-state
map.addLayer({
    id: "point-layer",
    type: "circle",
    source: "my-source",
    paint: {
        "circle-radius": [
            "case",
            ["boolean", ["feature-state", "hover"], false],
            15,  // 悬停时
            10   // 默认
        ],
        "circle-color": [
            "case",
            ["boolean", ["feature-state", "hover"], false],
            "#ff6b6b",  // 悬停时
            "#ff0000"   // 默认
        ]
    }
});
```

## 动态注册/注销

```javascript
function clickHandler(e) {
    console.log("点击:", e.lngLat);
}

// 注册事件
function enableClick() {
    map.on("click", clickHandler);
}

// 注销事件
function disableClick() {
    map.off("click", clickHandler);
}
```

## 完整示例

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>地图事件示例</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; }
        .info-box {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: white;
            padding: 10px;
            border-radius: 4px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>
    <div id="map"></div>
    <div class="info-box">
        <p>中心: <span id="center"></span></p>
        <p>级别: <span id="zoom"></span></p>
        <p>点击: <span id="click"></span></p>
    </div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
    <script>
        var map = new TMapGL.Map("map", {
            zoom: 10,
            center: [116.40, 39.90]
        });

        var centerEl = document.getElementById("center");
        var zoomEl = document.getElementById("zoom");
        var clickEl = document.getElementById("click");

        // 更新信息
        function updateInfo() {
            var center = map.getCenter();
            centerEl.textContent = center.lng.toFixed(4) + ", " + center.lat.toFixed(4);
            zoomEl.textContent = map.getZoom().toFixed(2);
        }

        // 地图加载完成
        map.on("load", updateInfo);

        // 地图移动结束
        map.on("moveend", updateInfo);

        // 点击地图
        map.on("click", function(e) {
            clickEl.textContent = e.lngLat.lng.toFixed(4) + ", " + e.lngLat.lat.toFixed(4);
        });
    </script>
</body>
</html>
```
