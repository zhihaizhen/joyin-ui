<!-- /* eslint-disable */
<template>
  <div class="container">
    <div class="tools">
      数量：<a-input v-model="inputNum"></a-input>
      宽度: <a-input v-model="chartWidth"></a-input>
      高度: <a-input v-model="chartHeight"></a-input>
      <a-button @click="onUpdate">确定</a-button>
    </div>
    <div class="legend">
      <span>日期：{{ candleInfo.time?.year }}-{{ candleInfo.time?.month }}-{{ candleInfo.time?.day }}</span>
      <span>最高价: {{ candleInfo.high ? candleInfo.high.toFixed(4) : '-' }}</span> 
      <span>开盘价: {{ candleInfo.open ? candleInfo.open.toFixed(4) : '-' }} </span>
      <span>最低价: {{ candleInfo.low ? candleInfo.low.toFixed(4) : '-' }} </span>
      <span>收盘价: {{ candleInfo.close ? candleInfo.close.toFixed(4) : '-' }} </span>
    </div>
    <div class="legendMa">
      <span>MA3: {{ sma3Info?.value ? sma3Info.value?.toFixed(2) : '-' }} </span>
      <span>MA5: {{ sma5Info?.value ? sma5Info.value?.toFixed(2) : '-' }} </span>
      <span>MA10: {{ sma10Info?.value ? sma10Info.value?.toFixed(2) : '-' }} </span>
    </div>
    
    <div id="chartContainer" class="main" style="width: 1000px; height: 500px;">
    </div>
  </div>
  
</template>

<script>
import { createChart, CrosshairMode } from 'lightweight-charts';
export default {
  name: 'demoTest',
  data () {
    return {
      candleInfo: {},
      sma3Info: {},
      sma5Info: {},
      sma10Info: {},
      chartInstance: null,
      chartSeries: null,
      inputNum: 1000,
      chartWidth: 800,
      chartHeight: 500,

      sma3Line: null,
      sma5Line: null,
      sma10Line: null
    }
  },
  mounted() {
    this.initChart();
  },
  methods: {
    onUpdate() {
      this.chartInstance.applyOptions({
        width: this.chartWidth,
        height: this.chartHeight,
      });
      const barData = this.generateBarsData();
      this.chartSeries.setData(barData);
      this.sma3Line.setData(this.calculateSMA(barData, 3));
      this.sma5Line.setData(this.calculateSMA(barData, 5));
      this.sma10Line.setData(this.calculateSMA(barData, 10));
    },  
    
    initChart() {
      let chart = createChart(document.getElementById('chartContainer'), {
        width: this.chartWidth,
        height: this.chartHeight,
        
        // layout: {
        //   background: {
        //           type: 'solid',
        //           color: '#000000',
        //       },
        //   textColor: 'rgba(255, 255, 255, 0.9)',
        // },
        // grid: {
        //   vertLines: {
        //     color: 'rgba(255, 255, 255, 1)',
        //   },
        //   horzLines: {
        //     color: 'rgba(255, 255, 255, 1)',
        //   },
        // },
        crosshair: {
          mode: CrosshairMode.Normal,
        },
        rightPriceScale: {
          borderColor: 'rgba(197, 203, 206, 0.8)',
        },
        timeScale: {
          borderColor: 'rgba(197, 203, 206, 0.8)',
        },
      });
      this.chartInstance = chart;
      this.chartSeries = chart.addCandlestickSeries({
        upColor: 'red',
        downColor: 'green',
        borderDownColor: 'green',
        borderUpColor: 'red',
        wickDownColor: 'green',
        wickUpColor: 'red',
      });
      const myData = this.generateBarsData();
      this.chartSeries.setData(myData);

      this.sma3Line = chart.addLineSeries({
        color: 'rgba(0, 94, 157, 1)',
        lineWidth: 1,
      });
      this.sma3Line.setData(this.calculateSMA(myData, 3));
      this.sma5Line = chart.addLineSeries({
        color: 'rgba(255, 165, 0, 1)',
        lineWidth: 1,
      });
      this.sma5Line.setData(this.calculateSMA(myData, 5));
      this.sma10Line = chart.addLineSeries({
        color: 'rgba(238, 130, 238, 1)',
        lineWidth: 1,
      });
      this.sma10Line.setData(this.calculateSMA(myData, 10));

      chart.subscribeCrosshairMove((param) => {
        if (param.time) {
          this.candleInfo = param.seriesData.get(this.chartSeries);
          
          this.sma3Info = param.seriesData.get(this.sma3Line);
          this.sma5Info = param.seriesData.get(this.sma5Line);
          this.sma10Info = param.seriesData.get(this.sma10Line);
        }else {
          this.candleInfo =  {};
          this.sma10Info = {};
        }
      });
    },
    generateBarsData() {
      let res = [];
      const cDate = new Date();
      const timeTo = {
        day: cDate.getDay(),
        month: cDate.getMonth() + 1,
        year: cDate.getFullYear()
      };
      cDate.setDate(cDate.getDate() - this.inputNum);
      const timeFrom = {
        day: cDate.getDay(),
        month: cDate.getMonth() + 1,
        year: cDate.getFullYear()
      }
      const period = {
        timeFrom,
        timeTo
      };
      let controlPoints = generateControlPoints(res, period);
      for (let i = 0; i < controlPoints.length - 1; i++) {
        let left = controlPoints[i];
        let right = controlPoints[i + 1];
        fillBarsSegment(left, right, res);
      }
      return res;

      function fillBarsSegment(left, right, points) {
        let deltaY = right.price - left.price;
        let deltaX = right.index - left.index;
        let angle = deltaY / deltaX;
        for (let i = left.index; i <= right.index; i++) {
          let basePrice = left.price + (i - left.index) * angle;
          let openNoise = (0.1 - Math.random() * 0.2) + 1;
          let closeNoise = (0.1 - Math.random() * 0.2) + 1;
          let open = basePrice * openNoise;
          let close = basePrice * closeNoise;
          let high = Math.max(basePrice * (1 + Math.random() * 0.2), open, close);
          let low = Math.min(basePrice * (1 - Math.random() * 0.2), open, close);
          points[i].open = open;
          points[i].high = high;
          points[i].low = low;
          points[i].close = close;
        }
      }

      function generateControlPoints(res, period, dataMultiplier) {
        let time = period !== undefined ? period.timeFrom : {
          day: 1,
          month: 1,
          year: 2018
        };
        let timeTo = period !== undefined ? period.timeTo : {
          day: 1,
          month: 1,
          year: 2019
        };
        let days = getDiffDays(time, timeTo);
        dataMultiplier = dataMultiplier || 1;
        let controlPoints = [];
        controlPoints.push({
          index: 0,
          price: getRandomPrice() * dataMultiplier
        });
        for (let i = 0; i < days; i++) {
          if (i > 0 && i < days - 1 && Math.random() < 0.05) {
            controlPoints.push({
              index: i,
              price: getRandomPrice() * dataMultiplier
            });
          }
          res.push({
            time: time
          });
          time = nextBusinessDay(time);
        }
        controlPoints.push({
          index: res.length - 1,
          price: getRandomPrice() * dataMultiplier
        });
        return controlPoints;
      }

      function getDiffDays(dateFrom, dateTo) {
        let df = convertBusinessDayToUTCTimestamp(dateFrom);
        let dt = convertBusinessDayToUTCTimestamp(dateTo);
        let diffTime = Math.abs(dt.getTime() - df.getTime());
        return Math.ceil(diffTime / (1000 * 60 * 60 * 24));
      }

      function convertBusinessDayToUTCTimestamp(date) {
        return new Date(Date.UTC(date.year, date.month - 1, date.day, 0, 0, 0, 0));
      }

      function nextBusinessDay(time) {
        let d = convertBusinessDayToUTCTimestamp({
          year: time.year,
          month: time.month,
          day: time.day + 1
        });
        return {
          year: d.getUTCFullYear(),
          month: d.getUTCMonth() + 1,
          day: d.getUTCDate()
        };
      }

      function getRandomPrice() {
        return 10 + Math.round(Math.random() * 10000) / 100;
      }
    },
    calculateSMA(data, count) {
      let avg = function(data) {
        let sum = 0;
        for (let i = 0; i < data.length; i++) {
          sum += data[i].close;
        }
        return sum / data.length;
      };
      let result = [];
      for (let i = count - 1, len = data.length; i < len; i++) {
        let val = avg(data.slice(i - count + 1, i));
        result.push({
          time: data[i].time,
          value: val
        });
      }
      return result;
    },
    formatDate(date, format = "YYYY-MM-DD") {
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, "0");
      const day = String(date.getDate()).padStart(2, "0");
      let formattedDate = format.replace("YYYY", year);
      formattedDate = formattedDate.replace("MM", month);
      formattedDate = formattedDate.replace("DD", day);
      return formattedDate;
    }
  }
}
</script>

<style>
</style>


<!-- 
<style scoped lang="less">
.container {
  display: flex;
  justify-content: center;
}
.tools {
  position: absolute;
	left: 150px;
	top: 10px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  input {
    width: 100px;
    margin-right: 10px;
  }
}
.legendMa { 
  left: 50px !important;
  >span {
    color: rgba(0, 94, 157, 1);
    &:nth-child(2) {
      color: rgba(255, 165, 0, 1);
    }
    &:nth-child(3) {
      color: rgba(238, 130, 238, 1);
    }
  }
}
.legend, .legendMa {
	position: absolute;
	left: 150px;
	top: 50px;
	z-index: 1;
	font-size: 12px;
	line-height: 18px;
	font-weight: 300;
  color: black;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-start;
  >span {
    width: 100px;
  }
}
.main {
  margin-top: 50px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
</style> -->
