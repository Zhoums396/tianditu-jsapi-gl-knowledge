# 3D 柱状图数据可视化

使用 fill-extrusion 图层在地图上创建 3D 柱状图，展示城市数据对比。

## 完整示例

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>3D柱状图数据可视化</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; padding: 0; }
        .legend {
            position: absolute;
            bottom: 30px;
            right: 10px;
            background: rgba(255,255,255,0.95);
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
            font-family: Arial, sans-serif;
            font-size: 13px;
        }
        .legend h4 { margin: 0 0 10px 0; font-size: 14px; }
        .legend-item { display: flex; align-items: center; margin: 5px 0; }
        .legend-color { width: 20px; height: 20px; margin-right: 8px; border-radius: 3px; }
    </style>
</head>
<body>
    <div id="map"></div>
    <div class="legend">
        <h4>📊 城市GDP（万亿元）</h4>
        <div class="legend-item"><div class="legend-color" style="background:#ff4444"></div>北京: 4.3</div>
        <div class="legend-item"><div class="legend-color" style="background:#44aaff"></div>上海: 4.7</div>
        <div class="legend-item"><div class="legend-color" style="background:#44dd44"></div>广州: 3.0</div>
        <div class="legend-item"><div class="legend-color" style="background:#ffaa00"></div>深圳: 3.5</div>
    </div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=${TIANDITU_TOKEN}" type="text/javascript"></script>
    <script>
        // 城市数据
        var cityData = [
            { name: "北京", lng: 116.40, lat: 39.90, value: 4.3, color: "#ff4444" },
            { name: "上海", lng: 121.47, lat: 31.23, value: 4.7, color: "#44aaff" },
            { name: "广州", lng: 113.26, lat: 23.13, value: 3.0, color: "#44dd44" },
            { name: "深圳", lng: 114.06, lat: 22.54, value: 3.5, color: "#ffaa00" }
        ];

        // 地图初始化 - 3D视角
        var map = new TMapGL.Map("map", {
            zoom: 5,
            center: [116.0, 30.0],
            pitch: 55,
            bearing: -15
        });

        map.on("load", function() {
            // 为每个城市创建3D柱状图
            cityData.forEach(function(city, index) {
                var size = 0.8;  // 柱子底面大小（经纬度）
                var height = city.value * 80000;  // 柱子高度
                
                // 创建正方形底面
                var polygon = {
                    type: "FeatureCollection",
                    features: [{
                        type: "Feature",
                        properties: { name: city.name, value: city.value },
                        geometry: {
                            type: "Polygon",
                            coordinates: [[
                                [city.lng - size/2, city.lat - size/2],
                                [city.lng + size/2, city.lat - size/2],
                                [city.lng + size/2, city.lat + size/2],
                                [city.lng - size/2, city.lat + size/2],
                                [city.lng - size/2, city.lat - size/2]
                            ]]
                        }
                    }]
                };

                map.addSource("city-" + index, { type: "geojson", data: polygon });

                // 添加3D拉伸图层
                map.addLayer({
                    id: "city-3d-" + index,
                    type: "fill-extrusion",
                    source: "city-" + index,
                    paint: {
                        "fill-extrusion-color": city.color,
                        "fill-extrusion-height": height,
                        "fill-extrusion-base": 0,
                        "fill-extrusion-opacity": 0.9
                    }
                });
            });

            // 添加城市名称标注
            var labelData = {
                type: "FeatureCollection",
                features: cityData.map(function(city) {
                    return {
                        type: "Feature",
                        properties: { name: city.name + "\n" + city.value + "万亿" },
                        geometry: { type: "Point", coordinates: [city.lng, city.lat] }
                    };
                })
            };

            map.addSource("city-labels", { type: "geojson", data: labelData });
            map.addLayer({
                id: "city-label-layer",
                type: "symbol",
                source: "city-labels",
                layout: {
                    "text-field": ["get", "name"],
                    "text-font": ["Microsoft YaHei"],
                    "text-size": 14,
                    "text-offset": [0, -3],
                    "text-anchor": "bottom"
                },
                paint: {
                    "text-color": "#333333",
                    "text-halo-color": "#ffffff",
                    "text-halo-width": 2
                }
            });

            // 导航控件
            map.addControl(new TMapGL.NavigationControl({
                showZoom: true,
                showCompass: true,
                visualizePitch: true
            }), "top-right");
        });
    </script>
</body>
</html>
```

## fill-extrusion 图层属性

| 属性 | 类型 | 说明 |
|------|------|------|
| fill-extrusion-color | string | 柱子颜色 |
| fill-extrusion-height | number | 柱子高度（米） |
| fill-extrusion-base | number | 柱子底部高度（米） |
| fill-extrusion-opacity | number | 透明度 0-1 |

## 关键点

1. **3D 视角**：设置 `pitch: 55` 倾斜角度，`bearing` 旋转角度
2. **柱子底面**：使用 Polygon 定义正方形底面
3. **高度映射**：`fill-extrusion-height` 与数据值成正比
4. **图例**：使用 HTML+CSS 创建悬浮图例
5. **标注**：使用 symbol 图层添加城市名称
