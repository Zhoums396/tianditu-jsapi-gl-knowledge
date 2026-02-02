# 覆盖物

圆形、掩膜等覆盖物的绑定。

## 画圆（指定半径米数）

绘制固定地理距离（米）的圆形，随地图缩放保持真实大小。

### ⚠️ 重要警告

**`circle-radius` 的单位是像素，不是米！**

- ❌ 错误：`"circle-radius": 5000` （这是5000像素，会覆盖整个屏幕）
- ✅ 正确：必须用 `metersToPixels()` 函数将米转换为像素

### 核心公式（必须使用）

```javascript
/**
 * 【必须使用】将米转换为指定缩放级别的像素半径
 * @param {number} meters - 半径（米）
 * @param {number} lat - 圆心纬度
 * @param {number} zoom - 地图缩放级别
 * @returns {number} 像素半径
 */
function metersToPixels(meters, lat, zoom) {
    var earthCircumference = 40075016.686;
    var metersPerPixel = earthCircumference * Math.cos(lat * Math.PI / 180) / Math.pow(2, zoom + 8);
    return meters / metersPerPixel;
}
```

### 完整示例（5公里圆）- 必须完全按此模式

```javascript
var circleCenter = [104.07, 30.67];  // 成都
var radiusMeters = 5000;  // 5公里 = 5000米

// 【必须定义】米转像素的函数
function metersToPixels(meters, lat, zoom) {
    var earthCircumference = 40075016.686;
    var metersPerPixel = earthCircumference * Math.cos(lat * Math.PI / 180) / Math.pow(2, zoom + 8);
    return meters / metersPerPixel;
}

map.on("load", function() {
    var minZoom = map.getMinZoom();
    var zoom = map.getZoom();
    var maxZoom = map.getMaxZoom();
    var lat = circleCenter[1];

    // 【必须】计算各级别的像素半径
    var circleRadius = {
        stops: [
            [minZoom, metersToPixels(radiusMeters, lat, minZoom)],
            [zoom, metersToPixels(radiusMeters, lat, zoom)],
            [maxZoom, metersToPixels(radiusMeters, lat, maxZoom)]
        ],
        base: 2
    };

    // 添加圆心数据源
    map.addSource("circle-source", {
        type: "geojson",
        data: {
            type: "Feature",
            geometry: {
                type: "Point",
                coordinates: circleCenter
            }
        }
    });

    // 添加圆形图层 - 注意 circle-radius 用的是上面计算的对象，不是数字！
    map.addLayer({
        id: "circle-layer",
        type: "circle",
        source: "circle-source",
        paint: {
            "circle-radius": circleRadius,  // 这里用对象，不是直接写数字！
            "circle-color": "#ff0000",
            "circle-opacity": 0.3,
            "circle-stroke-color": "#ff0000",
            "circle-stroke-width": 2,
            "circle-pitch-alignment": "map"
        }
    });

    // 添加圆心标记
    var markerEl = document.createElement("div");
    markerEl.style.width = "12px";
    markerEl.style.height = "12px";
    markerEl.style.backgroundColor = "#ff0000";
    markerEl.style.borderRadius = "50%";
    markerEl.style.border = "2px solid #fff";
    
    new TMapGL.Marker({ element: markerEl })
        .setLngLat(circleCenter)
        .addTo(map);
});
```

### 注意事项

1. **radiusMeters 单位是米**：5公里 = 5000米
2. **必须定义 metersToPixels 函数**
3. **circle-radius 必须用 stops 对象**，不能直接写数字
2. **必须用 metersToPixels 函数**：将米转换为各缩放级别的像素值
3. **stops 数组**：定义不同缩放级别对应的像素半径，保证圆的地理大小不变
```

## 区域掩膜

突出显示指定区域，遮盖其他区域。

### 原理

使用带洞的多边形实现掩膜效果：外环覆盖全球，内环是需要显示的区域。

### 示例

```javascript
map.on("load", function() {
    // 带洞的面：外环覆盖全球，内环是显示区域
    var maskGeoJSON = {
        type: "Feature",
        geometry: {
            type: "Polygon",
            coordinates: [
                // 外环：覆盖全球
                [
                    [-180, 90],
                    [180, 90],
                    [180, -90],
                    [-180, -90],
                    [-180, 90]
                ],
                // 内环：显示区域（天安门广场）
                [
                    [116.40042, 39.906667],
                    [116.41163, 39.906667],
                    [116.41163, 39.914099],
                    [116.40042, 39.914099],
                    [116.40042, 39.906667]
                ]
            ]
        }
    };

    map.addSource("mask-source", {
        type: "geojson",
        data: maskGeoJSON
    });

    map.addLayer({
        id: "mask-layer",
        type: "fill",
        source: "mask-source",
        paint: {
            "fill-color": "#ffffff",
            "fill-opacity": 0.8
        }
    });
});
```

### 动态更新掩膜

```javascript
// 更新显示区域
function updateMaskArea(bounds) {
    var maskGeoJSON = {
        type: "Feature",
        geometry: {
            type: "Polygon",
            coordinates: [
                [[-180, 90], [180, 90], [180, -90], [-180, -90], [-180, 90]],
                [
                    [bounds.west, bounds.south],
                    [bounds.east, bounds.south],
                    [bounds.east, bounds.north],
                    [bounds.west, bounds.north],
                    [bounds.west, bounds.south]
                ]
            ]
        }
    };
    
    map.getSource("mask-source").setData(maskGeoJSON);
}

// 使用示例
updateMaskArea({
    west: 116.35,
    south: 39.85,
    east: 116.45,
    north: 39.95
});
```

## 完整示例

### 画圆示例

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>画圆示例</title>
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
            center: [116.35, 39.91]
        });

        var circleCenter = [116.35, 39.91];
        var radius = 100;

        function getPixelRadius(z, radius) {
            var centerPoint = map.project(circleCenter);
            var radiusX = centerPoint.x + radius;
            var radiusLngLat = map.unproject([radiusX, centerPoint.y]);
            var centerLngLat = new TMapGL.LngLat(circleCenter[0], circleCenter[1]);
            var meter = centerLngLat.distanceTo(radiusLngLat);
            var ratio = 40075020 / Math.pow(2, z) / 512;
            return meter / ratio / Math.cos((circleCenter[1] * Math.PI) / 180);
        }

        map.on("load", function() {
            var minZoom = map.getMinZoom();
            var zoom = map.getZoom();
            var maxZoom = map.getMaxZoom();

            var circleRadius = {
                stops: [
                    [minZoom, getPixelRadius(minZoom, radius)],
                    [zoom, radius],
                    [maxZoom, getPixelRadius(maxZoom, radius)]
                ],
                base: 2
            };

            map.addSource("circle-source", {
                type: "geojson",
                data: {
                    type: "Feature",
                    geometry: {
                        type: "Point",
                        coordinates: circleCenter
                    }
                }
            });

            map.addLayer({
                id: "circle-layer",
                type: "circle",
                source: "circle-source",
                paint: {
                    "circle-radius": circleRadius,
                    "circle-color": "#ff0000",
                    "circle-opacity": 0.2,
                    "circle-stroke-color": "#ff0000",
                    "circle-stroke-width": 2,
                    "circle-pitch-alignment": "map"
                }
            });
        });
    </script>
</body>
</html>
```

### 掩膜示例

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>区域掩膜示例</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; }
    </style>
</head>
<body>
    <div id="map"></div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
    <script>
        var map = new TMapGL.Map("map", {
            zoom: 14,
            center: [116.406025, 39.910382]
        });

        map.on("load", function() {
            var maskGeoJSON = {
                type: "Feature",
                geometry: {
                    type: "Polygon",
                    coordinates: [
                        [[-180, 90], [180, 90], [180, -90], [-180, -90], [-180, 90]],
                        [
                            [116.40042, 39.906667],
                            [116.41163, 39.906667],
                            [116.41163, 39.914099],
                            [116.40042, 39.914099],
                            [116.40042, 39.906667]
                        ]
                    ]
                }
            };

            map.addSource("mask-source", {
                type: "geojson",
                data: maskGeoJSON
            });

            map.addLayer({
                id: "mask-layer",
                type: "fill",
                source: "mask-source",
                paint: {
                    "fill-color": "#ffffff",
                    "fill-opacity": 0.8
                }
            });
        });
    </script>
</body>
</html>
```
