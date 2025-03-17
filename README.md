<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>省级中药备案要求可视化看板</title>
    <script src="https://cdn.staticfile.org/echarts/5.3.2/echarts.min.js"></script>
    <style>
      #china-map {
        width: 100%;
        height: 100vh;
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
    <div id="china-map"></div>
    <div id="legend-panel">
      <h3 style="margin:0 0 10px 0; color: #333;">备案要求档位说明</h3>
      <div id="legend-items"></div>
    </div>

    <script>
      function initializeChart() {
        // 示例数据（省份名称必须与 china.geo.json 中的 "name" 字段完全一致）
        const provinceData = {
          "北京": { grade: 1 },    // 注意：此处为 "北京"，而非 "北京市"
          "上海": { grade: 2 },
          "广东": { grade: 3 }
        };

        const chart = echarts.init(document.getElementById("china-map"));
        
        // 加载新仓库中的地图文件
        fetch("https://raw.githubusercontent.com/natee/highcharts-china-geo/master/geojson/china.geo.json")
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
                nameProperty: "name", // 对应 china.geo.json 中的字段
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

        window.addEventListener("resize", () => chart.resize());
      }

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

      window.onload = initializeChart;
    </script>
  </body>
</html>
