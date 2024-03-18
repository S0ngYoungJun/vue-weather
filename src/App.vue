<template>
  <div class="weather-app">
    <div class="weather-info">
      <h1>{{ cityName }}</h1>
      <p>현재 온도: {{ currentTemperature }} °C</p>
      <p>체감 온도: {{ feelsLikeTemperature }} °C</p>
      <p v-if="forecastData.length > 0">일기 예보:</p>
      <ul v-for="forecast in forecastData" :key="forecast.dt">
        <li>{{ forecast.dt_txt }} - {{ forecast.main.temp }} °C</li>
      </ul>
      <p v-if="airPollution">미세먼지 농도: {{ airPollution }} μg/m³</p>
    </div>
    <div v-if="error" class="error-message">
      {{ errorMessage }}
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      cityName: 'Daejeon',
      currentTemperature: '',
      feelsLikeTemperature: '',
      forecastData: [],
      airPollution: '',
      error: false,
      errorMessage: ''
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
        axios.get(airPollutionUrl)
      ]);

      this.currentTemperature = currentWeatherResponse.data.main.temp;
      this.feelsLikeTemperature = currentWeatherResponse.data.main.feels_like;
      this.forecastData = forecastWeatherResponse.data.list.slice(0, 8); // 5일 동안의 3시간 간격 데이터를 가져옴
      this.airPollution = airPollutionResponse.data.list[0].components.pm10;
    } catch (error) {
      this.error = true;
      this.errorMessage = '날씨 정보를 불러올 수 없습니다.';
      console.error(error);
    }
  }
};
</script>

<style>
.weather-app {
  font-family: Arial, sans-serif;
  text-align: center;
}

.weather-info {
  margin-top: 50px;
}

.error-message {
  color: red;
}
</style>
