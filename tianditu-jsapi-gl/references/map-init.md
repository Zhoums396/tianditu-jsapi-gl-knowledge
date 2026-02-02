# 地图初始化

## 引入资源

### 在线引入

```html
<script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
```

> 需要在天地图官网申请开发者密钥 (tk)

## 基础初始化

### HTML 结构

```html
<div id="map" style="width: 100%; height: 100%;"></div>
```

### JavaScript 初始化

```javascript
// 创建地图实例
var map = new TMapGL.Map("map", {
    center: [116.40, 39.90],  // 中心点 [经度, 纬度]
    zoom: 12                   // 缩放级别
});
```

## 构造函数

```javascript
new TMapGL.Map(container, options)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| container | string \| HTMLElement | 地图容器的 ID 或 DOM 元素 |
| options | Object | 配置参数 |

## 配置选项

### 完整配置示例

```javascript
var map = new TMapGL.Map("map", {
    // 基础配置
    center: [116.35, 39.91],      // 中心点 [经度, 纬度]，默认 [0, 0]
    zoom: 10,                      // 缩放级别，默认 0
    minZoom: 0,                    // 最小缩放级别，默认 -2
    maxZoom: 22,                   // 最大缩放级别，默认 22
    
    // 地图类型
    mapType: "vt",                 // vt(矢量) | img(影像) | ter(地形)
    styleId: "normal",             // normal | black | blue
    projection: "EPSG:3857",       // EPSG:3857 | EPSG:4326
    
    // 视角控制
    pitch: 0,                      // 倾斜角度 [0, 60]
    bearing: 0,                    // 旋转角度 [0, 360]
    minPitch: 0,                   // 最小倾斜角
    maxPitch: 60,                  // 最大倾斜角
    
    // 范围限制
    bounds: null,                  // 初始显示范围 [minLng, minLat, maxLng, maxLat]
    maxBounds: null,               // 最大显示范围
    
    // 交互控制
    scrollZoom: true,              // 滚轮缩放
    boxZoom: true,                 // Shift+拖拽框选缩放
    dragRotate: true,              // 右键拖拽旋转
    dragPan: true,                 // 左键拖拽平移
    keyboard: true,                // 键盘控制
    doubleClickZoom: true,         // 双击缩放
    touchZoomRotate: true,         // 触摸缩放旋转
    touchPitch: true,              // 触摸倾斜
    
    // 图层显隐
    visibility: {
        annotation: true,          // 注记
        road: true,                // 道路
        hyd: true,                 // 水系
        veg: true,                 // 绿地
        resa: true,                // 居民地
        railway: true              // 铁路
    }
});
```

### 配置选项说明

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| center | Array | [0, 0] | 地图中心点 [经度, 纬度] |
| zoom | number | 0 | 缩放级别 |
| minZoom | number | -2 | 最小缩放级别 |
| maxZoom | number | 22 | 最大缩放级别 |
| mapType | string | "vt" | 地图类型：vt(矢量)/img(影像)/ter(地形) |
| styleId | string | "normal" | 地图样式：normal/black/blue |
| projection | string | "EPSG:3857" | 投影：EPSG:3857/EPSG:4326 |
| pitch | number | 0 | 倾斜角度 [0, 60] |
| bearing | number | 0 | 旋转角度 [0, 360] |
| bounds | Array | null | 初始显示范围 |
| maxBounds | Array | null | 最大显示范围 |

## 视图控制方法

### 设置中心点

```javascript
// 设置中心点
map.setCenter([116.40, 39.90]);

// 获取中心点
var center = map.getCenter();
console.log(center.lng, center.lat);
```

### 设置缩放级别

```javascript
// 设置缩放级别
map.setZoom(12);

// 获取缩放级别
var zoom = map.getZoom();

// 设置缩放范围
map.setMinZoom(5);
map.setMaxZoom(18);
```

### 设置倾斜角和旋转角

```javascript
// 设置倾斜角（0-60度）
map.setPitch(45);

// 设置旋转角（0-360度）
map.setBearing(90);

// 获取当前值
var pitch = map.getPitch();
var bearing = map.getBearing();
```

### 视图跳转

```javascript
// 无动画跳转
map.jumpTo({
    center: [116.40, 39.90],
    zoom: 12,
    pitch: 45,
    bearing: 30
});

// 有动画飞行
map.flyTo({
    center: [116.40, 39.90],
    zoom: 12,
    pitch: 45,
    bearing: 30
});
```

### 适应范围

```javascript
// 根据范围自适应视野
var bounds = new TMapGL.LngLatBounds(
    [116.3, 39.8],
    [116.5, 40.0]
);
map.fitBounds(bounds, { padding: 50 });
```

### 地图平移

```javascript
// 按像素平移
map.panBy([100, 100]);

// 平移到指定位置
map.setCenter([110.35, 35.91]);
```

## 获取地图状态

```javascript
// 地图类型
var mapType = map.getMapType();

// 地图投影
var projection = map.getProjection();

// 地图中心点
var center = map.getCenter();

// 地图级别
var zoom = map.getZoom();

// 地图显示范围
var bounds = map.getBounds();

// 地图最大显示范围
var maxBounds = map.getMaxBounds();

// 倾斜角度
var pitch = map.getPitch();

// 旋转角度
var bearing = map.getBearing();
```

## 动态切换

### 切换地图类型

```javascript
// 矢量地图
map.setMapType("vt");

// 影像地图
map.setMapType("img");

// 地形地图
map.setMapType("ter");
```

### 切换投影

```javascript
// 墨卡托投影
map.setProjection("EPSG:3857");

// 经纬度直投
map.setProjection("EPSG:4326");
```

### 控制图层显隐

```javascript
map.setVisibility({
    annotation: true,   // 注记
    road: false,        // 道路
    hyd: true,          // 水系
    veg: true,          // 绿地
    resa: true,         // 居民地
    railway: false      // 铁路
});
```

## 常见场景

### 显示北京地图

```javascript
var map = new TMapGL.Map("map", {
    center: [116.40, 39.90],
    zoom: 12
});
```

### 显示卫星影像

```javascript
var map = new TMapGL.Map("map", {
    center: [116.40, 39.90],
    zoom: 12,
    mapType: "img"
});
```

### 3D 倾斜视图

```javascript
var map = new TMapGL.Map("map", {
    center: [116.40, 39.90],
    zoom: 15,
    pitch: 60,
    bearing: 45
});
```

### 限制显示范围

```javascript
var map = new TMapGL.Map("map", {
    center: [116.35, 39.91],
    zoom: 10,
    maxBounds: [116.0, 39.5, 117.0, 40.5]  // 限制在北京区域
});
```
