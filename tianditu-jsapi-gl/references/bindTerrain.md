# 3D 地形图

创建带有山影效果和3D倾斜视角的地形地图。

## 完整示例

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>3D地形图</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; padding: 0; }
    </style>
</head>
<body>
    <div id="map"></div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=${TIANDITU_TOKEN}" type="text/javascript"></script>
    <script>
        // 地图初始化 - 使用地形底图和3D视角
        var map = new TMapGL.Map("map", {
            zoom: 10,
            center: [103.0, 30.0],  // 四川山区
            pitch: 60,              // 倾斜角度（0-85度）
            bearing: 30,            // 旋转角度
            style: "terrain"        // 地形底图样式
        });
        
        map.on("load", function() {
            // 添加地形数据源
            map.addSource("terrain-dem", {
                type: "raster-dem",
                tiles: [
                    "https://lcdata.tianditu.gov.cn/terrain-rgb_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=terrain-rgb&TILEMATRIXSET=w&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&FORMAT=image/png"
                ],
                tileSize: 256
            });
            
            // 设置地形
            map.setTerrain({ 
                source: "terrain-dem",
                exaggeration: 1.5  // 地形夸张系数
            });
            
            // 添加山影图层
            map.addLayer({
                id: "hillshade-layer",
                type: "hillshade",
                source: "terrain-dem",
                paint: {
                    "hillshade-illumination-direction": 335,
                    "hillshade-illumination-anchor": "map",
                    "hillshade-exaggeration": 0.5,
                    "hillshade-shadow-color": "#000000",
                    "hillshade-highlight-color": "#ffffff"
                }
            });
            
            // 添加导航控件（支持3D旋转）
            map.addControl(new TMapGL.NavigationControl({
                showZoom: true,
                showCompass: true,
                visualizePitch: true  // 重要：启用倾斜控制
            }), "top-right");
            
            // 添加比例尺
            map.addControl(new TMapGL.ScaleControl(), "bottom-left");
        });
    </script>
</body>
</html>
```

## 关键参数

### 地图初始化参数

| 参数 | 类型 | 说明 |
|------|------|------|
| pitch | number | 倾斜角度，0-85度，0为垂直俯视 |
| bearing | number | 旋转角度，0-360度 |
| style | string | 底图样式，"terrain"为地形底图 |

### 地形设置

```javascript
map.setTerrain({
    source: "terrain-dem",    // 地形数据源ID
    exaggeration: 1.5         // 地形夸张系数，1为真实比例
});
```

### 山影图层样式

| 属性 | 类型 | 说明 |
|------|------|------|
| hillshade-illumination-direction | number | 光源方向（0-359度） |
| hillshade-exaggeration | number | 强度（0-1） |
| hillshade-shadow-color | string | 阴影颜色 |
| hillshade-highlight-color | string | 高光颜色 |

## 交互控制

```javascript
// 动态改变视角
map.flyTo({
    center: [103.0, 30.0],
    zoom: 12,
    pitch: 70,
    bearing: 45
});

// 重置为俯视视角
map.easeTo({
    pitch: 0,
    bearing: 0
});
```

## 推荐区域

展示3D地形效果较好的区域：
- 四川山区：[103.0, 30.0]
- 青藏高原：[91.1, 29.6]
- 云南山区：[100.2, 25.0]
- 新疆天山：[87.5, 43.5]
- 太行山区：[113.5, 37.0]
