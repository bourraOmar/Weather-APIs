<template>
  <div class="app-container">
    <h1>🌦️ تطبيق الطقس</h1>
    <form @submit.prevent="getWeather">
      <input v-model="city" type="text" placeholder="أدخل اسم المدينة" required />
      <button type="submit">🔍 بحث</button>
    </form>

    <div v-if="weather" class="weather-card">
      <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>
      <img :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`" alt="Weather icon" />
      <p>🌡️ درجة الحرارة: {{ weather.main.temp }}°C</p>
      <p>🌬️ الرطوبة: {{ weather.main.humidity }}%</p>
      <p>☁️ الحالة: {{ weather.weather[0].description }}</p>
    </div>

    <p v-if="error" class="error">{{ error }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const city = ref('')
const weather = ref(null)
const error = ref('')

const API_KEY = 'YOUR_API_KEY' // عوّضها بالمفتاح ديالك من OpenWeatherMap

const getWeather = async () => {
  error.value = ''
  weather.value = null
  try {
    const res = await axios.get(
      `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&units=metric&lang=ar&appid=${API_KEY}`
    )
    weather.value = res.data
  } catch (err) {
    error.value = '⚠️ لم يتم العثور على المدينة. جرب اسم آخر.'
  }
}
</script>

<style scoped>
.app-container {
  max-width: 600px;
  margin: auto;
  padding: 2rem;
  text-align: center;
  font-family: Arial, sans-serif;
}
input {
  padding: 0.5rem;
  width: 60%;
  font-size: 1rem;
  margin-right: 0.5rem;
}
button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
}
.weather-card {
  margin-top: 2rem;
  padding: 1rem;
  border-radius: 12px;
  background-color: #f0f8ff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
.error {
  margin-top: 1rem;
  color: red;
}
@media (max-width: 600px) {
  input {
    width: 100%;
    margin-bottom: 1rem;
  }
  button {
    width: 100%;
  }
}
</style>
