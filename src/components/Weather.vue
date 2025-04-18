<template>
  <div class="app-container">
    <!-- Navbar -->
    <header class="navbar">
      <h1 class="logo">🌤️ WeatherApp</h1>
    </header>

    <!-- Main -->
    <main class="content">
      <!-- Search -->
      <form @submit.prevent="getWeather" class="search-form">
        <input
          v-model="city"
          type="text"
          placeholder="Enter city"
          required
          class="input-field"
        />
        <button type="submit" class="search-button">Search</button>
      </form>

      <!-- Weather Data -->
      <section v-if="weather" class="weather-section">
        <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>
        <img 
          :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"
          alt="Weather icon" class="mx-[35%]"
        />
        <p>🌡️ Temperature: {{ weather.main.temp }}°C</p>
        <p>💧 Humidity: {{ weather.main.humidity }}%</p>
        <p>🌥️ Condition: {{ weather.weather[0].description }}</p>
      </section>

      <!-- Forecast -->
      <section v-if="forecast" class="forecast-section">
        <h3>🔮 5-Time Forecast:</h3>
        <ul>
          <li v-for="(item, i) in forecast.list.slice(0, 5)" :key="i">
            {{ item.dt_txt }} — {{ item.main.temp }}°C — {{ item.weather[0].description }}
          </li>
        </ul>
      </section>

      <!-- Error -->
      <p v-if="error" class="error-message">{{ error }}</p>
    </main>

    <!-- Footer -->
    <footer class="footer">
      <p>© 2025 WeatherApp — Made with ❤️</p>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const city = ref('')
const weather = ref(null)
const forecast = ref(null)
const error = ref('')

const API_KEY = 'f7b0f8a61c20f0a0a26420a320cd3df3'

const getWeather = async () => {
  error.value = ''
  weather.value = null
  forecast.value = null
  try {
    const res = await axios.get(
      `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&units=metric&lang=en&appid=${API_KEY}`
    )
    weather.value = res.data

    const forecastRes = await axios.get(
      `https://api.openweathermap.org/data/2.5/forecast?q=${city.value}&units=metric&lang=en&appid=${API_KEY}`
    )
    forecast.value = forecastRes.data
  } catch (err) {
    error.value = '⚠️ City not found. Please try another.'
  }
}
</script>

<style scoped>
/* General */
.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: #121212;
  color: #e0e0e0;
  font-family: 'Segoe UI', sans-serif;
}

/* Navbar */
.navbar {
  padding: 1rem 2rem;
  background-color: #1f1f1f;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
}
.logo {
  margin: 0;
  font-size: 1.8rem;
}

/* Content */
.content {
  flex: 1;
  padding: 2rem;
  max-width: 800px;
  margin: 0 auto;
}

/* Form */
.search-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
}
.input-field {
  padding: 0.7rem 1rem;
  width: 80%;
  max-width: 400px;
  border-radius: 8px;
  border: 1px solid #333;
  background-color: #1e1e1e;
  color: #e0e0e0;
}
.search-button {
  padding: 0.6rem 1.2rem;
  border: none;
  background: #00bcd4;
  color: white;
  border-radius: 8px;
  cursor: pointer;
  transition: 0.3s;
}
.search-button:hover {
  background: #008ba3;
}

/* Weather & Forecast */
.weather-section,
.forecast-section {
  margin-top: 2rem;
  padding: 1.5rem;
  border-radius: 12px;
  background-color: #1e1e1e;
  box-shadow: 0 0 15px rgba(0, 255, 255, 0.05);
  text-align: center;
}

.forecast-section ul {
  margin-top: 1rem;
  padding-left: 0;
  list-style: none;
}
.forecast-section li {
  padding: 0.5rem 0;
  border-bottom: 1px solid #333;
}

/* Error */
.error-message {
  margin-top: 2rem;
  color: #ff6b6b;
  text-align: center;
}

/* Footer */
.footer {
  padding: 1rem 2rem;
  text-align: center;
  background-color: #1f1f1f;
  font-size: 0.9rem;
  color: #aaa;
}
</style>
