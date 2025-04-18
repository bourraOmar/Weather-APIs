<template>
  <div class="flex flex-col min-h-screen bg-gradient-to-b from-blue-50 to-indigo-100">
    <!-- Navbar -->
    <nav class="bg-white shadow-md sticky top-0 z-10">
      <div class="max-w-6xl mx-auto px-4">
        <div class="flex justify-between items-center h-16">
          <!-- Logo and App Name -->
          <div class="flex items-center space-x-3">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-blue-500" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M17 18a5 5 0 0 0-10 0"></path>
              <line x1="12" y1="9" x2="12" y2="2"></line>
              <line x1="4.22" y1="10.22" x2="5.64" y2="11.64"></line>
              <line x1="1" y1="18" x2="3" y2="18"></line>
              <line x1="21" y1="18" x2="23" y2="18"></line>
              <line x1="18.36" y1="11.64" x2="19.78" y2="10.22"></line>
              <line x1="23" y1="22" x2="1" y2="22"></line>
              <polyline points="8 6 12 2 16 6"></polyline>
            </svg>
            <span class="font-semibold text-xl tracking-tight text-blue-700">WeatherVue</span>
          </div>
          
          <!-- Navigation Links -->
          <div class="hidden md:flex space-x-6">
            <a href="#" class="text-blue-700 font-medium">Home</a>
            <a href="#" class="text-gray-600 hover:text-blue-700 transition">Forecast</a>
            <a href="#" class="text-gray-600 hover:text-blue-700 transition">Maps</a>
            <a href="#" class="text-gray-600 hover:text-blue-700 transition">About</a>
          </div>
          
          <!-- Mobile menu button -->
          <div class="md:hidden">
            <button @click="toggleMobileMenu" class="text-gray-500 hover:text-blue-700 focus:outline-none">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path v-if="mobileMenuOpen" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                <path v-else stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
              </svg>
            </button>
          </div>
        </div>
        
        <!-- Mobile menu -->
        <div v-if="mobileMenuOpen" class="md:hidden pb-3">
          <a href="#" class="block py-2 text-blue-700 font-medium">Home</a>
          <a href="#" class="block py-2 text-gray-600 hover:text-blue-700 transition">Forecast</a>
          <a href="#" class="block py-2 text-gray-600 hover:text-blue-700 transition">Maps</a>
          <a href="#" class="block py-2 text-gray-600 hover:text-blue-700 transition">About</a>
        </div>
      </div>
    </nav>

    <!-- Main Content -->
    <main class="flex-grow flex flex-col items-center justify-center py-12 px-4">
      <div class="w-full max-w-md bg-white rounded-3xl shadow-xl overflow-hidden">
        <!-- Header -->
        <div class="bg-gradient-to-r from-blue-500 to-indigo-600 px-6 py-4">
          <h2 class="text-2xl font-bold text-white text-center">Weather Forecast</h2>
        </div>
        
        <!-- Search -->
        <div class="p-6">
          <div class="relative">
            <input
              v-model="city"
              @keyup.enter="fetchWeather"
              type="text"
              placeholder="Search for a city..."
              class="w-full pl-10 pr-4 py-3 rounded-xl border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-400 transition"
            />
            <span class="absolute left-3 top-3.5 text-gray-400">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <circle cx="11" cy="11" r="8"></circle>
                <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
              </svg>
            </span>
            <button
              @click="fetchWeather"
              class="absolute right-2 top-[5px] bg-blue-600 text-white p-2 rounded-lg hover:bg-blue-700 transition"
            >
              <span v-if="loading">
                <svg class="animate-spin h-5 w-5" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
              </span>
              <span v-else>Search</span>
            </button>
          </div>
          
         
        </div>
        
        <!-- Error message -->
        <div v-if="error" class="mx-6 mb-6 p-4 bg-red-50 border border-red-100 rounded-xl text-center">
          <p class="text-red-600">{{ error }}</p>
        </div>
        
        <!-- Weather loading -->
        <div v-if="loading && !error" class="flex justify-center items-center p-12">
          <svg class="animate-spin h-8 w-8 text-blue-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
        </div>
        
        <!-- Weather data -->
        <div v-if="weatherData && !loading" class="px-6 pb-8">
          <div class="bg-gradient-to-br from-blue-400 to-indigo-500 rounded-2xl p-6 text-white shadow-lg">
            <div class="flex justify-between items-center">
              <div>
                <h3 class="text-2xl font-bold">{{ weatherData.name }}</h3>
                <p class="text-blue-100 capitalize">{{ weatherData.weather[0].description }}</p>
              </div>
              <div class="weather-icon">
                <svg v-if="isCloudyWeather" xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 text-blue-100" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 10h-1.26A8 8 0 109 20h9a5 5 0 000-10z"></path>
                </svg>
                <svg v-else-if="isSunnyWeather" xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 text-yellow-300" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <circle cx="12" cy="12" r="5"></circle>
                  <line x1="12" y1="1" x2="12" y2="3"></line>
                  <line x1="12" y1="21" x2="12" y2="23"></line>
                  <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
                  <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
                  <line x1="1" y1="12" x2="3" y2="12"></line>
                  <line x1="21" y1="12" x2="23" y2="12"></line>
                  <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
                  <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
                </svg>
                <svg v-else-if="isRainyWeather" xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 text-blue-100" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M20 16.58A5 5 0 0018 7h-1.26A8 8 0 104 15.25"></path>
                  <line x1="8" y1="16" x2="8.01" y2="16"></line>
                  <line x1="8" y1="20" x2="8.01" y2="20"></line>
                  <line x1="12" y1="18" x2="12.01" y2="18"></line>
                  <line x1="12" y1="22" x2="12.01" y2="22"></line>
                  <line x1="16" y1="16" x2="16.01" y2="16"></line>
                  <line x1="16" y1="20" x2="16.01" y2="20"></line>
                </svg>
                <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 text-blue-100" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 10h-1.26A8 8 0 109 20h9a5 5 0 000-10z"></path>
                </svg>
              </div>
            </div>
            
            <div class="mt-6">
              <p class="text-5xl font-bold">{{ Math.round(weatherData.main.temp) }}°C</p>
              <p class="text-sm text-blue-100">Feels like {{ Math.round(weatherData.main.feels_like) }}°C</p>
            </div>
            
            <div class="grid grid-cols-3 gap-4 mt-6">
              <div class="flex flex-col items-center">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-blue-200" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M12 2v6"></path>
                  <path d="M12 22v-6"></path>
                  <path d="M4.93 4.93l4.24 4.24"></path>
                  <path d="M14.83 14.83l4.24 4.24"></path>
                  <path d="M2 12h6"></path>
                  <path d="M16 12h6"></path>
                  <path d="M4.93 19.07l4.24-4.24"></path>
                  <path d="M14.83 9.17l4.24-4.24"></path>
                </svg>
                <p class="text-sm mt-1">{{ weatherData.main.humidity }}%</p>
                <p class="text-xs text-blue-200">Humidity</p>
              </div>
              <div class="flex flex-col items-center">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-blue-200" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M9.59 4.59A2 2 0 1111 8H2m10.59 11.41A2 2 0 1014 16H2m15.73-8.27A2.5 2.5 0 1119.5 12H2"></path>
                </svg>
                <p class="text-sm mt-1">{{ weatherData.wind.speed }} m/s</p>
                <p class="text-xs text-blue-200">Wind</p>
              </div>
              <div class="flex flex-col items-center">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-blue-200" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M14 14.76V3.5a2.5 2.5 0 00-5 0v11.26a4.5 4.5 0 105 0z"></path>
                </svg>
                <p class="text-sm mt-1">{{ weatherData.main.pressure }} hPa</p>
                <p class="text-xs text-blue-200">Pressure</p>
              </div>
            </div>
          </div>
          
          <div class="mt-6 text-center text-sm text-gray-500">
            Last updated: {{ currentTime }}
          </div>
        </div>
      </div>
    </main>

    <!-- Footer -->
    <footer class="bg-white py-8 border-t border-gray-200">
      <div class="max-w-6xl mx-auto px-4">
        <div class="flex flex-col md:flex-row justify-between items-center">
          <!-- Logo and Copyright -->
          <div class="flex flex-col items-center md:items-start mb-6 md:mb-0">
            <div class="flex items-center space-x-2 mb-2">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-blue-500" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M17 18a5 5 0 0 0-10 0"></path>
                <line x1="12" y1="9" x2="12" y2="2"></line>
                <line x1="4.22" y1="10.22" x2="5.64" y2="11.64"></line>
                <line x1="1" y1="18" x2="3" y2="18"></line>
                <line x1="21" y1="18" x2="23" y2="18"></line>
                <line x1="18.36" y1="11.64" x2="19.78" y2="10.22"></line>
                <line x1="23" y1="22" x2="1" y2="22"></line>
                <polyline points="8 6 12 2 16 6"></polyline>
              </svg>
              <span class="font-semibold text-lg text-blue-700">WeatherVue</span>
            </div>
            <p class="text-sm text-gray-500">© 2025 WeatherVue. All rights reserved.</p>
          </div>
          
          <!-- Links -->
          <div class="grid grid-cols-2 md:grid-cols-3 gap-8">
            <div class="space-y-2">
              <h3 class="font-medium text-gray-800">Information</h3>
              <ul class="space-y-1">
                <li><a href="#" class="text-sm text-gray-600 hover:text-blue-600">About Us</a></li>
                <li><a href="#" class="text-sm text-gray-600 hover:text-blue-600">Contact</a></li>
                <li><a href="#" class="text-sm text-gray-600 hover:text-blue-600">Privacy Policy</a></li>
              </ul>
            </div>
            <div class="space-y-2">
              <h3 class="font-medium text-gray-800">Features</h3>
              <ul class="space-y-1">
                <li><a href="#" class="text-sm text-gray-600 hover:text-blue-600">Weather Maps</a></li>
                <li><a href="#" class="text-sm text-gray-600 hover:text-blue-600">5-Day Forecast</a></li>
                <li><a href="#" class="text-sm text-gray-600 hover:text-blue-600">Weather Alerts</a></li>
              </ul>
            </div>
            <div class="space-y-2 col-span-2 md:col-span-1">
              <h3 class="font-medium text-gray-800">Connect</h3>
              <div class="flex space-x-4">
                <!-- Social Icons -->
                <a href="#" class="text-gray-500 hover:text-blue-600">
                  <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 24 24">
                    <path d="M24 4.557c-.883.392-1.832.656-2.828.775 1.017-.609 1.798-1.574 2.165-2.724-.951.564-2.005.974-3.127 1.195-.897-.957-2.178-1.555-3.594-1.555-3.179 0-5.515 2.966-4.797 6.045-4.091-.205-7.719-2.165-10.148-5.144-1.29 2.213-.669 5.108 1.523 6.574-.806-.026-1.566-.247-2.229-.616-.054 2.281 1.581 4.415 3.949 4.89-.693.188-1.452.232-2.224.084.626 1.956 2.444 3.379 4.6 3.419-2.07 1.623-4.678 2.348-7.29 2.04 2.179 1.397 4.768 2.212 7.548 2.212 9.142 0 14.307-7.721 13.995-14.646.962-.695 1.797-1.562 2.457-2.549z"/>
                  </svg>
                </a>
                <a href="#" class="text-gray-500 hover:text-blue-600">
                  <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 24 24">
                    <path d="M19 0H5a5 5 0 00-5 5v14a5 5 0 005 5h14a5 5 0 005-5V5a5 5 0 00-5-5zm-3 7h-1.924C13.461 7 13 7.252 13 7.889V9h3l-.238 3H13v8h-3v-8H8V9h2V7.077C10 5.055 11.064 4 13.461 4H16v3z"/>
                  </svg>
                </a>
                <a href="#" class="text-gray-500 hover:text-blue-600">
                  <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 24 24">
                    <path d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zM12 0C8.741 0 8.333.014 7.053.072 2.695.272.273 2.69.073 7.052.014 8.333 0 8.741 0 12c0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98C8.333 23.986 8.741 24 12 24c3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98C15.668.014 15.259 0 12 0zm0 5.838a6.162 6.162 0 100 12.324 6.162 6.162 0 000-12.324zM12 16a4 4 0 110-8 4 4 0 010 8zm6.406-11.845a1.44 1.44 0 100 2.881 1.44 1.44 0 000-2.881z"/>
                  </svg>
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const city = ref('');
const weatherData = ref(null);
const loading = ref(false);
const error = ref(null);
const recentSearches = ref([]);
const currentTime = ref('');
const mobileMenuOpen = ref(false);

// Toggle mobile menu
const toggleMobileMenu = () => {
  mobileMenuOpen.value = !mobileMenuOpen.value;
};

// Weather condition computed properties
const isCloudyWeather = computed(() => {
  if (!weatherData.value) return false;
  const condition = weatherData.value.weather[0].main.toLowerCase();
  return condition.includes('cloud') || condition.includes('fog') || condition.includes('mist');
});

const isSunnyWeather = computed(() => {
  if (!weatherData.value) return false;
  const condition = weatherData.value.weather[0].main.toLowerCase();
  return condition.includes('clear') || condition.includes('sun');
});

const isRainyWeather = computed(() => {
  if (!weatherData.value) return false;
  const condition = weatherData.value.weather[0].main.toLowerCase();
  return condition.includes('rain') || condition.includes('drizzle') || condition.includes('storm');
});

// Update time
const updateTime = () => {
  const now = new Date();
  currentTime.value = now.toLocaleTimeString();
};

// Search with recent city
const searchRecent = (recentCity) => {
  city.value = recentCity;
  fetchWeather();
};

const fetchWeather = async () => {
  if (!city.value.trim()) {
    error.value = "Please enter a city name.";
    weatherData.value = null;
    return;
  }

  loading.value = true;
  error.value = null;

  try {
    const apiKey = import.meta.env.VITE_API_KEY;
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&units=metric&appid=${apiKey}`
    );

    if (!response.ok) throw new Error("City not found.");

    weatherData.value = await response.json();
    updateTime();
    
    // Add to recent searches if not already there
    if (!recentSearches.value.includes(city.value)) {
      recentSearches.value = [city.value, ...recentSearches.value].slice(0, 3);
    }
    
  } catch (err) {
    error.value = err.message;
    weatherData.value = null;
  } finally {
    loading.value = false;
  }
};
</script>