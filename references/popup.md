# Popup 信息弹窗

信息弹窗用于在地图上显示详细信息，可单独使用或与 Marker 配合。

## 快速开始

```javascript
map.on("load", function() {
    var popup = new TMapGL.Popup({ closeOnClick: false })
        .setLngLat([116.391358, 39.904966])
        .setHTML("<h1>天安门广场</h1>");

    popup.addTo(map);
});
```

## 构造函数

```javascript
new TMapGL.Popup(options)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| closeOnClick | boolean | true | 点击地图时是否关闭弹窗 |
| closeButton | boolean | true | 是否显示关闭按钮 |
| anchor | string | "bottom" | 弹窗锚点位置 |
| offset | Array \| number | 0 | 像素偏移 |
| className | string | "" | 自定义 CSS 类名 |
| maxWidth | string | "240px" | 最大宽度 |

### anchor 锚点位置

| 值 | 说明 |
|-----|------|
| center | 中心 |
| top | 顶部 |
| bottom | 底部（默认） |
| left | 左侧 |
| right | 右侧 |
| top-left | 左上 |
| top-right | 右上 |
| bottom-left | 左下 |
| bottom-right | 右下 |

## 方法

### 内容设置

```javascript
// 设置 HTML 内容
popup.setHTML("<h3>标题</h3><p>内容</p>");

// 设置纯文本
popup.setText("这是纯文本内容");

// 设置 DOM 元素
var div = document.createElement("div");
div.innerHTML = "<strong>自定义内容</strong>";
popup.setDOMContent(div);
```

### 位置设置

```javascript
// 设置位置
popup.setLngLat([116.40, 39.90]);

// 获取位置
var lngLat = popup.getLngLat();
```

### 显示控制

```javascript
// 添加到地图
popup.addTo(map);

// 从地图移除
popup.remove();

// 判断是否打开
var isOpen = popup.isOpen();
```

### 偏移设置

```javascript
// 设置偏移量
popup.setOffset([0, -30]);
```

### 最大宽度

```javascript
popup.setMaxWidth("300px");
```

## 常见场景

### 基础弹窗

```javascript
map.on("load", function() {
    var popup = new TMapGL.Popup({ closeOnClick: false })
        .setLngLat([116.391358, 39.904966])
        .setHTML("<h1>天安门广场</h1>");

    popup.addTo(map);
});
```

### 自定义样式弹窗

```javascript
var popup = new TMapGL.Popup({
    closeOnClick: true,
    closeButton: true,
    className: "custom-popup",
    maxWidth: "300px",
    offset: [0, -10]
})
.setLngLat([116.40, 39.90])
.setHTML(`
    <div style="padding: 10px;">
        <h3 style="margin: 0 0 8px 0;">北京</h3>
        <p style="margin: 0; color: #666;">中国首都</p>
    </div>
`)
.addTo(map);
```

### 与 Marker 配合使用

```javascript
map.on("load", function() {
    // 创建标记元素
    var el = document.createElement("div");
    el.style.backgroundImage = "url(http://lbs.tianditu.gov.cn/js-api-v5-portal/image/marker.png)";
    el.style.width = "37px";
    el.style.height = "33px";
    el.style.cursor = "pointer";

    // 创建弹窗
    var popup = new TMapGL.Popup({ 
        closeOnClick: true, 
        offset: [0, -30] 
    })
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

### 点击地图显示弹窗

```javascript
var popup = new TMapGL.Popup();

map.on("click", function(e) {
    popup
        .setLngLat(e.lngLat)
        .setHTML("<p>经度: " + e.lngLat.lng.toFixed(4) + "<br>纬度: " + e.lngLat.lat.toFixed(4) + "</p>")
        .addTo(map);
});
```

### 移除弹窗

```javascript
var popup;

map.on("load", function() {
    popup = new TMapGL.Popup({ closeOnClick: false })
        .setLngLat([116.391358, 39.904966])
        .setHTML("<h1>天安门广场</h1>")
        .addTo(map);
});

// 移除弹窗
function removePopup() {
    popup && popup.remove();
}
```

### 富文本内容弹窗

```javascript
var htmlContent = `
<div style="padding: 10px; min-width: 200px;">
    <img src="photo.jpg" style="width: 100%; border-radius: 4px;" />
    <h3 style="margin: 10px 0 5px 0;">景点名称</h3>
    <p style="margin: 0; color: #666; font-size: 14px;">这是景点的详细描述信息。</p>
    <div style="margin-top: 10px;">
        <span style="color: #ff6b6b;">★★★★★</span>
        <span style="color: #999; font-size: 12px;">4.8分</span>
    </div>
</div>
`;

var popup = new TMapGL.Popup({
    maxWidth: "280px",
    closeButton: true
})
.setLngLat([116.40, 39.90])
.setHTML(htmlContent)
.addTo(map);
```
