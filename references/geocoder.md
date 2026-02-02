# 地理编码

地址与经纬度坐标之间的转换。

## 地址转坐标（地理编码）

将文本地址转换为经纬度坐标。

### 方法

```javascript
map.getGeoCoderLocation(address)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| address | string | 地址字符串 |

### 返回值

```javascript
{
    status: "0",           // "0" 表示成功
    location: {
        lon: 116.35,       // 经度
        lat: 39.91         // 纬度
    }
}
```

### 示例

```javascript
map.getGeoCoderLocation("北京市海淀区莲花池西路28号").then(function(json) {
    if (json.status === "0") {
        var lng = json.location.lon;
        var lat = json.location.lat;
        console.log("坐标:", lng, lat);
        
        // 定位到该位置
        map.flyTo({
            center: [lng, lat],
            zoom: 15
        });
    } else {
        console.log("地址解析失败:", json.message);
    }
});
```

## 坐标转地址（逆地理编码）

将经纬度坐标转换为文本地址。

### 方法

```javascript
map.getGeoCoderAddress(lng, lat)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| lng | number | 经度 |
| lat | number | 纬度 |

### 返回值

```javascript
{
    status: "0",
    result: {
        formatted_address: "北京市东城区天安门",
        // 其他地址信息...
    }
}
```

### 示例

```javascript
map.getGeoCoderAddress(116.39, 39.40).then(function(json) {
    if (json.status === "0") {
        var address = json.result.formatted_address;
        console.log("地址:", address);
    }
});
```

## 坐标转换

经纬度坐标与像素坐标之间的转换。

### 经纬度转像素

```javascript
var point = map.project([116.40, 39.90]);
console.log("像素坐标:", point.x, point.y);
```

### 像素转经纬度

```javascript
var lngLat = map.unproject([100, 150]);
console.log("经纬度:", lngLat.lng, lngLat.lat);
```

## 常见场景

### 搜索定位

```javascript
async function searchAddress(address) {
    try {
        var result = await map.getGeoCoderLocation(address);
        if (result.status === "0") {
            var lng = result.location.lon;
            var lat = result.location.lat;
            
            // 飞行到目标位置
            map.flyTo({
                center: [lng, lat],
                zoom: 15
            });
            
            // 添加标记
            var el = document.createElement("div");
            el.style.backgroundImage = "url(marker.png)";
            el.style.width = "32px";
            el.style.height = "32px";
            
            new TMapGL.Marker({ element: el })
                .setLngLat([lng, lat])
                .addTo(map);
            
            return { lng, lat };
        } else {
            throw new Error("地址解析失败");
        }
    } catch (error) {
        console.error(error);
    }
}
```

### 点击显示地址

```javascript
map.on("click", function(e) {
    var lng = e.lngLat.lng;
    var lat = e.lngLat.lat;
    
    map.getGeoCoderAddress(lng, lat).then(function(json) {
        if (json.status === "0") {
            new TMapGL.Popup()
                .setLngLat(e.lngLat)
                .setHTML("<p>" + json.result.formatted_address + "</p>")
                .addTo(map);
        }
    });
});
```

### 批量地址转换

```javascript
async function batchGeocode(addresses) {
    var results = [];
    for (var i = 0; i < addresses.length; i++) {
        var result = await map.getGeoCoderLocation(addresses[i]);
        if (result.status === "0") {
            results.push({
                address: addresses[i],
                lng: result.location.lon,
                lat: result.location.lat
            });
        }
    }
    return results;
}

// 使用示例
var addresses = ["北京市天安门", "上海市外滩", "广州市珠江新城"];
batchGeocode(addresses).then(function(results) {
    results.forEach(function(item) {
        console.log(item.address + ": " + item.lng + ", " + item.lat);
    });
});
```

## 完整示例

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>地理编码示例</title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; }
        .search-box {
            position: absolute;
            top: 10px;
            left: 10px;
            background: white;
            padding: 10px;
            border-radius: 4px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.2);
        }
        .search-box input {
            padding: 8px;
            width: 200px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        .search-box button {
            padding: 8px 16px;
            background: #1890ff;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="map"></div>
    <div class="search-box">
        <input type="text" id="address" placeholder="输入地址" value="北京市天安门" />
        <button onclick="search()">搜索</button>
        <p id="result"></p>
    </div>
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=您的密钥"></script>
    <script>
        var map = new TMapGL.Map("map", {
            center: [116.40, 39.90],
            zoom: 10
        });

        var currentMarker = null;

        function search() {
            var address = document.getElementById("address").value;
            var resultEl = document.getElementById("result");
            
            map.getGeoCoderLocation(address).then(function(json) {
                if (json.status === "0") {
                    var lng = json.location.lon;
                    var lat = json.location.lat;
                    resultEl.innerHTML = "坐标: " + lng.toFixed(4) + ", " + lat.toFixed(4);
                    
                    // 移除旧标记
                    if (currentMarker) {
                        currentMarker.remove();
                    }
                    
                    // 飞行到目标位置
                    map.flyTo({
                        center: [lng, lat],
                        zoom: 15
                    });
                    
                    // 添加新标记
                    var el = document.createElement("div");
                    el.style.backgroundImage = "url(http://lbs.tianditu.gov.cn/js-api-v5-portal/image/marker.png)";
                    el.style.width = "37px";
                    el.style.height = "33px";
                    
                    currentMarker = new TMapGL.Marker({ element: el })
                        .setLngLat([lng, lat])
                        .addTo(map);
                } else {
                    resultEl.innerHTML = "地址解析失败";
                }
            });
        }

        // 点击地图显示地址
        map.on("click", function(e) {
            map.getGeoCoderAddress(e.lngLat.lng, e.lngLat.lat).then(function(json) {
                if (json.status === "0") {
                    new TMapGL.Popup()
                        .setLngLat(e.lngLat)
                        .setHTML("<p>" + json.result.formatted_address + "</p>")
                        .addTo(map);
                }
            });
        });
    </script>
</body>
</html>
```
