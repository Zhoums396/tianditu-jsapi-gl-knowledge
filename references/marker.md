# Marker 点标记

## 快速开始

```javascript
map.on("load", function() {
    var el = document.createElement("div");
    el.style.backgroundImage = "url(http://lbs.tianditu.gov.cn/js-api-v5-portal/image/marker.png)";
    el.style.width = "37px";
    el.style.height = "33px";
    el.style.cursor = "pointer";

    var marker = new TMapGL.Marker({ element: el })
        .setLngLat([116.40, 39.90])
        .addTo(map);
});
```

## 构造函数

```javascript
new TMapGL.Marker(options)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| element | HTMLElement | 默认图标 | 自定义标记 DOM 元素 |
| anchor | string | "center" | 锚点位置 |
| offset | Array | [0, 0] | 像素偏移 [x, y] |
| draggable | boolean | false | 是否可拖拽 |
| rotation | number | 0 | 旋转角度 |
| pitchAlignment | string | "auto" | 倾斜对齐方式 |
| rotationAlignment | string | "auto" | 旋转对齐方式 |

### anchor 锚点位置

| 值 | 说明 |
|-----|------|
| center | 中心（默认） |
| top | 顶部中心 |
| bottom | 底部中心 |
| left | 左侧中心 |
| right | 右侧中心 |
| top-left | 左上角 |
| top-right | 右上角 |
| bottom-left | 左下角 |
| bottom-right | 右下角 |

## 方法

### 位置

```javascript
// 设置位置
marker.setLngLat([116.40, 39.90]);

// 获取位置
var lngLat = marker.getLngLat();
console.log(lngLat.lng, lngLat.lat);
```

### 添加到地图

```javascript
// 添加到地图
marker.addTo(map);

// 从地图移除
marker.remove();
```

### 弹窗绑定

```javascript
// 创建弹窗
var popup = new TMapGL.Popup({ offset: [0, -30] })
    .setHTML("<h3>标题</h3><p>内容</p>");

// 绑定弹窗
marker.setPopup(popup);

// 获取弹窗
var p = marker.getPopup();

// 切换弹窗显示
marker.togglePopup();
```

### 偏移

```javascript
// 设置像素偏移
marker.setOffset([10, -5]);

// 获取偏移
var offset = marker.getOffset();
```

### 旋转

```javascript
// 设置旋转角度
marker.setRotation(45);

// 获取旋转角度
var rotation = marker.getRotation();
```

### 拖拽

```javascript
// 设置可拖拽
marker.setDraggable(true);

// 获取是否可拖拽
var draggable = marker.isDraggable();
```

## 常见场景

### 默认标记点

```javascript
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

### 自定义图标

```javascript
map.on("load", function() {
    var el = document.createElement("div");
    el.style.backgroundImage = "url(custom-icon.png)";
    el.style.width = "32px";
    el.style.height = "32px";
    el.style.backgroundSize = "cover";

    var marker = new TMapGL.Marker({ 
        element: el,
        anchor: "bottom"  // 锚点在底部
    })
    .setLngLat([116.40, 39.90])
    .addTo(map);
});
```

### 带弹窗的标记点

```javascript
map.on("load", function() {
    var el = document.createElement("div");
    el.className = "marker";
    el.style.backgroundImage = "url(http://lbs.tianditu.gov.cn/js-api-v5-portal/image/marker.png)";
    el.style.width = "37px";
    el.style.height = "33px";
    el.style.cursor = "pointer";

    // 创建弹窗
    var popup = new TMapGL.Popup({ closeOnClick: true, offset: [0, -30] })
        .setHTML("<h3>天安门</h3><p>北京市中心</p>");

    // 创建标记并绑定弹窗
    var marker = new TMapGL.Marker({ element: el })
        .setLngLat([116.391358, 39.904966])
        .setPopup(popup)
        .addTo(map);

    // 点击标记切换弹窗
    el.addEventListener("click", function(e) {
        e.stopPropagation();
        marker.togglePopup();
    });
});
```

### 多个标记点

```javascript
var markersData = [
    { lng: 116.40, lat: 39.90, title: "点1" },
    { lng: 116.45, lat: 39.95, title: "点2" },
    { lng: 116.35, lat: 39.85, title: "点3" }
];

map.on("load", function() {
    markersData.forEach(function(data) {
        var el = document.createElement("div");
        el.style.backgroundImage = "url(http://lbs.tianditu.gov.cn/js-api-v5-portal/image/marker.png)";
        el.style.width = "37px";
        el.style.height = "33px";
        el.style.cursor = "pointer";

        var marker = new TMapGL.Marker({ element: el })
            .setLngLat([data.lng, data.lat])
            .addTo(map);

        el.addEventListener("click", function() {
            new TMapGL.Popup()
                .setLngLat([data.lng, data.lat])
                .setHTML("<h3>" + data.title + "</h3>")
                .addTo(map);
        });
    });
});
```

### 点击删除标记

```javascript
map.on("load", function() {
    var el = document.createElement("div");
    el.style.backgroundImage = "url(http://lbs.tianditu.gov.cn/js-api-v5-portal/image/marker.png)";
    el.style.width = "37px";
    el.style.height = "33px";
    el.style.cursor = "pointer";

    var marker = new TMapGL.Marker({ element: el })
        .setLngLat([116.35, 39.91])
        .addTo(map);

    // 点击标记删除
    el.addEventListener("click", function() {
        marker.remove();
    });
});
```

### 可拖拽标记

```javascript
var marker = new TMapGL.Marker({ 
    element: el,
    draggable: true 
})
.setLngLat([116.40, 39.90])
.addTo(map);

// 监听拖拽结束
marker.on("dragend", function() {
    var lngLat = marker.getLngLat();
    console.log("新位置:", lngLat.lng, lngLat.lat);
});
```
