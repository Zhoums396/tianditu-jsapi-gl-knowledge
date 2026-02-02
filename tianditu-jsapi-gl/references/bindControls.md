# 地图控件

内置控件的添加和配置。

## 控件添加

```javascript
map.addControl(control, position)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| control | Control | 控件实例 |
| position | string | 位置：top-left, top-right, bottom-left, bottom-right |

## 控件移除

```javascript
map.removeControl(control)
```

## 缩放控件

导航控件，包含缩放按钮和指北针。

### 构造函数

```javascript
new TMapGL.NavigationControl(options)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| showZoom | boolean | true | 是否显示缩放按钮 |
| showCompass | boolean | true | 是否显示指北针 |
| visualizePitch | boolean | false | 是否通过指北针控制倾斜 |

### 示例

```javascript
var nav = new TMapGL.NavigationControl({
    showZoom: true,
    showCompass: true,
    visualizePitch: false
});

map.addControl(nav, "top-left");
```

## 全屏控件

### 构造函数

```javascript
new TMapGL.FullscreenControl(options)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| container | HTMLElement | map.getContainer() | 全屏容器 |

### 示例

```javascript
var fullscreen = new TMapGL.FullscreenControl({
    container: map.getContainer()
});

map.addControl(fullscreen, "top-right");
```

## 比例尺控件

### 构造函数

```javascript
new TMapGL.ScaleControl(options)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| maxWidth | number | 100 | 最大宽度（像素） |
| unit | string | "metric" | 单位：metric(米), imperial(英寸), nautical(海里) |

### 示例

```javascript
var scale = new TMapGL.ScaleControl({
    maxWidth: 100,
    unit: "metric"
});

map.addControl(scale, "bottom-left");
```

## 三维地形控件

控制 3D 地形的显示。

### 构造函数

```javascript
new TMapGL.TerrainControl(options)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| source | string | DEM 数据源 ID |
| exaggeration | number | 地形夸张倍数 |

### 示例

```javascript
map.on("load", function() {
    // 添加 DEM 数据源
    map.addSource("raster-dem-source", {
        type: "raster-dem",
        tiles: [
            "https://lcdata.tianditu.gov.cn/terrain-rgb_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=terrain-rgb&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=您的密钥"
        ]
    });

    // 添加山影图层
    map.addLayer({
        id: "hillshade-layer",
        source: "raster-dem-source",
        type: "hillshade",
        paint: {
            "hillshade-illumination-direction": 335,
            "hillshade-exaggeration": 0.5
        }
    });

    // 添加地形控件
    map.addControl(new TMapGL.TerrainControl({
        source: "raster-dem-source",
        exaggeration: 1
    }));
});
```

## 常见场景

### 添加多个控件

```javascript
// 左上角：缩放控件
map.addControl(new TMapGL.NavigationControl(), "top-left");

// 右上角：全屏控件
map.addControl(new TMapGL.FullscreenControl(), "top-right");

// 左下角：比例尺
map.addControl(new TMapGL.ScaleControl(), "bottom-left");
```

### 完整示例

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>地图控件示例</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; }
    </style>
</head>
<body>
    <div id="map"></div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
    <script>
        var map = new TMapGL.Map("map", {
            zoom: 10,
            center: [116.40, 39.90]
        });

        // 缩放控件
        map.addControl(new TMapGL.NavigationControl({
            showZoom: true,
            showCompass: true
        }), "top-left");

        // 全屏控件
        map.addControl(new TMapGL.FullscreenControl(), "top-right");

        // 比例尺
        map.addControl(new TMapGL.ScaleControl({
            maxWidth: 100,
            unit: "metric"
        }), "bottom-left");
    </script>
</body>
</html>
```

## 山影图层

### hillshade 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| hillshade-illumination-direction | number | 335 | 光源方向角度 [0, 359] |
| hillshade-illumination-anchor | string | "viewport" | 光源参照：map/viewport |
| hillshade-exaggeration | number | 0.5 | 强度 [0, 1] |
| hillshade-shadow-color | string | "#000000" | 阴影颜色 |
| hillshade-highlight-color | string | "#FFFFFF" | 高光颜色 |
| hillshade-accent-color | string | "#000000" | 强调色 |

### 山影图层示例

```javascript
map.on("load", function() {
    map.addSource("dem-source", {
        type: "raster-dem",
        tiles: [
            "https://lcdata.tianditu.gov.cn/terrain-rgb_w/wmts?..."
        ]
    });

    map.addLayer({
        id: "hillshade",
        source: "dem-source",
        type: "hillshade",
        paint: {
            "hillshade-illumination-direction": 335,
            "hillshade-illumination-anchor": "map",
            "hillshade-exaggeration": 0.5,
            "hillshade-shadow-color": "#000000",
            "hillshade-highlight-color": "#FFFFFF"
        }
    });
});
```
