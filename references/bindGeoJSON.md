# GeoJSON 数据绑定

加载和展示 GeoJSON 数据的完整指南。

## ⚠️ 重要：可视化必须自动定位到数据范围

**生成 GeoJSON 可视化代码时，必须：**
1. 使用 `fetch` 加载数据后计算边界
2. 调用 `map.fitBounds()` 自动定位到数据范围
3. 用户才能看到数据！否则地图可能显示在数据范围之外

## 🔥 标准可视化模板（必须遵循）

### 多边形/面数据可视化（Polygon/MultiPolygon）

```javascript
map.on("load", function() {
    fetch("GeoJSON数据URL")
        .then(function(res) { return res.json(); })
        .then(function(geojsonData) {
            // 1. 添加数据源
            map.addSource("geojson-source", {
                type: "geojson",
                data: geojsonData
            });

            // 2. 添加填充图层
            map.addLayer({
                id: "geojson-fill",
                type: "fill",
                source: "geojson-source",
                paint: {
                    "fill-color": "#3388ff",
                    "fill-opacity": 0.5
                }
            });

            // 3. 添加边框图层
            map.addLayer({
                id: "geojson-line",
                type: "line",
                source: "geojson-source",
                paint: {
                    "line-color": "#333333",
                    "line-width": 1
                }
            });

            // 4. 【必须】计算边界并自动定位
            // 使用简单的 min/max 方式计算边界，避免 LngLatBounds 的问题
            var minLng = Infinity, maxLng = -Infinity;
            var minLat = Infinity, maxLat = -Infinity;
            
            // 递归提取所有坐标点
            function extractCoords(arr) {
                if (Array.isArray(arr) && arr.length >= 2 && typeof arr[0] === "number" && typeof arr[1] === "number") {
                    // 这是一个坐标点 [lng, lat]
                    minLng = Math.min(minLng, arr[0]);
                    maxLng = Math.max(maxLng, arr[0]);
                    minLat = Math.min(minLat, arr[1]);
                    maxLat = Math.max(maxLat, arr[1]);
                } else if (Array.isArray(arr)) {
                    // 递归处理嵌套数组
                    arr.forEach(extractCoords);
                }
            }
            
            geojsonData.features.forEach(function(feature) {
                if (feature.geometry && feature.geometry.coordinates) {
                    extractCoords(feature.geometry.coordinates);
                }
            });
            
            // 使用计算出的边界定位地图
            if (minLng !== Infinity) {
                map.fitBounds([
                    [minLng, minLat],  // 西南角
                    [maxLng, maxLat]   // 东北角
                ], { padding: 50 });
            }

            // 5. 点击弹窗
            map.on("click", "geojson-fill", function(e) {
                var props = e.features[0].properties;
                var html = "<div style='max-width:300px;'>";
                for (var key in props) {
                    if (props[key] != null) {
                        html += "<b>" + key + ":</b> " + props[key] + "<br>";
                    }
                }
                html += "</div>";
                new TMapGL.Popup()
                    .setLngLat(e.lngLat)
                    .setHTML(html)
                    .addTo(map);
            });

            // 6. 鼠标指针
            map.on("mouseenter", "geojson-fill", function() {
                map.getCanvas().style.cursor = "pointer";
            });
            map.on("mouseleave", "geojson-fill", function() {
                map.getCanvas().style.cursor = "";
            });
        });
});
```

### 点数据可视化（Point）

```javascript
map.on("load", function() {
    fetch("GeoJSON数据URL")
        .then(function(res) { return res.json(); })
        .then(function(geojsonData) {
            // 1. 添加数据源
            map.addSource("points-source", {
                type: "geojson",
                data: geojsonData
            });

            // 2. 添加圆点图层
            map.addLayer({
                id: "points-layer",
                type: "circle",
                source: "points-source",
                paint: {
                    "circle-radius": 8,
                    "circle-color": "#ff6b6b",
                    "circle-stroke-width": 2,
                    "circle-stroke-color": "#ffffff"
                }
            });

            // 3. 【必须】计算边界并自动定位
            var minLng = Infinity, maxLng = -Infinity;
            var minLat = Infinity, maxLat = -Infinity;
            
            geojsonData.features.forEach(function(feature) {
                if (feature.geometry && feature.geometry.coordinates) {
                    var coords = feature.geometry.coordinates;
                    minLng = Math.min(minLng, coords[0]);
                    maxLng = Math.max(maxLng, coords[0]);
                    minLat = Math.min(minLat, coords[1]);
                    maxLat = Math.max(maxLat, coords[1]);
                }
            });
            
            if (minLng !== Infinity) {
                map.fitBounds([
                    [minLng, minLat],
                    [maxLng, maxLat]
                ], { padding: 50 });
            }

            // 4. 点击弹窗
            map.on("click", "points-layer", function(e) {
                var props = e.features[0].properties;
                var html = "<div>";
                for (var key in props) {
                    if (props[key] != null) {
                        html += "<b>" + key + ":</b> " + props[key] + "<br>";
                    }
                }
                html += "</div>";
                new TMapGL.Popup()
                    .setLngLat(e.lngLat)
                    .setHTML(html)
                    .addTo(map);
            });
        });
});
```

### 线数据可视化（LineString）

```javascript
map.on("load", function() {
    fetch("GeoJSON数据URL")
        .then(function(res) { return res.json(); })
        .then(function(geojsonData) {
            map.addSource("lines-source", {
                type: "geojson",
                data: geojsonData
            });

            map.addLayer({
                id: "lines-layer",
                type: "line",
                source: "lines-source",
                paint: {
                    "line-color": "#3388ff",
                    "line-width": 3
                }
            });

            // 【必须】计算边界并自动定位
            var minLng = Infinity, maxLng = -Infinity;
            var minLat = Infinity, maxLat = -Infinity;
            
            geojsonData.features.forEach(function(feature) {
                if (feature.geometry && feature.geometry.coordinates) {
                    feature.geometry.coordinates.forEach(function(coord) {
                        if (Array.isArray(coord) && coord.length >= 2) {
                            minLng = Math.min(minLng, coord[0]);
                            maxLng = Math.max(maxLng, coord[0]);
                            minLat = Math.min(minLat, coord[1]);
                            maxLat = Math.max(maxLat, coord[1]);
                        }
                    });
                }
            });
            
            if (minLng !== Infinity) {
                map.fitBounds([
                    [minLng, minLat],
                    [maxLng, maxLat]
                ], { padding: 50 });
            }
        });
});
```

## 分类着色（根据属性字段设置不同颜色）

```javascript
// 使用 match 表达式根据属性值设置颜色
"fill-color": [
    "match",
    ["get", "ZONE"],           // 属性字段名
    "100年洪水区", "#0000ff",   // 值 -> 颜色
    "500年洪水区", "#00ff00",
    "洪水道", "#ff0000",
    "#888888"                   // 默认颜色
]
```

## 添加图例

```html
<style>
    .legend {
        position: absolute;
        bottom: 20px;
        right: 20px;
        background: white;
        padding: 10px;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0,0,0,0.2);
        z-index: 1;
    }
    .legend-item { display: flex; align-items: center; margin-bottom: 5px; }
    .legend-color { width: 20px; height: 20px; margin-right: 8px; }
</style>

<div class="legend">
    <h3>图例标题</h3>
    <div class="legend-item">
        <div class="legend-color" style="background: #0000ff;"></div>
        <span>类别1</span>
    </div>
    <div class="legend-item">
        <div class="legend-color" style="background: #00ff00;"></div>
        <span>类别2</span>
    </div>
</div>
```

---

## 基础用法

## 数据源配置

### addSource

```javascript
map.addSource(sourceId, sourceOptions)
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sourceId | string | 是 | 数据源唯一 ID |
| sourceOptions | object | 是 | 数据源配置 |

### sourceOptions

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| type | string | - | 必填，固定为 "geojson" |
| data | string \| object | - | 必填，GeoJSON URL 或对象 |
| generateId | boolean | false | 自动为每个要素生成唯一 ID |
| promoteId | string | "" | 用作要素唯一标识的属性名 |
| lineMetrics | boolean | false | 是否计算线长度（线渐变需要） |

### 使用 URL 加载

```javascript
map.addSource("geojson-source", {
    type: "geojson",
    data: "http://lbs.tianditu.gov.cn/js-api-v5-portal/geojson/point.geojson"
});
```

### 使用 GeoJSON 对象

```javascript
map.addSource("geojson-source", {
    type: "geojson",
    data: {
        type: "FeatureCollection",
        features: [
            {
                type: "Feature",
                geometry: {
                    type: "Point",
                    coordinates: [116.40, 39.90]
                },
                properties: {
                    name: "北京",
                    population: 21540000
                }
            }
        ]
    }
});
```

## 图层类型

### 点图层 (circle)

```javascript
map.addLayer({
    id: "point-layer",
    type: "circle",
    source: "geojson-source",
    minzoom: 0,
    maxzoom: 24,
    filter: ["==", ["get", "type"], "school"],
    layout: {
        visibility: "visible",
        "circle-sort-key": ["get", "importance"]
    },
    paint: {
        "circle-radius": 10,
        "circle-color": "#ff0000",
        "circle-blur": 0,
        "circle-opacity": 1,
        "circle-stroke-width": 2,
        "circle-stroke-color": "#ffffff",
        "circle-stroke-opacity": 1,
        "circle-translate": [0, 0],
        "circle-translate-anchor": "map",
        "circle-pitch-scale": "map",
        "circle-pitch-alignment": "viewport"
    }
});
```

### circle 属性说明

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| circle-radius | number | 5 | 圆半径（像素） |
| circle-color | string | "#000000" | 填充颜色 |
| circle-blur | number | 0 | 模糊程度 |
| circle-opacity | number | 1 | 透明度 [0,1] |
| circle-stroke-width | number | 0 | 边框宽度 |
| circle-stroke-color | string | "#000000" | 边框颜色 |
| circle-stroke-opacity | number | 1 | 边框透明度 |
| circle-translate | Array | [0,0] | 偏移 [x,y] |
| circle-pitch-scale | string | "map" | 倾斜时缩放基准 |
| circle-pitch-alignment | string | "viewport" | 倾斜时朝向 |

### 线图层 (line)

```javascript
map.addLayer({
    id: "line-layer",
    type: "line",
    source: "geojson-source",
    layout: {
        "line-cap": "round",
        "line-join": "round",
        "line-miter-limit": 2,
        visibility: "visible"
    },
    paint: {
        "line-opacity": 1,
        "line-color": "#ff0000",
        "line-width": 4,
        "line-gap-width": 0,
        "line-offset": 0,
        "line-blur": 0,
        "line-dasharray": [4, 2],
        "line-translate": [0, 0],
        "line-translate-anchor": "map"
    }
});
```

### line 属性说明

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| line-cap | string | "butt" | 端点形状：butt/round/square |
| line-join | string | "miter" | 连接处形状：bevel/round/miter |
| line-color | string | "#000000" | 线颜色 |
| line-width | number | 1 | 线宽（像素） |
| line-opacity | number | 1 | 透明度 |
| line-dasharray | Array | - | 虚线样式 [实线长, 空隙长] |
| line-gap-width | number | 0 | 内部缝隙宽度 |
| line-offset | number | 0 | 偏移量 |
| line-blur | number | 0 | 模糊半径 |

### 面图层 (fill)

```javascript
map.addLayer({
    id: "fill-layer",
    type: "fill",
    source: "geojson-source",
    layout: {
        "fill-sort-key": ["get", "area"],
        visibility: "visible"
    },
    paint: {
        "fill-antialias": true,
        "fill-opacity": 0.7,
        "fill-color": "#ffff00",
        "fill-outline-color": "#ff0000",
        "fill-translate": [0, 0],
        "fill-translate-anchor": "map"
    }
});
```

### fill 属性说明

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| fill-antialias | boolean | true | 抗锯齿 |
| fill-opacity | number | 1 | 透明度 |
| fill-color | string | "#000000" | 填充颜色 |
| fill-outline-color | string | - | 边框颜色（宽度固定1px） |
| fill-pattern | string | - | 填充纹理图片名称 |
| fill-translate | Array | [0,0] | 偏移 [x,y] |

### 文本图层 (symbol - text)

```javascript
map.addLayer({
    id: "text-layer",
    type: "symbol",
    source: "geojson-source",
    filter: ["has", "name"],
    layout: {
        "symbol-placement": "point",
        "text-field": ["get", "name"],
        "text-font": ["WenQuanYi Micro Hei Mono"],
        "text-size": 14,
        "text-max-width": 10,
        "text-anchor": "center",
        "text-offset": [0, 0],
        "text-allow-overlap": false,
        visibility: "visible"
    },
    paint: {
        "text-opacity": 1,
        "text-color": "#333333",
        "text-halo-color": "#ffffff",
        "text-halo-width": 2,
        "text-halo-blur": 0
    }
});
```

### 图标图层 (symbol - icon)

```javascript
// 先加载图片
map.loadImage("icon.png").then(function(image) {
    map.addImage("my-icon", image.data);

    map.addLayer({
        id: "icon-layer",
        type: "symbol",
        source: "geojson-source",
        layout: {
            "icon-image": "my-icon",
            "icon-size": 1,
            "icon-anchor": "bottom",
            "icon-rotate": 0,
            "icon-allow-overlap": true
        },
        paint: {
            "icon-opacity": 1
        }
    });
});
```

## 数据源操作

### 获取数据源

```javascript
var source = map.getSource("my-source");
```

### 更新数据

```javascript
var source = map.getSource("my-source");
source.setData(newGeoJSON);
```

### 移除数据源

```javascript
// 先移除依赖的图层
map.removeLayer("my-layer");
// 再移除数据源
map.removeSource("my-source");
```

## 图层操作

### 添加图层

```javascript
map.addLayer(layerOptions);

// 在指定图层之前插入
map.addLayer(layerOptions, "beforeLayerId");
```

### 获取图层

```javascript
var layer = map.getLayer("my-layer");
```

### 移除图层

```javascript
map.removeLayer("my-layer");
```

### 图层显隐

```javascript
// 隐藏图层
map.setLayoutProperty("my-layer", "visibility", "none");

// 显示图层
map.setLayoutProperty("my-layer", "visibility", "visible");
```

### 动态修改样式

```javascript
// 修改填充颜色
map.setPaintProperty("my-layer", "fill-color", "#00ff00");

// 修改圆半径
map.setPaintProperty("point-layer", "circle-radius", 15);
```

## 表达式

使用表达式实现数据驱动的样式。

### 获取属性值

```javascript
["get", "propertyName"]
```

### 条件过滤

```javascript
// 相等
filter: ["==", ["get", "type"], "school"]

// 大于
filter: [">", ["get", "population"], 1000000]

// 组合条件
filter: ["all",
    ["==", ["get", "type"], "city"],
    [">", ["get", "population"], 500000]
]
```

### 分级样式

```javascript
paint: {
    "circle-color": [
        "case",
        ["<", ["get", "value"], 30], "#00ff00",
        ["<", ["get", "value"], 70], "#ffff00",
        "#ff0000"
    ],
    "circle-radius": [
        "interpolate", ["linear"], ["get", "population"],
        10000, 4,
        100000, 8,
        1000000, 16
    ]
}
```

## 完整示例

### 加载并展示 GeoJSON

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>GeoJSON 示例</title>
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
            center: [114.29, 30.58]
        });

        map.on("load", function() {
            // 添加数据源
            map.addSource("points", {
                type: "geojson",
                data: "http://lbs.tianditu.gov.cn/js-api-v5-portal/geojson/point.geojson"
            });

            // 添加点图层
            map.addLayer({
                id: "points-layer",
                type: "circle",
                source: "points",
                paint: {
                    "circle-radius": 8,
                    "circle-color": "#ff6b6b",
                    "circle-stroke-width": 2,
                    "circle-stroke-color": "#ffffff"
                }
            });

            // 点击要素显示信息
            map.on("click", "points-layer", function(e) {
                var feature = e.features[0];
                new TMapGL.Popup()
                    .setLngLat(e.lngLat)
                    .setHTML("<h3>" + feature.properties.name + "</h3>")
                    .addTo(map);
            });

            // 鼠标悬停效果
            map.on("mouseenter", "points-layer", function() {
                map.getCanvas().style.cursor = "pointer";
            });
            map.on("mouseleave", "points-layer", function() {
                map.getCanvas().style.cursor = "";
            });
        });
    </script>
</body>
</html>
```

### 多图层组合

```javascript
map.on("load", function() {
    // 添加面数据源
    map.addSource("regions", {
        type: "geojson",
        data: "regions.geojson"
    });

    // 面填充图层
    map.addLayer({
        id: "regions-fill",
        type: "fill",
        source: "regions",
        paint: {
            "fill-color": "#627BC1",
            "fill-opacity": 0.5
        }
    });

    // 面边框图层
    map.addLayer({
        id: "regions-outline",
        type: "line",
        source: "regions",
        paint: {
            "line-color": "#627BC1",
            "line-width": 2
        }
    });

    // 标注图层
    map.addLayer({
        id: "regions-label",
        type: "symbol",
        source: "regions",
        layout: {
            "text-field": ["get", "name"],
            "text-size": 12
        },
        paint: {
            "text-color": "#333333",
            "text-halo-color": "#ffffff",
            "text-halo-width": 1
        }
    });
});
```
