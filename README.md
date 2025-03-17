<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>省级中药备案要求可视化看板</title>
    <!-- 使用更稳定的 ECharts CDN -->
    <script src="https://cdn.staticfile.org/echarts/5.3.2/echarts.min.js"></script>
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
        top: 20px;
        right: 20px;
        background: rgba(255, 255, 255, 0.9);
        padding: 15px;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0,0,0,0.1);
      }
    </style>
  </head>
  <body>
    <!-- 地图容器 -->
    <div id="china-map"></div>
    
    <!-- 图例面板 -->
    <div id="legend-panel">
      <h3 style="margin:0 0 10px 0; color: #333;">备案要求档位说明</h3>
      <div id="legend-items"></div>
    </div>

    <script>
      // ========== 地图初始化逻辑 ==========
      function initializeChart() {
        const provinceData = {
          // 示例数据（需替换为实际数据）
          "北京市": { value: 90, grade: 1 },
          "上海市": { value: 80, grade: 2 },
          "广东省": { value: 70, grade: 3 }
        };

        const chart = echarts.init(document.getElementById("china-map"));
        
        // 从本地加载地图JSON文件
        fetch("data/100000_full.json")  // 本地路径
          .then((response) => response.json())
          .then((mapJson) => {
            echarts.registerMap("china", mapJson);
            
            const option = {
              tooltip: {
                trigger: "item",
                formatter: (params) => {
                  const data = provinceData[params.name];
                  return `${params.name}<br/>备案要求档位: ${data?.grade || '无数据'}`;
                }
              },
              visualMap: {
                left: "right",
                min: 1,
                max: 3,
                inRange: { color: ['#90c7ff', '#73b3ff', '#2f7ed8'] },
                text: ["高", "低"],
                calculable: true
              },
              series: [{
                type: "map",
                map: "china",
                data: Object.entries(provinceData).map(([name, data]) => ({
                  name,
                  value: data.grade
                })),
                label: { show: true }
              }]
            };

            chart.setOption(option);
            createLegend();
          });

        // 窗口自适应
        window.addEventListener("resize", () => chart.resize());
      }

      // ========== 图例生成函数 ==========
      function createLegend() {
        const legendItems = document.getElementById("legend-items");
        const grades = [
          { color: "#90c7ff", desc: "档位1: 基础备案要求" },
          { color: "#73b3ff", desc: "档位2: 中等备案要求" },
          { color: "#2f7ed8", desc: "档位3: 高级备案要求" }
        ];

        legendItems.innerHTML = grades
          .map(
            (grade) => `
              <div style="margin: 5px 0; display: flex; align-items: center">
                <div style="width: 20px; height: 20px; background: ${grade.color}; margin-right: 8px"></div>
                <span>${grade.desc}</span>
              </div>
            `
          )
          .join("");
      }

      // 页面加载完成后直接初始化
      window.onload = initializeChart;
    </script>
  </body>
</html>
