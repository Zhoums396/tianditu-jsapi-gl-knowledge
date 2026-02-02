# 点聚合

将密集的点数据自动聚合，提升可视化效果和性能。

## 快速开始（使用内联示例数据）

当没有用户上传的数据时，使用以下模板生成带有模拟示例数据的完整代码：

```javascript
map.on("load", function() {
    // 示例数据：北京市周边的随机点位
    var sampleData = {
        type: "FeatureCollection",
        features: []
    };
    
    // 生成随机点位数据（以北京为中心，100个点）
    var centerLng = 116.40;
    var centerLat = 39.90;
    for (var i = 0; i < 100; i++) {
        sampleData.features.push({
            type: "Feature",
            geometry: {
                type: "Point",
                coordinates: [
                    centerLng + (Math.random() - 0.5) * 0.5,
                    centerLat + (Math.random() - 0.5) * 0.5
                ]
            },
            properties: {
                name: "点位 " + (i + 1),
                category: ["餐饮", "酒店", "景点", "学校"][Math.floor(Math.random() * 4)]
            }
        });
    }
    
    map.addSource("cluster-source", {
        type: "geojson",
        data: sampleData,
        cluster: true,
        clusterMaxZoom: 14,
        clusterRadius: 50
    });

    // 聚合点（分级颜色）
    map.addLayer({
        id: "clusters",
        type: "circle",
        source: "cluster-source",
        filter: ["has", "point_count"],
        paint: {
            "circle-color": [
                "step",
                ["get", "point_count"],
                "#51bbd6",   // < 10
                10, "#f1f075", // 10-30
                30, "#f28cb1"  // >= 30
            ],
            "circle-radius": [
                "step",
                ["get", "point_count"],
                15,     // < 10
                10, 20, // 10-30
                30, 25  // >= 30
            ]
        }
    });

    // 聚合数量标签
    map.addLayer({
        id: "cluster-count",
        type: "symbol",
        source: "cluster-source",
        filter: ["has", "point_count"],
        layout: {
            "text-field": "{point_count_abbreviated}",            "text-font": ["Microsoft YaHei"],            "text-size": 12
        },
        paint: {
            "text-color": "#ffffff"
        }
    });

    // 未聚合的点
    map.addLayer({
        id: "unclustered-point",
        type: "circle",
        source: "cluster-source",
        filter: ["!", ["has", "point_count"]],
        paint: {
            "circle-color": "#11b4da",
            "circle-radius": 6,
            "circle-stroke-width": 1,
            "circle-stroke-color": "#ffffff"
        }
    });

    // 点击聚合点展开
    map.on("click", "clusters", async function(e) {
        var features = map.queryRenderedFeatures(e.point, { layers: ["clusters"] });
        var clusterId = features[0].properties.cluster_id;
        var zoom = await map.getSource("cluster-source").getClusterExpansionZoom(clusterId);
        map.flyTo({ center: features[0].geometry.coordinates, zoom: zoom });
    });

    // 鼠标样式
    map.on("mouseenter", "clusters", function() { map.getCanvas().style.cursor = "pointer"; });
    map.on("mouseleave", "clusters", function() { map.getCanvas().style.cursor = ""; });
});
```

## 数据源配置

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| cluster | boolean | false | 是否开启聚合 |
| clusterRadius | number | 50 | 聚合半径（像素） |
| clusterMaxZoom | number | - | 最大聚合级别 |
| clusterMinPoints | number | 2 | 最小聚合点数 |
| clusterProperties | object | - | 聚合属性统计 |

## 聚合样式

### 分级颜色

根据聚合数量设置不同颜色：

```javascript
map.addLayer({
    id: "clusters",
    type: "circle",
    source: "cluster-source",
    filter: ["has", "point_count"],
    paint: {
        "circle-color": [
            "step",
            ["get", "point_count"],
            "#51bbd6",   // 默认颜色（< 30）
            30, "#f1f075", // 30-60
            60, "#f28cb1"  // >= 60
        ],
        "circle-radius": [
            "step",
            ["get", "point_count"],
            20,     // 默认半径（< 30）
            30, 30, // 30-60
            60, 40  // >= 60
        ]
    }
});
```

### 聚合数量标签

```javascript
map.addLayer({
    id: "cluster-count",
    type: "symbol",
    source: "cluster-source",
    filter: ["has", "point_count"],
    layout: {
        "text-field": "{point_count_abbreviated}",
        "text-font": ["WenQuanYi Micro Hei Mono"],
        "text-size": 12
    },
    paint: {
        "text-color": "#ffffff"
    }
});
```

## 交互操作

### 点击聚合点展开

```javascript
map.on("click", "clusters", async function(e) {
    var features = map.queryRenderedFeatures(e.point, {
        layers: ["clusters"]
    });
    var clusterId = features[0].properties.cluster_id;
    
    // 获取下一级聚合的缩放级别
    var zoom = await map.getSource("cluster-source")
        .getClusterExpansionZoom(clusterId);
    
    // 飞行到该位置
    map.flyTo({
        center: features[0].geometry.coordinates,
        zoom: zoom
    });
});
```

### 获取聚合内的要素

```javascript
map.on("click", "clusters", async function(e) {
    var features = map.queryRenderedFeatures(e.point, {
        layers: ["clusters"]
    });
    var clusterId = features[0].properties.cluster_id;
    
    // 获取聚合内的所有要素
    var children = await map.getSource("cluster-source")
        .getClusterLeaves(clusterId, 100, 0);
    
    console.log("聚合内的要素:", children);
});
```

## 完整示例

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>点聚合示例</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; }
    </style>
</head>
<body>
    <div id="map"></div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
    <script>
        var map = new TMapGL.Map("map", {
            zoom: 9,
            center: [116.35, 39.91]
        });

        map.on("load", function() {
            map.addSource("geojson-source", {
                type: "geojson",
                data: "http://lbs.tianditu.gov.cn/js-api-v5-portal/geojson/school.geojson",
                cluster: true,
                clusterMaxZoom: 14,
                clusterRadius: 40
            });

            // 聚合点图层
            map.addLayer({
                id: "clusters",
                type: "circle",
                source: "geojson-source",
                filter: ["has", "point_count"],
                paint: {
                    "circle-color": [
                        "step",
                        ["get", "point_count"],
                        "#51bbd6",
                        30, "#f1f075",
                        60, "#f28cb1"
                    ],
                    "circle-radius": [
                        "step",
                        ["get", "point_count"],
                        20,
                        30, 30,
                        60, 40
                    ]
                }
            });

            // 聚合数量文本
            map.addLayer({
                id: "cluster-count",
                type: "symbol",
                source: "geojson-source",
                filter: ["has", "point_count"],
                layout: {
                    "text-field": "{point_count_abbreviated}",
                    "text-font": ["WenQuanYi Micro Hei Mono"],
                    "text-size": 12
                }
            });

            // 未聚合的点
            map.addLayer({
                id: "unclustered-point",
                type: "circle",
                source: "geojson-source",
                filter: ["!", ["has", "point_count"]],
                paint: {
                    "circle-color": "#11b4da",
                    "circle-radius": 4,
                    "circle-stroke-width": 1,
                    "circle-stroke-color": "#fff"
                }
            });

            // 点击聚合点展开
            map.on("click", "clusters", async function(e) {
                var features = map.queryRenderedFeatures(e.point, {
                    layers: ["clusters"]
                });
                var clusterId = features[0].properties.cluster_id;
                var zoom = await map.getSource("geojson-source")
                    .getClusterExpansionZoom(clusterId);
                map.flyTo({
                    center: features[0].geometry.coordinates,
                    zoom: zoom
                });
            });

            // 鼠标样式
            map.on("mouseenter", "clusters", function() {
                map.getCanvas().style.cursor = "pointer";
            });
            map.on("mouseleave", "clusters", function() {
                map.getCanvas().style.cursor = "";
            });
        });
    </script>
</body>
</html>
```
