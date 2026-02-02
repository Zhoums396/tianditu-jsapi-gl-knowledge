# 第三方图层服务

加载 WMS、WMTS、TMS 等第三方地图服务。

## WMS 服务

Web Map Service 标准服务。

### 数据源配置

```javascript
map.addSource("wms-source", {
    type: "raster",
    tiles: [
        "http://your-server/wms?SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&FORMAT=image/png&TRANSPARENT=true&LAYERS=layer_name&SRS={epsg}&WIDTH=256&HEIGHT=256&BBOX={bbox}"
    ],
    minzoom: 1,
    maxzoom: 18
});
```

### 占位符

| 占位符 | 说明 |
|--------|------|
| {epsg} | 坐标系，如 EPSG:3857 或 EPSG:4326 |
| {bbox} | 范围，自动填充 |
| {z} | 级别 |
| {x} | 列号 |
| {y} | 行号 |

### 图层配置

```javascript
map.addLayer({
    id: "wms-layer",
    type: "raster",
    source: "wms-source",
    minzoom: 0,
    maxzoom: 24,
    layout: {
        visibility: "visible"
    },
    paint: {
        "raster-opacity": 1,
        "raster-fade-duration": 0
    }
});
```

### 完整示例

```javascript
map.on("load", function() {
    map.addSource("wms-source", {
        type: "raster",
        tiles: [
            "http://example.com/wms?SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&FORMAT=image/png&TRANSPARENT=true&LAYERS=my_layer&SRS={epsg}&WIDTH=256&HEIGHT=256&BBOX={bbox}"
        ],
        minzoom: 1,
        maxzoom: 18
    });

    map.addLayer({
        id: "wms-layer",
        type: "raster",
        source: "wms-source",
        paint: {
            "raster-opacity": 1
        }
    });
});
```

## WMTS 服务

Web Map Tile Service 瓦片服务。

### 数据源配置

```javascript
map.addSource("wmts-source", {
    type: "raster",
    tiles: [
        "http://your-server/wmts?layer=layer_name&style=&tilematrixset=EPSG:900913&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix=EPSG:900913:{z}&TileCol={x}&TileRow={y}"
    ],
    minzoom: 1,
    maxzoom: 18,
    zoomOffset: 0  // EPSG:4326 且从0级开始时设为 -1
});
```

### zoomOffset 说明

- EPSG:3857：`zoomOffset: 0`
- EPSG:4326 且瓦片从0级开始：`zoomOffset: -1`

### 投影切换

```javascript
function changeProjection(projection) {
    map.setProjection(projection);
    
    var zoomOffset = 0;
    var tiles;
    
    if (projection === "EPSG:4326") {
        zoomOffset = -1;
        tiles = ["http://server/wmts?...EPSG:4326..."];
    } else {
        tiles = ["http://server/wmts?...EPSG:900913..."];
    }
    
    var source = map.getSource("wmts-source");
    source.setZoomOffset(zoomOffset);
    source.setTiles(tiles);
}
```

## TMS 服务

Tile Map Service 瓦片服务。

### 数据源配置

```javascript
map.addSource("tms-source", {
    type: "raster",
    scheme: "tms",  // 指定 TMS 格式
    tiles: [
        "http://your-server/tms/1.0.0/layer@EPSG:900913@png/{z}/{x}/{y}.png"
    ],
    minzoom: 1,
    maxzoom: 18,
    zoomOffset: 0
});
```

### 与 XYZ 的区别

TMS 的 Y 轴方向与 XYZ 相反，需要设置 `scheme: "tms"`。

## raster 图层属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| raster-opacity | number | 1 | 透明度 [0,1] |
| raster-hue-rotate | number | 0 | 色调角度 [0, 359] |
| raster-brightness-min | number | 0 | 最小亮度 [0, 1] |
| raster-brightness-max | number | 1 | 最大亮度 [0, 1] |
| raster-saturation | number | 0 | 饱和度 [-1, 1] |
| raster-contrast | number | 0 | 对比度 [-1, 1] |
| raster-resampling | string | "linear" | 采样：linear/nearest |
| raster-fade-duration | number | 300 | 淡入淡出时间（毫秒） |

## 矢量瓦片

加载矢量瓦片服务。

### 数据源配置

```javascript
map.addSource("vector-source", {
    type: "vector",
    tiles: [
        "http://your-server/mvt?x={x}&y={y}&z={z}"
    ]
});
```

### 图层配置

```javascript
map.addLayer({
    id: "vector-layer",
    type: "fill-extrusion",  // 或 line、symbol、circle 等
    source: "vector-source",
    "source-layer": "layer_name",  // 矢量瓦片中的图层名
    paint: {
        "fill-extrusion-height": ["get", "height"],
        "fill-extrusion-base": ["get", "base"],
        "fill-extrusion-color": ["get", "color"],
        "fill-extrusion-opacity": 0.6
    }
});
```

## 完整示例

### WMS 服务示例

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>WMS 服务</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; }
        .controls {
            position: absolute;
            top: 10px;
            left: 10px;
            background: white;
            padding: 10px;
        }
    </style>
</head>
<body>
    <div id="map"></div>
    <div class="controls">
        <label>
            <input type="radio" name="proj" value="EPSG:3857" checked onclick="changeProjection(this.value)" />
            墨卡托投影
        </label>
        <label>
            <input type="radio" name="proj" value="EPSG:4326" onclick="changeProjection(this.value)" />
            经纬度直投
        </label>
    </div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
    <script>
        var map = new TMapGL.Map("map", {
            zoom: 3,
            center: [116.39, 39.91]
        });

        map.on("load", function() {
            map.addSource("wms-source", {
                type: "raster",
                tiles: [
                    "http://your-server/wms?SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&FORMAT=image/png&TRANSPARENT=true&LAYERS=layer&SRS={epsg}&WIDTH=256&HEIGHT=256&BBOX={bbox}"
                ],
                minzoom: 1,
                maxzoom: 18
            });

            map.addLayer({
                id: "wms-layer",
                type: "raster",
                source: "wms-source",
                paint: {
                    "raster-opacity": 1,
                    "raster-fade-duration": 0
                }
            });
        });

        function changeProjection(value) {
            map.setProjection(value);
        }
    </script>
</body>
</html>
```

### 3D 建筑示例

```javascript
map.on("load", function() {
    map.addSource("buildings-source", {
        type: "vector",
        tiles: ["http://server/buildings/mvt?x={x}&y={y}&z={z}"]
    });

    map.addLayer({
        id: "buildings-3d",
        type: "fill-extrusion",
        source: "buildings-source",
        "source-layer": "buildings",
        paint: {
            "fill-extrusion-height": ["get", "height"],
            "fill-extrusion-base": 0,
            "fill-extrusion-color": "#aaa",
            "fill-extrusion-opacity": 0.8,
            "fill-extrusion-vertical-gradient": true
        }
    });
});
```
