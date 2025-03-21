<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>省级中药备案要求可视化看板</title>
    <!-- 使用国内稳定 CDN -->
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
        z-index: 1000;
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
      // ========== 地图初始化逻辑 ==========
      function initializeChart() {
        // 省份数据（名称必须与 china.geo.json 中的 "name" 字段完全一致）
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

          // 补充其他省份数据...
        };

        const chart = echarts.init(document.getElementById("china-map"));
        
        // 加载 GitHub 上的地图文件
        fetch("https://raw.githubusercontent.com/natee/highcharts-china-geo/master/geojson/china.geo.json")
          .then((response) => response.json())
          .then((mapJson) => {
            // 注册地图
            echarts.registerMap("china", mapJson);

            // 配置项
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
                text: ["低", "高"],
                calculable: true
              },
              series: [{
                type: "map",
                map: "china",
                nameProperty: "name", // 关键字段：匹配 JSON 中的 "name" 属性
                data: Object.entries(provinceData).map(([name, data]) => ({
                  name,
                  value: data.grade
                })),
                label: {
                  show: true,
                  fontSize: 12,
                  color: "#333"
                },
                emphasis: {
                  label: { show: true, color: "#000" },
                  itemStyle: { areaColor: "#ffe08c" }
                }
              }]
            };

            chart.setOption(option);
            createLegend(); // 生成图例
          })
          .catch((error) => {
            console.error("地图加载失败:", error);
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
              <div style="margin: 8px 0; display: flex; align-items: center">
                <div style="width: 20px; height: 20px; background: ${grade.color}; margin-right: 10px; border-radius: 3px"></div>
                <span style="font-size: 14px">${grade.desc}</span>
              </div>
            `
          )
          .join("");
      }

      // 页面加载后自动初始化
      window.onload = initializeChart;
    </script>
  </body>
</html>
