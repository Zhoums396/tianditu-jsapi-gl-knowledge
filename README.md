# Tianditu JSAPI GL Knowledge Base

天地图 JavaScript API v5 (GL版) 知识库，用于 LLM 驱动的地图代码生成。

## 📁 目录结构

```
tianditu-jsapi-gl/
├── SKILL.md              # 技能概述和使用指南
├── README.md             # 本文件
└── references/           # API 参考文档
    ├── map-init.md       # 地图初始化
    ├── map-style.md      # 地图样式
    ├── base-classes.md   # 基础类 (LngLat, LngLatBounds, Point)
    ├── marker.md         # 标记点
    ├── popup.md          # 弹窗
    ├── bindOverlays.md   # 覆盖物 (圆、线、面、掩膜)
    ├── bindGeoJSON.md    # GeoJSON 数据绑定
    ├── bindHeatmap.md    # 热力图
    ├── bindCluster.md    # 点聚合
    ├── bindControls.md   # 地图控件
    ├── bindEvents.md     # 事件监听
    ├── bindRasterLayers.md # 栅格图层
    └── geocoder.md       # 地理编码
```

## 🎯 用途

这个知识库供 LLM (如 GPT-4) 参考，生成符合天地图 API v5 规范的代码。

## 📖 使用方式

1. **knowledgeLoader.js** 读取这些 Markdown 文件
2. 根据用户需求选择相关的功能文档
3. 将文档内容作为 LLM 的上下文
4. LLM 根据文档生成正确的代码

## ⚠️ API 规范

天地图 v5 基于 MapLibre GL 风格 API，**不是**传统的叠加物 API。

### ✅ 正确的 API
- `map.addSource()` + `map.addLayer()` - 添加数据和图层
- `new TMapGL.Marker()` - 标记点
- `new TMapGL.Popup()` - 弹窗
- `map.fitBounds()` - 自适应边界

### ❌ 不存在的 API
- `TMapGL.Circle` - 不存在！用 `addLayer({ type: "circle" })`
- `TMapGL.Polygon` - 不存在！用 `addLayer({ type: "fill" })`
- `TMapGL.Polyline` - 不存在！用 `addLayer({ type: "line" })`

## 📝 License

MIT
