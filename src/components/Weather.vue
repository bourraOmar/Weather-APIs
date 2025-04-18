<template>
  <div class="app-container">
    <!-- Navbar -->
    <header class="navbar">
      <div class="navbar-content">
        <h1 class="logo">🌤️ WeatherApp</h1>
        <p class="tagline">Real-time weather forecasts</p>
      </div>
    </header>

    <!-- Main -->
    <main class="content">
      <!-- Search -->
      <form @submit.prevent="getWeather" class="search-form">
        <div class="search-group">
          <input
            v-model="city"
            type="text"
            placeholder="Enter city name..."
            required
            class="input-field"
          />
          <button type="submit" class="search-button">
            <span>Search</span>
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="11" cy="11" r="8"></circle>
              <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
            </svg>
          </button>
        </div>
      </form>

      <!-- Weather Data -->
      <section v-if="weather" class="weather-section">
        <div class="weather-header">
          <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>
          <p class="current-date">{{ formatDate(weather.dt) }}</p>
        </div>
        
        <div class="weather-main">
          <div class="weather-icon">
            <img 
              :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@4x.png`"
              :alt="weather.weather[0].description"
            />
            <p class="weather-description">{{ capitalizeFirstLetter(weather.weather[0].description) }}</p>
          </div>
          
          <div class="weather-temp">
            <p class="temperature">{{ Math.round(weather.main.temp) }}°C</p>
            <div class="temp-range">
              <span>H: {{ Math.round(weather.main.temp_max) }}°</span>
              <span>L: {{ Math.round(weather.main.temp_min) }}°</span>
            </div>
          </div>
        </div>
        
        <div class="weather-details">
          <div class="detail-item">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
              <circle cx="9" cy="7" r="4"></circle>
              <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
              <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
            </svg>
            <div>
              <p class="detail-label">Humidity</p>
              <p class="detail-value">{{ weather.main.humidity }}%</p>
            </div>
          </div>
          
          <div class="detail-item">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M4 15s1-1 4-1 5 2 8 2 4-1 4-1V3s-1 1-4 1-5-2-8-2-4 1-4 1z"></path>
              <line x1="4" y1="22" x2="4" y2="15"></line>
            </svg>
            <div>
              <p class="detail-label">Wind</p>
              <p class="detail-value">{{ weather.wind.speed }} km/h</p>
            </div>
          </div>
          
          <div class="detail-item">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="12" cy="12" r="10"></circle>
              <polyline points="12 6 12 12 16 14"></polyline>
            </svg>
            <div>
              <p class="detail-label">Pressure</p>
              <p class="detail-value">{{ weather.main.pressure }} hPa</p>
            </div>
          </div>
        </div>
      </section>

      <!-- Forecast -->
      <section v-if="forecast" class="forecast-section">
        <h3>5-Day Forecast</h3>
        <div class="forecast-cards">
          <div v-for="(item, i) in filteredForecast" :key="i" class="forecast-card">
            <p class="forecast-day">{{ formatDay(item.dt) }}</p>
            <img 
              :src="`https://openweathermap.org/img/wn/${item.weather[0].icon}@2x.png`"
              :alt="item.weather[0].description"
            />
            <div class="forecast-temps">
              <span class="forecast-temp-high">{{ Math.round(item.main.temp_max) }}°</span>
              <span class="forecast-temp-low">{{ Math.round(item.main.temp_min) }}°</span>
            </div>
          </div>
        </div>
      </section>

      <!-- Error -->
      <div v-if="error" class="error-message">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="10"></circle>
          <line x1="12" y1="8" x2="12" y2="12"></line>
          <line x1="12" y1="16" x2="12.01" y2="16"></line>
        </svg>
        <p>{{ error }}</p>
      </div>
    </main>

    <!-- Footer -->
    <footer class="footer">
      <p>© 2025 WeatherApp — Made with ❤️ by weather enthusiasts</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'

const city = ref('')
const weather = ref(null)
const forecast = ref(null)
const error = ref('')

const API_KEY = 'f7b0f8a61c20f0a0a26420a320cd3df3'

const filteredForecast = computed(() => {
  if (!forecast.value) return []
  // Get one forecast per day at noon
  return forecast.value.list.filter((item, index) => {
    return item.dt_txt.includes('12:00:00') || index === 0
  }).slice(0, 5)
})

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
    error.value = 'City not found. Please try another location.'
  }
}

const capitalizeFirstLetter = (string) => {
  return string.charAt(0).toUpperCase() + string.slice(1)
}

const formatDate = (timestamp) => {
  const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' }
  return new Date(timestamp * 1000).toLocaleDateString('en-US', options)
}

const formatDay = (timestamp) => {
  const options = { weekday: 'short' }
  return new Date(timestamp * 1000).toLocaleDateString('en-US', options)
}
</script>

<style scoped>
/* General */
.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: linear-gradient(135deg, #121212 0%, #1a1a2e 100%);
  color: #f8f9fa;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  line-height: 1.6;
}

/* Navbar */
.navbar {
  padding: 1.5rem 2rem;
  background-color: rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.navbar-content {
  max-width: 1200px;
  margin: 0 auto;
}

.logo {
  margin: 0;
  font-size: 1.8rem;
  font-weight: 700;
  background: linear-gradient(90deg, #00bcd4 0%, #00d4b8 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  display: inline-block;
}

.tagline {
  margin: 0.25rem 0 0;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.7);
}

/* Content */
.content {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

/* Search Form */
.search-form {
  margin-bottom: 2rem;
}

.search-group {
  display: flex;
  max-width: 600px;
  margin: 0 auto;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  border-radius: 12px;
  overflow: hidden;
}

.input-field {
  flex: 1;
  padding: 1rem 1.5rem;
  border: none;
  background-color: rgba(255, 255, 255, 0.1);
  color: #f8f9fa;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.input-field:focus {
  outline: none;
  background-color: rgba(255, 255, 255, 0.15);
}

.input-field::placeholder {
  color: rgba(255, 255, 255, 0.5);
}

.search-button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0 1.5rem;
  border: none;
  background: linear-gradient(90deg, #00bcd4 0%, #00d4b8 100%);
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.search-button:hover {
  background: linear-gradient(90deg, #00a5c4 0%, #00c4a8 100%);
}

.search-button svg {
  stroke: white;
}

/* Weather Section */
.weather-section {
  margin: 2rem auto;
  padding: 2rem;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
  max-width: 800px;
}

.weather-header {
  text-align: center;
  margin-bottom: 2rem;
}

.weather-header h2 {
  margin: 0;
  font-size: 2rem;
  font-weight: 700;
}

.current-date {
  margin: 0.5rem 0 0;
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.9rem;
}

.weather-main {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 2rem;
  margin-bottom: 2rem;
}

.weather-icon {
  text-align: center;
}

.weather-icon img {
  height: 120px;
  width: auto;
}

.weather-description {
  margin: 0.5rem 0 0;
  font-size: 1.1rem;
  color: rgba(255, 255, 255, 0.9);
}

.weather-temp {
  text-align: center;
}

.temperature {
  font-size: 3.5rem;
  font-weight: 700;
  margin: 0;
  line-height: 1;
}

.temp-range {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-top: 0.5rem;
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
}

.weather-details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1.5rem;
  margin-top: 2rem;
}

.detail-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.detail-item svg {
  stroke: #00bcd4;
}

.detail-label {
  margin: 0;
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.6);
}

.detail-value {
  margin: 0.25rem 0 0;
  font-size: 1.1rem;
  font-weight: 600;
}

/* Forecast Section */
.forecast-section {
  margin: 3rem auto;
  max-width: 800px;
}

.forecast-section h3 {
  font-size: 1.5rem;
  margin-bottom: 1.5rem;
  text-align: center;
}

.forecast-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 1rem;
}

.forecast-card {
  padding: 1.5rem 1rem;
  text-align: center;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.05);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.forecast-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
}

.forecast-day {
  margin: 0 0 1rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.9);
}

.forecast-card img {
  height: 50px;
  width: auto;
  margin: 0 auto;
}

.forecast-temps {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.forecast-temp-high {
  font-weight: 600;
}

.forecast-temp-low {
  color: rgba(255, 255, 255, 0.6);
}

/* Error Message */
.error-message {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 1.5rem;
  margin: 2rem auto;
  max-width: 600px;
  background: rgba(255, 107, 107, 0.1);
  border-radius: 12px;
  border: 1px solid rgba(255, 107, 107, 0.2);
  color: #ff6b6b;
}

.error-message svg {
  stroke: #ff6b6b;
}

/* Footer */
.footer {
  padding: 1.5rem 2rem;
  text-align: center;
  background-color: rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.6);
}

/* Responsive Adjustments */
@media (max-width: 768px) {
  .weather-main {
    flex-direction: column;
    text-align: center;
  }
  
  .weather-details {
    grid-template-columns: 1fr;
  }
  
  .forecast-cards {
    grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  }
}

@media (max-width: 480px) {
  .navbar {
    padding: 1rem;
  }
  
  .content {
    padding: 1rem;
  }
  
  .search-group {
    flex-direction: column;
  }
  
  .search-button {
    justify-content: center;
    padding: 0.75rem;
  }
  
  .weather-section {
    padding: 1.5rem;
  }
  
  .temperature {
    font-size: 3rem;
  }
}
</style>