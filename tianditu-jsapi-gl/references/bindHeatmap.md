# 热力图

基于点数据密度生成热力图可视化。

## 快速开始

```javascript
map.on("load", function() {
    map.addSource("heatmap-source", {
        type: "geojson",
        data: "points.geojson"
    });

    map.addLayer({
        id: "heatmap-layer",
        type: "heatmap",
        source: "heatmap-source",
        paint: {
            "heatmap-radius": 20,
            "heatmap-opacity": 0.8
        }
    });
});
```

## 热力图配置

### paint 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| heatmap-radius | number | 30 | 热力点影响半径（像素） |
| heatmap-weight | number | 1 | 权重值，可基于属性动态设置 |
| heatmap-intensity | number | 1 | 强度因子 |
| heatmap-color | expression | - | 密度到颜色的映射 |
| heatmap-opacity | number | 1 | 透明度 [0,1] |

### heatmap-color 颜色渐变

```javascript
"heatmap-color": [
    "interpolate",
    ["linear"],
    ["heatmap-density"],
    0, "rgba(33,102,172,0)",     // 密度0：透明
    0.2, "rgb(103,169,207)",     // 密度0.2：浅蓝
    0.4, "rgb(209,229,240)",     // 密度0.4：淡蓝
    0.6, "rgb(253,219,199)",     // 密度0.6：淡橙
    0.8, "rgb(239,138,98)",      // 密度0.8：橙色
    1, "rgb(178,24,43)"          // 密度1：红色
]
```

## 完整示例

### 基础热力图

```javascript
map.on("load", function() {
    map.addSource("geojson-source", {
        type: "geojson",
        data: "http://lbs.tianditu.gov.cn/js-api-v5-portal/geojson/school.geojson"
    });

    map.addLayer({
        id: "school-heat",
        type: "heatmap",
        source: "geojson-source",
        maxzoom: 14,
        paint: {
            // 热力图权重
            "heatmap-weight": 1,
            // 根据级别调整强度
            "heatmap-intensity": [
                "interpolate",
                ["linear"],
                ["zoom"],
                9, 4,
                14, 8
            ],
            // 颜色渐变
            "heatmap-color": [
                "interpolate",
                ["linear"],
                ["heatmap-density"],
                0, "rgba(33,102,172,0)",
                0.2, "rgb(103,169,207)",
                0.4, "rgb(209,229,240)",
                0.6, "rgb(253,219,199)",
                0.8, "rgb(239,138,98)",
                1, "rgb(178,24,43)"
            ],
            // 根据级别调整半径
            "heatmap-radius": [
                "interpolate",
                ["linear"],
                ["zoom"],
                9, 7,
                14, 18
            ],
            // 根据级别调整透明度
            "heatmap-opacity": [
                "interpolate",
                ["linear"],
                ["zoom"],
                12, 1,
                14, 0
            ]
        }
    });
});
```

### 热力图 + 点图层组合

放大后显示具体点位：

```javascript
map.on("load", function() {
    map.addSource("data-source", {
        type: "geojson",
        data: "points.geojson"
    });

    // 热力图层（小级别显示）
    map.addLayer({
        id: "heat-layer",
        type: "heatmap",
        source: "data-source",
        maxzoom: 14,
        paint: {
            "heatmap-weight": 1,
            "heatmap-radius": [
                "interpolate", ["linear"], ["zoom"],
                9, 7,
                14, 18
            ],
            "heatmap-opacity": [
                "interpolate", ["linear"], ["zoom"],
                12, 1,
                14, 0
            ]
        }
    });

    // 点图层（大级别显示）
    map.addLayer({
        id: "point-layer",
        type: "circle",
        source: "data-source",
        minzoom: 12,
        paint: {
            "circle-radius": [
                "interpolate", ["linear"], ["zoom"],
                12, 2,
                16, 18
            ],
            "circle-color": "rgb(178,24,43)",
            "circle-stroke-color": "white",
            "circle-stroke-width": 1,
            "circle-opacity": [
                "interpolate", ["linear"], ["zoom"],
                12, 0,
                13, 1
            ]
        }
    }, "heat-layer");
});
```

### 基于属性的加权热力图

```javascript
map.addLayer({
    id: "weighted-heat",
    type: "heatmap",
    source: "geojson-source",
    paint: {
        // 根据属性值设置权重
        "heatmap-weight": [
            "interpolate",
            ["linear"],
            ["get", "magnitude"],
            0, 0,
            5, 1,
            10, 3
        ],
        "heatmap-radius": 25,
        "heatmap-color": [
            "interpolate",
            ["linear"],
            ["heatmap-density"],
            0, "rgba(0, 0, 255, 0)",
            0.2, "blue",
            0.4, "cyan",
            0.6, "lime",
            0.8, "yellow",
            1, "red"
        ]
    }
});
```
