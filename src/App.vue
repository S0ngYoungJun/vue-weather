<template>
  <div>
    <div class="weather-app">
      <div class="weather-info">
        <h1>{{ cityName }}</h1>
        <p>현재 온도: {{ currentTemperature }} °C</p>
        <p>체감 온도: {{ feelsLikeTemperature }} °C</p>
        <p>24시간 중 최고 온도: {{ maxTemperature }} °C</p>
        <p>24시간 중 최저 온도: {{ minTemperature }} °C</p>
        <p>현재 날씨: {{ currentWeather }}</p>
        <img v-if="weatherIcon" :src="'http://openweathermap.org/img/w/' + weatherIcon + '.png'" alt="Weather Icon">
        <p v-if="forecastData.length > 0">일기 예보:</p>
        <!-- <ul v-for="forecast in forecastData" :key="forecast.dt">
          <li>{{ convertUtcToKst(forecast.dt_txt) }} {{ forecast.main.temp }} °C</li>
        </ul> -->
        <p v-if="airPollution">미세먼지 농도: {{ airPollution }} μg/m³</p>
        <div class="air-quality-box">
          <span class="air-quality-label">미세먼지 상태:</span>
          <div class="air-quality-indicator" :class="airQualityClass"></div>
          <span class="air-quality-text">{{ airQuality }}</span>
        </div>
      </div>
    </div>
    <div class="chart-container">
      <LineChart :forecastData="forecastData" />
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import LineChart from '@/components/LineChart.vue';

export default {
  components: {
    LineChart,
  },
  data() {
    return {
      cityName: 'Daejeon',
      currentTemperature: '',
      feelsLikeTemperature: '',
      currentWeather: '',
      weatherIcon: '',
      forecastData: [],
      airPollution: '',
    };
  },
  async created() {
    try {
      const apiKey = '7d1188304c32f5fd98a19d8e6440523b';
      const city = 'Daejeon';
      const currentWeatherUrl = `https://api.openweathermap.org/data/2.5/weather?q=${city}&units=metric&appid=${apiKey}`;
      const forecastWeatherUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${city}&units=metric&appid=${apiKey}`;
      const airPollutionUrl = `https://api.openweathermap.org/data/2.5/air_pollution?lat=36.3504&lon=127.3845&appid=${apiKey}`;

      const [currentWeatherResponse, forecastWeatherResponse, airPollutionResponse] = await Promise.all([
        axios.get(currentWeatherUrl),
        axios.get(forecastWeatherUrl),
        axios.get(airPollutionUrl),
      ]);

      this.currentTemperature = currentWeatherResponse.data.main.temp;
      this.feelsLikeTemperature = currentWeatherResponse.data.main.feels_like;
      this.currentWeather = currentWeatherResponse.data.weather[0].description;
      this.weatherIcon = currentWeatherResponse.data.weather[0].icon;
      this.forecastData = forecastWeatherResponse.data.list.slice(0, 8);
      this.airPollution = airPollutionResponse.data.list[0].components.pm10;

      console.log('Forecast data:', this.forecastData);
    } catch (error) {
      console.error(error);
    }
  },
  
  computed: {
    airQuality() {
      if (this.airPollution >= 0 && this.airPollution < 30) {
        return '쾌적';
      } else if (this.airPollution >= 30 && this.airPollution < 80) {
        return '보통(마스크 소지 권장)';
      } else if (this.airPollution >= 80 && this.airPollution < 150) {
        return '나쁨(마스크 착용 권장)';
      } else {
        return '매우나쁨(마스크 착용 필수!)';
      }
    },
    airQualityClass() {
      if (this.airPollution >= 0 && this.airPollution < 30) {
        return 'blue';
      } else if (this.airPollution >= 30 && this.airPollution < 80) {
        return 'green';
      } else if (this.airPollution >= 80 && this.airPollution < 150) {
        return 'orange';
      } else {
        return 'red';
      }
    },
    maxTemperature() {
      if (this.forecastData.length === 0) return '';
      return Math.max(...this.forecastData.map(data => data.main.temp));
    },
    minTemperature() {
      if (this.forecastData.length === 0) return '';
      return Math.min(...this.forecastData.map(data => data.main.temp));
    }
  },
  methods: {
    convertUtcToKst(utcTime) {
      const utcDateTime = new Date(utcTime);
      const kstDateTime = new Date(utcDateTime.getTime() + (9 * 60 * 60 * 1000));
      return `${kstDateTime.getFullYear()}-${(kstDateTime.getMonth() + 1).toString().padStart(2, '0')}-${kstDateTime.getDate().toString().padStart(2, '0')} ${kstDateTime.getHours().toString().padStart(2, '0')}:${kstDateTime.getMinutes().toString().padStart(2, '0')}:${kstDateTime.getSeconds().toString().padStart(2, '0')}`;
    },
  },
};
</script>

<style>
.weather-app {
  font-family: Arial, sans-serif;
  text-align: center;
  margin-bottom: 20px;
}

.chart-container {
  width: 80%;
  margin: 0 auto;
}

.air-quality-box {
  display: flex;
  align-items: center;
}

.air-quality-label {
  margin-right: 10px;
}

.air-quality-indicator {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  margin-right: 5px;
}

.blue {
  background-color: blue;
}

.green {
  background-color: green;
}

.orange {
  background-color: orange;
}

.red {
  background-color: red;
}
</style>
