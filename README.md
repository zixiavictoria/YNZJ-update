<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>省级中药备案要求可视化看板</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.3.2/dist/echarts.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/echarts/map/js/china.js"></script>
    <style>
      #china-map {
        width: 100%;
        height: 100vh;
      }
      body {
        margin: 0;
        position: relative; /* 新增定位基准 */
      }
      /* 新增图例面板样式 */
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
    <!-- 新增的图例说明面板 -->
    <div id="legend-panel">
      <h3 style="margin:0 0 10px 0; color: #333;">备案要求档位说明</h3>
      <div id="legend-items"></div>
    </div>

    <script>
      // 已验证的标准化数据（34个省级行政区）
      const provinceData = {
        湖南省: 1,
        山西省: 1,
        上海市: 1,
        河南省: 1,
        陕西省: 1,
        甘肃省: 1,
        宁夏回族自治区: 1,
        江西省: 1,
        四川省: 1,
        吉林省: 2,
        山东省: 3,
        青海省: 3,
        河北省: 4,
        江苏省: 4,
        浙江省: 4,
        福建省: 4,
        广东省: 4,
        海南省: 4,
        云南省: 4,
        内蒙古自治区: 4,
        广西壮族自治区: 4,
        安徽省: 4,
        贵州省: 5,
        北京市: 6,
        天津市: 6,
        湖北省: 6,
        重庆市: 6,
        辽宁省: 6,
        黑龙江省: 6,
        西藏自治区: 0,
        新疆维吾尔自治区: 0,
        澳门特别行政区: 0,
        香港特别行政区: 0,
        台湾省: 0,
      }

      const chart = echarts.init(document.getElementById("china-map"))

      // 动态加载高精度地图数据（支持港澳台）
      fetch("https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json")
        .then((response) => response.json())
        .then((mapJson) => {
          echarts.registerMap("china", mapJson)

          const option = {
            title: {
              text: "医疗机构应用传统工艺配制中药制剂备案政策要求分布图",
              subtext: "免药效+急毒+长毒的要求-颜色由浅入深难度依次增加",
              left: "center",
              textStyle: { fontSize: 18 },
            },
            tooltip: {
              trigger: "item",
              formatter: (params) => `
                            <b>${params.name}</b><br/>
                            备案档位：${params.value}级<br/>
                            ${getRequirementDesc(params.value)}
                        `,
            },
            visualMap: {
              min: 0,
              max: 6,
              left: "right",
              orient: "vertical",
              calculable: true,
              inRange: {
                color: [
                  "#CCCCCC", // 0级
                  "#E6F5D0", // 1级
                  "#B8E186", // 2级
                  "#7EBC41", // 3级
                  "#4D9221", // 4级
                  "#276419", // 5级
                  "#00441B", // 6级
                ],
              },
              textStyle: { color: "#333" },
            },
            series: [
              {
                type: "map",
                map: "china",
                roam: true,
                label: {
                  show: true,
                  fontSize: 10,
                  formatter: "{b}",
                },
                data: Object.entries(provinceData).map(([name, value]) => ({
                  name,
                  value,
                  itemStyle: { borderColor: "#FFF" },
                })),
                emphasis: {
                  itemStyle: {
                    areaColor: "#FFD700",
                    borderWidth: 1,
                  },
                },
              },
            ],
          }

          chart.setOption(option)
          createLegend() // 初始化地图后创建图例
        })

      // 窗口自适应
      window.addEventListener("resize", () => chart.resize())

      // 档位说明生成器
      function getRequirementDesc(grade) {
        const descMap = {
          0: "未检索数据",
          1: "同国家基础要求：佐证材料不明确",
          2: "需提供5年+60例临床数据",
          3: "需提供5年+60例临床数据+病例分布",
          4: "需提供5年+100例临床数据",
          5: "需提供5年+100例临床数据+每个适应症需至少60例",
          6: "需提供5年+100例临床数据+病例分布",
        }
        return descMap[grade] || "未知档位要求"
      }


// 创建图例说明（修复版本）
function createLegend() {
  const colors = [
    "#CCCCCC","#E6F5D0","#B8E186","#7EBC41",
    "#4D9221","#276419","#00441B"
  ]
  const legendContainer = document.getElementById('legend-items')
  
  // 正确遍历0-6级
  legendContainer.innerHTML = 
    colors.map((color, grade) => `
      <div class="legend-item">
        <div class="color-block" style="background:${color}"></div>
        <div>
          <strong>${grade}级</strong>: ${getRequirementDesc(grade)}
        </div>
      </div>
    `).join('')
}


        </script>
  </body>
</html>
