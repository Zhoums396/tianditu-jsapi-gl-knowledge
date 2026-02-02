# 基础类

## LngLat 经纬度坐标

表示地理坐标点（经纬度）。

### 构造函数

```javascript
new TMapGL.LngLat(lng, lat)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| lng | number | 经度 |
| lat | number | 纬度 |

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| lng | number | 经度 |
| lat | number | 纬度 |

### 方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `distanceTo(lngLat)` | number | 计算两点之间的距离（米） |
| `toArray()` | Array | 返回 [lng, lat] 数组 |
| `toString()` | string | 返回字符串表示 |

### 示例

```javascript
// 创建坐标点
var lngLat = new TMapGL.LngLat(116.404, 39.915);

// 获取坐标值
console.log(lngLat.lng);  // 116.404
console.log(lngLat.lat);  // 39.915

// 计算两点距离
var lngLat2 = new TMapGL.LngLat(116.414, 39.925);
var distance = lngLat.distanceTo(lngLat2);
console.log(distance);  // 返回距离（米）

// 转为数组
var arr = lngLat.toArray();  // [116.404, 39.915]
```

### 坐标系说明

天地图 API 使用 **CGCS2000** 坐标系（国家2000坐标系），与 WGS84 坐标系基本一致，可直接使用 GPS 坐标。

---

## LngLatBounds 地理范围

表示地理矩形区域。

### 构造函数

```javascript
new TMapGL.LngLatBounds(sw, ne)
// 或
new TMapGL.LngLatBounds([swLng, swLat, neLng, neLat])
```

| 参数 | 类型 | 说明 |
|------|------|------|
| sw | LngLat \| Array | 西南角坐标 |
| ne | LngLat \| Array | 东北角坐标 |

### 方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `getSouthWest()` | LngLat | 获取西南角坐标 |
| `getNorthEast()` | LngLat | 获取东北角坐标 |
| `getCenter()` | LngLat | 获取中心点坐标 |
| `extend(lngLat)` | LngLatBounds | 扩展范围以包含指定点 |
| `contains(lngLat)` | boolean | 判断点是否在范围内 |
| `toArray()` | Array | 返回 [[swLng, swLat], [neLng, neLat]] |

### 示例

```javascript
// 创建地理范围
var bounds = new TMapGL.LngLatBounds(
    [116.3, 39.8],   // 西南角
    [116.5, 40.0]    // 东北角
);

// 获取中心点
var center = bounds.getCenter();

// 判断点是否在范围内
var point = new TMapGL.LngLat(116.4, 39.9);
console.log(bounds.contains(point));  // true

// 获取当前地图视野范围
var mapBounds = map.getBounds();

// 自适应视野到指定范围
map.fitBounds(bounds, { padding: 50 });
```

---

## Point 像素坐标

表示屏幕像素坐标点。

### 构造函数

```javascript
// 通过 map.project() 获取
var point = map.project([lng, lat]);
```

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| x | number | X 坐标（像素） |
| y | number | Y 坐标（像素） |

### 示例

```javascript
// 经纬度转像素坐标
var point = map.project([116.404, 39.915]);
console.log(point.x, point.y);

// 像素坐标转经纬度
var lngLat = map.unproject([point.x, point.y]);
console.log(lngLat.lng, lngLat.lat);
```

---

## 坐标转换

### 经纬度转像素

```javascript
var point = map.project([lng, lat]);
console.log(point.x, point.y);
```

### 像素转经纬度

```javascript
var lngLat = map.unproject([x, y]);
console.log(lngLat.lng, lngLat.lat);
```

### 完整示例

```javascript
var map = new TMapGL.Map("map", {
    zoom: 10,
    center: [116.35, 39.91]
});

// 经纬度坐标转地图容器像素坐标
function project(lng, lat) {
    var pixel = map.project([lng, lat]);
    console.log("像素坐标:", pixel.x, pixel.y);
    return pixel;
}

// 地图容器像素坐标转经纬度坐标
function unproject(x, y) {
    var lngLat = map.unproject([x, y]);
    console.log("经纬度:", lngLat.lng.toFixed(6), lngLat.lat.toFixed(6));
    return lngLat;
}
```
