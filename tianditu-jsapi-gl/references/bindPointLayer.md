# Point 类数据可视化

适用于 **Point** 和 **MultiPoint** 几何类型的 GeoJSON 数据。

## ⚠️ 严格要求

**生成代码时必须完整复制下方模板，只替换 `${DATA_URL}` 为实际URL，禁止修改任何代码逻辑。**

## 完整模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>点数据可视化</title>
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
                    
                    map.addSource("points-source", {
                        type: "geojson",
                        data: geojsonData
                    });

                    map.addLayer({
                        id: "points-layer",
                        type: "circle",
                        source: "points-source",
                        paint: {
                            "circle-radius": 6,
                            "circle-color": "#ff4444",
                            "circle-stroke-color": "#ffffff",
                            "circle-stroke-width": 2
                        }
                    });

                    // 自动定位
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
                    geojsonData.features.forEach(function(f) {
                        if (f.geometry && f.geometry.coordinates) processCoords(f.geometry.coordinates);
                    });
                    if (bounds.minLng !== Infinity) {
                        map.fitBounds([[bounds.minLng, bounds.minLat], [bounds.maxLng, bounds.maxLat]], { padding: 50 });
                    }

                    // 点击弹窗
                    map.on("click", "points-layer", function(e) {
                        if (e.features && e.features.length > 0) {
                            var props = e.features[0].properties;
                            var html = Object.entries(props)
                                .filter(function(p) { return p[1] !== null && p[1] !== ""; })
                                .slice(0, 10)
                                .map(function(p) { return "<b>" + p[0] + ":</b> " + p[1]; })
                                .join("<br>");
                            new TMapGL.Popup({ maxWidth: "300px" })
                                .setLngLat(e.lngLat)
                                .setHTML(html || "无属性信息")
                                .addTo(map);
                        }
                    });

                    map.on("mouseenter", "points-layer", function() { map.getCanvas().style.cursor = "pointer"; });
                    map.on("mouseleave", "points-layer", function() { map.getCanvas().style.cursor = ""; });
                })
                .catch(function(err) {
                    document.getElementById("loading").innerHTML = "加载失败: " + err.message;
                });
        });
    </script>
</body>
</html>
```

## circle 图层样式属性

| 属性 | 说明 | 示例值 |
|------|------|--------|
| circle-radius | 圆点半径 | 6 |
| circle-color | 填充颜色 | "#ff4444" |
| circle-stroke-color | 描边颜色 | "#ffffff" |
| circle-stroke-width | 描边宽度 | 2 |
| circle-opacity | 透明度 | 0.8 |
