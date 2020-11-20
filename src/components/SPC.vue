<template>
  <div class="chart">
    <div id="spc_chart" class="spc_chart" @click="sendMsgtoParent"/>
  </div>
</template>

<script>
import Plotly from "plotly.js-dist";
// import twLocale from "plotly.js-locales/zh-tw";
// import cnLocale from "plotly.js-locales/zh-cn";

export default {
  name: "spc_chart",
  props: {
    chartTitle: {
      type: String,
      default: "SPC Chart"
    },
    chartData: {
      type: Object,
      default: () => ({
        x: [0], // 時間序列
        y: [0] //defect 數量
      })
    },
    ucl: {
      // 管制上限
      type: Number,
      default: 5
    },
    lcl: {
      // 管制下限
      type: Number,
      default: -5
    },
    cl: {
      // 中位數
      type: Number,
      default: 0
    }
  },
  data: () => ({
    points: [],
    theme: {
      primary: "#2196F3",
      secondary: "#82B1FF",
      accent: "#82B1FF",
      error: "#FF5252",
      info: "#2196F3",
      success: "#4CAF50",
      warning: "#FFC107",
      background: "#E3F2FD",
      gridColor: "#d3d3d3",
      text: "#263238"
    }
  }),
  watch: {
    chartData: {
      handler() {
        this.drawPlot();
      },
      deep: true
    },
    ucl() {
      this.drawPlot();
    },
    lcl() {
      this.drawPlot();
    },
    cl() {
      this.drawPlot();
    }
  },
  methods: {
    myEventHandler() {
      this.drawPlot();
    },
    drawPlot() {
      // let offset = 0.75;
      // if (this.$i18n.locale === "tw") {
      //   Plotly.register(twLocale);
      //   Plotly.setPlotConfig({locale: 'zh-TW'})
      // } else if (this.$i18n.locale === "cn") {
      //   Plotly.register(cnLocale);
      //   Plotly.setPlotConfig({locale: 'zh-CN'})
      // }

      const selectorOptions = {
        buttons: [
          {
            step: "month",
            // stepmode: 'backward',
            count: 1,
            label: "1m"
          },
          {
            step: "month",
            stepmode: "backward",
            count: 6,
            label: "6m"
          },
          {
            step: "year",
            stepmode: "todate",
            count: 1,
            label: "YTD"
          },
          {
            step: "year",
            stepmode: "backward",
            count: 1,
            label: "1y"
          },
          {
            step: "all"
          }
        ],
        yanchor: "bottom",
        bgcolor: this.theme.secondary,
        bordercolor: this.theme.text,
        font: {
          family: "Courier",
          size: 16,
          color: this.theme.text
        }
      };


      const Data = {
        type: "scatter",
        x: this.chartData.x,
        y: this.chartData.y,
        mode: "lines+markers",
        name: "spc.DATA",
        showlegend: true,
        hoverinfo: "all",
        line: {
          // color: "blue",
          width: 2
        },
        marker: {
          // color: "blue",
          size: 7,
          symbol: "circle"
        }
      };

      const Viol = {
        type: "scatter",
        x: this.dataViol.x,
        y: this.dataViol.y,
        mode: "markers",
        name: "VIOLATION",
        showlegend: true,
        marker: {
          color: this.theme.error,
          line: { width: 4 },
          opacity: 1,
          size: 13,
          symbol: "circle-open"
        }
      };

      const CL = {
        type: "scatter",
        x: [
          this.chartData.x[0],
          this.dataLength,
          null,
          this.chartData.x[0],
          this.dataLength
        ],
        dx: 0,
        y: [this.lcl, this.lcl, null, this.ucl, this.ucl],
        mode: "lines",
        name: "spc.LCL_UCL",
        showlegend: true,
        line: {
          color: this.theme.error,
          width: 2,
          dash: "dash"
        }
      };

      const Center = {
        type: "scatter",
        x: [this.chartData.x[0] || 0, this.dataLength],
        y: [0, 0],
        mode: "lines",
        name: "spc.CL",
        showlegend: true,
        line: {
          color: 'grey',
          width: 2,
          opacity: 0.5, 
          
        }
      };

      const CCL = [
        {
          type: "line",
          xref: "paper",
          x0: 0,
          x1: 0.8,
          y0: this.cl,
          y1: this.cl,
          line: {
            color: 'grey',
            opacity: 0.5, 
            width: 2
          }
        },
        {
          type: "line",
          xref: "paper",
          x0: 0,
          y0: this.lcl,
          x1: 0.8,
          y1: this.lcl,
          line: {
            color: this.theme.error,
            opacity: 0.8, 
            width: 2,
            dash: "dash"
          }
        },
        {
          type: "line",
          xref: "paper",
          x0: 0,
          y0: this.ucl,
          x1: 0.8,
          y1: this.ucl,
          line: {
            opacity: 0.8, 
            color: this.theme.error,
            width: 2,
            dash: "dash"
          }
        }
      ];

      let histo = {
        type: 'histogram',
        x: [...Array(this.chartData.y.length).keys()],
        y: this.chartData.y,
        name: 'Distribution',
        orientation: 'h',
        marker: {
          color: this.theme.primary,
          line: {
            color: this.theme.gridColor,
            width: 1
          }
        },
        xaxis: 'x2',
        yaxis: 'y2'
      };

      const data = [Data, Viol, CL, Center, histo];


      const layout = {
        title: this.chartTitle,
        shapes: CCL,
        xaxis: {
          domain: [0, 0.8], // 0 to 70% of width
          zeroline: false,
          gridcolor: this.theme.gridColor,
          rangeselector: selectorOptions,
          tickformat: "%Y-%m-%d\n%H:%M:%S",
        },
        yaxis: {
          zeroline: false,
          gridcolor: this.theme.gridColor,
          range: [this.dataMinMax.min, this.dataMinMax.max]
        },
        xaxis2: {
          domain: [0.85, 1] // 70 to 100% of width
        },
        yaxis2: {
          anchor: 'x2',
          showticklabels: false
        },
        template: {
          layout: {
            font: {
              family: "Courier",
              size: 18,
              color: this.theme.text
            },
            paper_bgcolor: this.theme.background,
            plot_bgcolor: this.theme.background
          }
        }
      };

      const options = {
        displaylogo: false
      };

      Plotly.newPlot("spc_chart", data, layout, options);
      
    },
    sendMsgtoParent(){
      this.$emit('getData',this.points)
    }
  },  
  ////////////////////////////////     ------------------end of methods
  computed: {
    dataLength() {
      return Math.abs(Math.ceil(Math.max(...this.chartData.x) * 1.2));
    },
    dataMinMax() {
      const min = Math.min(...this.chartData.y, this.lcl);
      const max = Math.max(...this.chartData.y, this.ucl);
      return {
        min: min - Math.abs(Math.ceil(min*1.2)),
        max: max + Math.abs(Math.ceil(max*0.5))
      };
    },
    dataViol() {
      let x = [];
      let y = [];

      this.chartData.y.reduce((acc, value, index) => {
        if (value > this.ucl || value < this.lcl) {
          x.push(this.chartData.x[index]);
          y.push(value);
        }
      });

      return { x, y };
    }
  },
  mounted() {
    let that =this
    this.drawPlot();

    var myPlot = document.getElementById('spc_chart')
    myPlot.on('plotly_click', function(data){
      console.log("data",data)
      var pts = [];
       
        pts.push(data.points[0].x)
        pts.push(data.points[0].y)
      that.points = pts
      console.log('pts',that.points)
      
    });

  },
  created() {
    window.addEventListener("resize", this.myEventHandler);
  },
  destroyed() {
    window.removeEventListener("resize", this.myEventHandler);
  }
};
</script>

<style scoped>
  .chart {
   
    padding-bottom: 20px;
    background-color: #E3F2FD
  }
</style>