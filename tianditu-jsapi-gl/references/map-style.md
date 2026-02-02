# 地图样式

## 个性化地图

天地图支持三种内置样式：

| 样式ID | 说明 |
|--------|------|
| normal | 标准地图（默认） |
| black | 黑色系深色地图 |
| blue | 蓝色系科技风地图 |

### 初始化时设置

```javascript
var map = new TMapGL.Map("map", {
    center: [116.40, 39.90],
    zoom: 12,
    styleId: "black"  // normal | black | blue
});
```

### 黑色地图示例

```javascript
var map = new TMapGL.Map("map", {
    zoom: 10,
    center: [116.35, 39.91],
    styleId: "black"
});
```

### 蓝色地图示例

```javascript
var map = new TMapGL.Map("map", {
    zoom: 10,
    center: [116.35, 39.91],
    styleId: "blue"
});
```

## 地图类型

| 类型 | 说明 |
|------|------|
| vt | 矢量地图（默认） |
| img | 影像地图（卫星图） |
| ter | 地形地图 |

### 设置地图类型

```javascript
// 初始化时设置
var map = new TMapGL.Map("map", {
    center: [116.40, 39.90],
    zoom: 12,
    mapType: "img"  // 影像地图
});

// 动态切换
map.setMapType("ter");  // 切换到地形图
```

### 影像地图

```javascript
var map = new TMapGL.Map("map", {
    zoom: 10,
    center: [116.35, 39.91],
    mapType: "img"
});
```

### 地形地图

```javascript
var map = new TMapGL.Map("map", {
    zoom: 8,
    center: [116.35, 39.91],
    mapType: "ter"
});
```

## 图层显隐控制

控制地图要素的显示与隐藏。

### 可控制的图层

| 图层名 | 说明 |
|--------|------|
| annotation | 注记（地名文字） |
| road | 道路 |
| hyd | 水系 |
| veg | 绿地 |
| resa | 居民地 |
| railway | 铁路 |

### 初始化时设置

```javascript
var map = new TMapGL.Map("map", {
    zoom: 12,
    center: [116.35, 39.91],
    visibility: {
        annotation: true,
        road: true,
        hyd: true,
        veg: true,
        resa: true,
        railway: true
    }
});
```

### 动态控制

```javascript
// 隐藏道路和铁路
map.setVisibility({
    annotation: true,
    road: false,
    hyd: true,
    veg: true,
    resa: true,
    railway: false
});
```

### 完整示例

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>控制地图图层显隐</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; }
        .box { position: absolute; top: 10px; left: 10px; background: white; padding: 10px; }
    </style>
</head>
<body>
    <div id="map"></div>
    <ul class="box">
        <li><input type="checkbox" name="annotation" onclick="toggle(this)" checked /> 注记</li>
        <li><input type="checkbox" name="road" onclick="toggle(this)" checked /> 道路</li>
        <li><input type="checkbox" name="hyd" onclick="toggle(this)" checked /> 水系</li>
        <li><input type="checkbox" name="veg" onclick="toggle(this)" checked /> 绿地</li>
    </ul>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
    <script>
        var visibilities = {
            annotation: true,
            veg: true,
            hyd: true,
            road: true,
            railway: true,
            resa: true
        };

        var map = new TMapGL.Map("map", {
            zoom: 12,
            center: [116.35, 39.91],
            visibility: visibilities
        });

        function toggle(target) {
            var name = target.getAttribute("name");
            visibilities[name] = target.checked;
            map.setVisibility(visibilities);
        }
    </script>
</body>
</html>
```

## 地图投影

天地图支持两种投影方式：

| 投影 | 说明 |
|------|------|
| EPSG:3857 | 墨卡托投影（默认） |
| EPSG:4326 | 经纬度直投 |

### 设置投影

```javascript
// 初始化时设置
var map = new TMapGL.Map("map", {
    center: [116.40, 39.90],
    zoom: 4,
    mapType: "img",
    projection: "EPSG:4326"
});

// 动态切换
map.setProjection("EPSG:3857");
```

### 获取当前投影

```javascript
var projection = map.getProjection();
console.log(projection);  // "EPSG:3857" 或 "EPSG:4326"
```
