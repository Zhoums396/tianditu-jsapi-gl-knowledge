---
name: tianditu-jsapi-gl
description: 天地图 JSAPI WebGL (TMapGL) 开发指南。在编写、审查或调试使用天地图 API 的代码时应运用此技能。适用于涉及地图初始化、覆盖物渲染、图层管理、事件处理、控件交互或数据可视化的任务。当用户提及 TMapGL、天地图、tianditu、jsapi-v5 或相关地图开发需求时自动触发。
license: MIT
version: 1.0.0
---

# 天地图 JSAPI GL 开发指南

天地图 JSAPI WebGL 版本（v5.0）开发指南。包含地图初始化、覆盖物、事件、图层、数据可视化等核心模块的 API 说明和代码示例，旨在帮助开发者快速集成天地图并遵循正确的使用方式。

## 何时适用

在以下场景中参考这些指南：

- 创建新的地图页面或组件
- 在地图上添加标注、折线、多边形等覆盖物
- 处理地图交互事件（点击、拖拽、缩放等）
- 配置地图样式或切换图层
- 加载和可视化 GeoJSON 数据
- 创建热力图、聚合图等数据可视化效果
- 调试地图渲染或性能问题

## 快速参考

### 0. 基础概念

- [基础类](references/base-classes.md) - LngLat、LngLatBounds、Point

### 1. 地图

- [地图初始化](references/map-init.md) - 资源引入、创建实例、配置选项、视图控制
- [地图样式](references/map-style.md) - 个性化地图（黑色、蓝色）、图层显隐、投影

### 2. 地图覆盖物

- [点标记 Marker](references/marker.md) - 创建、自定义图标、点击事件
- [信息弹窗 Popup](references/popup.md) - 创建、内容设置、与 Marker 配合
- [覆盖物](references/bindOverlays.md) - 圆形、区域掩膜

### 3. 图层服务

- [GeoJSON 数据加载](references/bindGeoJSON.md) - 加载数据、点/线/面图层、样式配置
- [第三方服务](references/bindRasterLayers.md) - WMS/WMTS/TMS 图层加载

### 4. 数据可视化

- [热力图](references/bindHeatmap.md) - 创建热力图、颜色配置、权重
- [聚合图](references/bindCluster.md) - 点聚合、聚合样式、交互展开

### 5. 控件

- [地图控件](references/bindControls.md) - 导航、比例尺、全屏、地形

### 6. 事件

- [地图事件](references/bindEvents.md) - 点击、右键、图层事件、鼠标悬停

### 7. 工具服务

- [地理编码](references/geocoder.md) - 地址转坐标、坐标转地址、坐标转换

## 如何使用

请阅读各个参考文件以获取详细说明和代码示例：

```
references/map-init.md
```

每个参考文件包含：

- 功能简要说明
- 完整代码示例及解释
- API 参数说明和注意事项

## 引入方式

```html
<script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
```

## 基础示例

```javascript
// 创建地图实例
var map = new TMapGL.Map("map", {
    center: [116.40, 39.90],  // 中心点 [经度, 纬度]
    zoom: 12                   // 缩放级别
});

// 添加标记点
map.on("load", function() {
    var el = document.createElement("div");
    el.style.backgroundImage = "url(http://lbs.tianditu.gov.cn/js-api-v5-portal/image/marker.png)";
    el.style.width = "37px";
    el.style.height = "33px";

    var marker = new TMapGL.Marker({ element: el })
        .setLngLat([116.40, 39.90])
        .addTo(map);
});
```
