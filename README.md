<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>省级中药备案要求可视化看板</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.3.2/dist/echarts.min.js"></script>
    <style>
      #china-map {
        width: 100%;
        height: 100vh;
      }
      body {
        margin: 0;
        position: relative;
      }
      #legend-panel {
        position: absolute;
        right: 20px;
        top: 120px;
        background: rgba(255, 255, 255, 0.9);
        padding: 15px;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0,0,0,0.1);
        z-index: 1000;
        max-width: 280px;
      }
      .legend-item {
        margin: 8px 0;
        display: flex;
        align-items: center;
      }
      .color-block {
        width: 20px;
        height: 20px;
        margin-right: 10px;
        flex-shrink: 0;
      }
    </style>
  </head>
  <body>
    <div id="china-map"></div>
    <div id="legend-panel">
      <h3 style="margin:0 0 10px 0; color: #333;">备案要求档位说明</h3>
      <div id="legend-items"></div>
    </div>

    <script>
      const provinceData = {/* 数据保持不变，此处省略以节省空间 */}

      const chart = echarts.init(document.getElementById("china-map"))

      // 改用可靠的地图数据源
      fetch("https://cdn.jsdelivr.net/npm/echarts@5.3.2/map/json/china.json")
        .then((response) => response.json())
        .then((mapJson) => {
          echarts.registerMap("china", mapJson)

          const option = {
            // 保持原有配置，增加关键配置：
            series: [{
              type: "map",
              map: "china",
              roam: true,
              aspectScale: 0.85,  // 修正地图显示比例
              layoutCenter: ['50%', '53%'],  // 居中校准
              // 其他配置保持不变...
            }]
          }
          chart.setOption(option)
          createLegend()
        })

      // 保持原有函数不变，增加窗口尺寸同步：
      window.addEventListener("resize", () => {
        chart.resize()
        document.getElementById('legend-panel').style.top = '120px'
      })
    </script>
  </body>
</html>
