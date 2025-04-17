<template>
  <div class="min-h-screen flex flex-col items-center justify-center p-4 bg-gradient-to-br from-sky-100 to-blue-300">
    <h1 class="text-3xl font-bold mb-6 text-blue-800">🌦️ Weather App</h1>

    <form @submit.prevent="getWeather" class="flex flex-col sm:flex-row gap-4 w-full max-w-md">
      <input
        v-model="city"
        type="text"
        placeholder="Enter city name"
        required
        class="flex-1 px-4 py-2 border rounded-md shadow focus:outline-none focus:ring-2 focus:ring-blue-400"
      />
      <button
        type="submit"
        class="bg-blue-600 text-white px-6 py-2 rounded-md hover:bg-blue-700 transition"
      >
        Search
      </button>
    </form>

    <div v-if="weather" class="mt-8 bg-white p-6 rounded-xl shadow-lg text-center w-full max-w-md">
      <h2 class="text-xl font-semibold mb-2">
        {{ weather.name }}, {{ weather.sys.country }}
      </h2>
      <img
        :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"
        :alt="weather.weather[0].description"
        class="mx-auto"
      />
      <p class="text-lg">🌡️ Temperature: {{ weather.main.temp }}°C</p>
      <p>💧 Humidity: {{ weather.main.humidity }}%</p>
      <p>🌥️ Condition: {{ weather.weather[0].description }}</p>
    </div>

    <p v-if="error" class="text-red-600 mt-4 font-medium">{{ error }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const city = ref('')
const weather = ref(null)
const error = ref('')

const API_KEY = 'f7b0f8a61c20f0a0a26420a320cd3df3' // don't expose this in production

const getWeather = async () => {
  error.value = ''
  weather.value = null
  try {
    const res = await axios.get(
      `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&units=metric&lang=en&appid=${API_KEY}`
    )
    weather.value = res.data
  } catch (err) {
    error.value = '⚠️ City not found. Please try another one.'
  }
}
</script>
