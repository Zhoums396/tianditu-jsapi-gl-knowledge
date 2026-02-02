# GeoJSON 数据可视化

## ⚠️ 严格要求

**生成代码时必须：**
1. **完整复制下方模板**，只替换 `${DATA_URL}` 为实际URL
2. **禁止修改模板中的任何代码逻辑**
3. **禁止删除 filter、processCoords、showPopup 等任何部分**

## 🔥 完整模板（直接复制使用）

将 `${DATA_URL}` 替换为实际的 GeoJSON 数据 URL。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>GeoJSON 可视化</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; padding: 0; }
        .loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            padding: 20px 30px;
            border-radius: 8px;
            box-shadow: 0 2px 12px rgba(0,0,0,0.2);
            z-index: 1000;
        }
    </style>
</head>
<body>
    <div id="map"></div>
    <div id="loading" class="loading">正在加载数据...</div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=${TIANDITU_TOKEN}" type="text/javascript"></script>
    <script>
        var map = new TMapGL.Map("map", {
            zoom: 4,
            center: [108, 34]
        });

        map.on("load", function() {
            fetch("${DATA_URL}")
                .then(function(res) { return res.json(); })
                .then(function(geojsonData) {
                    document.getElementById("loading").style.display = "none";
                    
                    // 添加数据源
                    map.addSource("geojson-source", {
                        type: "geojson",
                        data: geojsonData
                    });

                    // 多边形填充层（Polygon + MultiPolygon）
                    map.addLayer({
                        id: "geojson-fill",
                        type: "fill",
                        source: "geojson-source",
                        filter: ["any", 
                            ["==", ["geometry-type"], "Polygon"],
                            ["==", ["geometry-type"], "MultiPolygon"]
                        ],
                        paint: {
                            "fill-color": "#3388ff",
                            "fill-opacity": 0.5
                        }
                    });

                    // 线图层（Polygon边界 + LineString + MultiLineString）
                    map.addLayer({
                        id: "geojson-line",
                        type: "line",
                        source: "geojson-source",
                        filter: ["any",
                            ["==", ["geometry-type"], "Polygon"],
                            ["==", ["geometry-type"], "MultiPolygon"],
                            ["==", ["geometry-type"], "LineString"],
                            ["==", ["geometry-type"], "MultiLineString"]
                        ],
                        paint: {
                            "line-color": "#1a73e8",
                            "line-width": 2
                        }
                    });

                    // 点图层（Point + MultiPoint）
                    map.addLayer({
                        id: "geojson-point",
                        type: "circle",
                        source: "geojson-source",
                        filter: ["any",
                            ["==", ["geometry-type"], "Point"],
                            ["==", ["geometry-type"], "MultiPoint"]
                        ],
                        paint: {
                            "circle-radius": 6,
                            "circle-color": "#ff4444",
                            "circle-stroke-color": "#ffffff",
                            "circle-stroke-width": 2
                        }
                    });

                    // 自动定位到数据范围
                    var bounds = { minLng: Infinity, maxLng: -Infinity, minLat: Infinity, maxLat: -Infinity };
                    function processCoords(coords) {
                        if (!Array.isArray(coords)) return;
                        if (typeof coords[0] === "number" && typeof coords[1] === "number") {
                            bounds.minLng = Math.min(bounds.minLng, coords[0]);
                            bounds.maxLng = Math.max(bounds.maxLng, coords[0]);
                            bounds.minLat = Math.min(bounds.minLat, coords[1]);
                            bounds.maxLat = Math.max(bounds.maxLat, coords[1]);
                        } else {
                            coords.forEach(processCoords);
                        }
                    }
                    geojsonData.features.forEach(function(feature) {
                        if (feature.geometry && feature.geometry.coordinates) {
                            processCoords(feature.geometry.coordinates);
                        }
                    });
                    if (bounds.minLng !== Infinity) {
                        map.fitBounds([[bounds.minLng, bounds.minLat], [bounds.maxLng, bounds.maxLat]], { padding: 50 });
                    }

                    // 点击弹窗显示属性
                    function showPopup(e) {
                        if (e.features && e.features.length > 0) {
                            var props = e.features[0].properties;
                            var content = Object.entries(props)
                                .filter(function(p) { return p[1] !== null && p[1] !== ""; })
                                .slice(0, 10)
                                .map(function(p) { return "<b>" + p[0] + ":</b> " + p[1]; })
                                .join("<br>");
                            new TMapGL.Popup({ maxWidth: "300px" })
                                .setLngLat(e.lngLat)
                                .setHTML(content || "无属性信息")
                                .addTo(map);
                        }
                    }
                    map.on("click", "geojson-fill", showPopup);
                    map.on("click", "geojson-point", showPopup);
                    map.on("click", "geojson-line", showPopup);

                    // 鼠标悬停效果
                    ["geojson-fill", "geojson-point", "geojson-line"].forEach(function(layerId) {
                        map.on("mouseenter", layerId, function() {
                            map.getCanvas().style.cursor = "pointer";
                        });
                        map.on("mouseleave", layerId, function() {
                            map.getCanvas().style.cursor = "";
                        });
                    });
                })
                .catch(function(error) {
                    document.getElementById("loading").innerHTML = "加载失败: " + error.message;
                });
        });
    </script>
</body>
</html>
```

## 图层类型对照表

| GeoJSON 几何类型 | 使用的图层类型 | 说明 |
|-----------------|---------------|------|
| Point | circle | 圆点 |
| MultiPoint | circle | 多个圆点 |
| LineString | line | 线 |
| MultiLineString | line | 多条线 |
| Polygon | fill + line | 填充 + 边框 |
| MultiPolygon | fill + line | 多个多边形 |

## 使用 filter 区分几何类型

```javascript
// 只显示点
filter: ["any", ["==", ["geometry-type"], "Point"], ["==", ["geometry-type"], "MultiPoint"]]

// 只显示线
filter: ["any", ["==", ["geometry-type"], "LineString"], ["==", ["geometry-type"], "MultiLineString"]]

// 只显示面
filter: ["any", ["==", ["geometry-type"], "Polygon"], ["==", ["geometry-type"], "MultiPolygon"]]
```

## 递归坐标处理函数

处理任意深度嵌套的坐标数组：

```javascript
function processCoords(coords) {
    if (!Array.isArray(coords)) return;
    if (typeof coords[0] === "number") {
        // 单个坐标点 [lng, lat]
        // 在这里处理坐标
    } else {
        coords.forEach(processCoords);
    }
}
```
