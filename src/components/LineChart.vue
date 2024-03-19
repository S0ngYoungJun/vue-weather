<template>
  <Line
    id="my-chart-id"
    :options="chartOptions"
    :data="chartData"
  />
</template>

<script>
import { Line } from 'vue-chartjs';
import { Chart, registerables } from "chart.js";

Chart.register(...registerables);

export default {
  name: 'LineChart',
  components: { Line },
  props: {
    forecastData: Array
  }, // forecastData prop 추가
  data() {
    return {
      chartData: {
        labels: [],
        datasets: [{
          label: 'Temperature',
          data: [],
          fill: false,
          borderColor: 'rgb(75, 192, 192)',
          tension: 0.1
        }]
      },
      chartOptions: {
        responsive: true,
        elements: {
          point: {
            radius: 5,
            backgroundColor: 'rgb(75, 192, 192)',
            hoverRadius: 8
          }
        }
      }
    };
  },
  watch: {
    // forecastData 변경 감지하여 차트 업데이트
    forecastData: {
      handler(newVal) {
        if (newVal) { // forecastData가 유효한 경우에만 업데이트
          this.updateChartData(newVal);
        }
      },
      immediate: true // 컴포넌트가 생성될 때도 실행
    }
  },
  methods: {
    // 차트 데이터 업데이트
    updateChartData(forecastData) {
      this.chartData.labels = forecastData.map(item => this.convertUtcToKst(item.dt_txt));
      this.chartData.datasets[0].data = forecastData.map(item => item.main.temp);
    },
    // UTC 시간을 KST 시간으로 변환하는 메서드
    convertUtcToKst(utcTime) {
      const utcDateTime = new Date(utcTime);
      const kstDateTime = new Date(utcDateTime.getTime() + (9 * 60 * 60 * 1000));
      return `${kstDateTime.getFullYear()}-${(kstDateTime.getMonth() + 1).toString().padStart(2, '0')}-${kstDateTime.getDate().toString().padStart(2, '0')} ${kstDateTime.getHours().toString().padStart(2, '0')}:${kstDateTime.getMinutes().toString().padStart(2, '0')}:${kstDateTime.getSeconds().toString().padStart(2, '0')}`;
    }
  }
};
</script>
