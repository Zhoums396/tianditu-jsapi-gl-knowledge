# 地图与 ECharts 图表联动

创建地图与 ECharts 图表联动展示，点击地图标记显示对应统计数据。

## 完整示例（必须使用此布局）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>地图与ECharts联动</title>
    <style>
        html, body { 
            width: 100%; 
            height: 100%; 
            margin: 0; 
            padding: 0; 
            display: flex;  /* 重要：使用 flex 布局 */
        }
        #map { 
            width: 60%; 
            height: 100%; 
        }
        #chart-container { 
            width: 40%; 
            height: 100%; 
            display: flex; 
            flex-direction: column;
            background: #f5f5f5;
        }
        .chart-panel { 
            flex: 1; 
            padding: 10px; 
        }
        .chart-title { 
            font-size: 14px; 
            font-weight: bold; 
            padding: 10px; 
            background: #fff;
            border-bottom: 1px solid #eee;
        }
        #chart { 
            width: 100%; 
            height: 100%; 
        }
    </style>
    <!-- 先加载 ECharts -->
    <script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>
</head>
<body>
    <div id="map"></div>
    <div id="chart-container">
        <div class="chart-title">📊 点击地图标记查看详细数据</div>
        <div class="chart-panel">
            <div id="chart"></div>
        </div>
    </div>
    
    <script src="https://api.tianditu.gov.cn/api/v5/js?tk=${TIANDITU_TOKEN}" type="text/javascript"></script>
    <script>
        // 示例城市数据
        var cityData = {
            "北京": { coordinates: [116.40, 39.90], values: [100, 200, 150, 180, 220] },
            "上海": { coordinates: [121.47, 31.23], values: [150, 180, 200, 160, 190] },
            "广州": { coordinates: [113.26, 23.13], values: [80, 120, 100, 140, 160] }
        };
        
        // 初始化地图
        var map = new TMapGL.Map("map", {
            zoom: 4,
            center: [116.40, 35.0]
        });
        
        // 初始化 ECharts
        var chartDom = document.getElementById('chart');
        var myChart = echarts.init(chartDom);
        
        map.on("load", function() {
            // 为每个城市创建标记
            for (var city in cityData) {
                (function(cityName, data) {
                    var el = document.createElement('div');
                    el.style.width = '30px';
                    el.style.height = '30px';
                    el.style.borderRadius = '50%';
                    el.style.backgroundColor = '#ff4444';
                    el.style.border = '3px solid #fff';
                    el.style.cursor = 'pointer';
                    el.style.boxShadow = '0 2px 6px rgba(0,0,0,0.3)';
                    el.title = cityName;
                    
                    var marker = new TMapGL.Marker({ element: el })
                        .setLngLat(data.coordinates);
                    marker.addTo(map);
                    
                    // 点击标记显示图表
                    el.addEventListener('click', function() {
                        showChart(cityName, data.values);
                    });
                })(city, cityData[city]);
            }
            
            // 默认显示北京数据
            showChart("北京", cityData["北京"].values);
        });
        
        // 显示图表
        function showChart(cityName, values) {
            var option = {
                title: {
                    text: cityName + ' 数据统计',
                    left: 'center'
                },
                tooltip: {
                    trigger: 'axis'
                },
                xAxis: {
                    type: 'category',
                    data: ['一月', '二月', '三月', '四月', '五月']
                },
                yAxis: {
                    type: 'value'
                },
                series: [{
                    name: '数值',
                    type: 'bar',
                    data: values,
                    itemStyle: {
                        color: '#5470c6'
                    }
                }]
            };
            myChart.setOption(option);
        }
        
        // 窗口大小变化时重绘图表
        window.addEventListener('resize', function() {
            myChart.resize();
        });
    </script>
</body>
</html>
```

## 关键要点

1. **布局必须使用 flex**：`display: flex` 让地图和图表并排显示
2. **地图宽度**：设置为 `60%` 或其他比例，不能是 `100%`
3. **图表容器**：需要嵌套结构，外层容器控制宽度，内层 `#chart` 用于渑染
4. **ECharts 先加载**：在 head 中加载 ECharts，在 body 末尾加载天地图
5. **响应式**：监听 `resize` 事件调用 `myChart.resize()`

## 图表类型选择

- **柱状图**：`type: 'bar'` - 适合对比数据
- **折线图**：`type: 'line'` - 适合趋势数据
- **饼图**：`type: 'pie'` - 适合占比数据
- **雷达图**：`type: 'radar'` - 适合多维度对比
