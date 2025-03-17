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
      // 修正后的省份名称（使用地理数据标准简称）
      const provinceData = {
        湖南: 1, 山西: 1, 上海: 1, 河南: 1,
        陕西: 1, 甘肃: 1, 宁夏: 1, 江西: 1,
        四川: 1, 吉林: 2, 山东: 3, 青海: 3,
        河北: 4, 江苏: 4, 浙江: 4, 福建: 4,
        广东: 4, 海南: 4, 云南: 4, 内蒙古: 4,
        广西: 4, 安徽: 4, 贵州: 5, 北京: 6,
        天津: 6, 湖北: 6, 重庆: 6, 辽宁: 6,
        黑龙江: 6, 西藏: 0, 新疆: 0, 澳门: 0,
        香港: 0, 台湾省: 0  // 特殊保留全称
      }

      const chart = echarts.init(document.getElementById("china-map"))

      fetch("https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json")
        .then(response => {
          if (!response.ok) throw new Error(`加载失败: ${response.status}`)
          return response.json()
        })
        .then(mapJson => {
          echarts.registerMap("china", mapJson)
          initChart()
        })
        .catch(error => {
          console.error(error)
          document.getElementById("china-map").innerHTML = 
            `<p style="padding:20px;color:red">地图加载失败：${error.message}</p>`
        })

      function initChart() {
        const option = {
          title: {
            text: "要求分布图",
            subtext: "颜色由浅入深难度依次增加",
            left: "center"
          },
          tooltip: {
            trigger: "item",
            formatter: params => `
              <b>${params.name}</b><br/>
              备案档位：${params.value}级<br/>
              ${getRequirementDesc(params.value)}
            `
          },
          visualMap: {
            min: 0, max: 6,
            left: "right",
            orient: "vertical",
            inRange: {
              color: ["#CCCCCC","#E6F5D0","#B8E186",
                     "#7EBC41","#4D9221","#276419","#00441B"]
            }
          },
          series: [{
            type: "map",
            map: "china",
            roam: true,
            label: { show: true, fontSize: 10 },
            data: Object.entries(provinceData).map(([name, value]) => ({
              name, value,
              itemStyle: { borderColor: "#FFF" }
            })),
            emphasis: {
              itemStyle: { areaColor: "#FFD700" }
            }
          }]
        }
        chart.setOption(option)
        createLegend()
        window.addEventListener("resize", () => chart.resize())
      }

      function getRequirementDesc(grade) {
        const descMap = {
          0: "未检索数据", 1: "同国家基础要求：佐证材料不明确",
          2: "需提供5年+60例临床数据", 3: "需提供5年+60例临床数据+病例分布",
          4: "需提供5年+100例临床数据", 5: "需提供5年+100例临床数据+每个适应症需至少60例",
          6: "需提供5年+100例临床数据+病例分布"
        }
        return descMap[grade] || "未知档位要求"
      }

      function createLegend() {
        const colors = ["#CCCCCC","#E6F5D0","#B8E186",
                       "#7EBC41","#4D9221","#276419","#00441B"]
        document.getElementById('legend-items').innerHTML = 
          colors.map((color, grade) => `
            <div class="legend-item">
              <div class="color-block" style="background:${color}"></div>
              <div><strong>${grade}级</strong>: ${getRequirementDesc(grade)}</div>
            </div>
          `).join('')
      }
    </script>
  </body>
</html>
