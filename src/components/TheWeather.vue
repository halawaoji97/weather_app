<template>
  <div>
    <div class="container" v-if="weatherData && !loading">
      <div class="weather-side">
        <div class="weather-gradient">
          <div class="date-container">
            <div class="location">
              <h1>{{ cityName }}, {{ country }}</h1>
            </div>
            <p>
              <span class="date-dayname">{{ dayName }}, </span>{{ currentDate }}
            </p>
          </div>
          <div class="weather-container">
            <Icon
              v-if="weatherData.weather"
              :icon="getWeatherIconName(weatherData.weather[0].id)"
              class="weather-icon"
              :class="getWeatherIconClass()"
            />
            <p class="weather-temp">{{ temperature }}°C</p>
            <p class="weather-desc" v-if="weatherData.weather">
              {{ weatherData.weather[0].description }}
            </p>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="loading-spinner">
      <div class="spinner"></div>
      <div>Loading</div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import { Icon } from '@iconify/vue'

interface WeatherData {
  name: string
  sys: {
    country: string
  }
  main: {
    temp: number
  }
  dt: number
  weather: {
    id: number
    description: string
  }[]
}

const apiKey = import.meta.env.VITE_APP_API_KEY
axios.defaults.baseURL = import.meta.env.VITE_APP_API_ADDRESS

const weatherData = ref<WeatherData | null>(null)
const cityName = ref<string>('')
const country = ref<string>('')
const temperature = ref<string>('')
const dayName = ref<string>('')
const currentDate = ref<string>('')

const getWeatherIconName = (conditionCode: number): string => {
  if (conditionCode >= 200 && conditionCode < 300) {
    return 'wi:day-lightning'
  } else if (conditionCode >= 300 && conditionCode < 600) {
    return 'wi:day-rain'
  } else if (conditionCode >= 600 && conditionCode < 700) {
    return 'wi:day-snow'
  } else if (conditionCode >= 700 && conditionCode < 800) {
    return 'wi:day-fog'
  } else if (conditionCode === 800) {
    return 'wi:day-sunny'
  } else if (conditionCode >= 801 && conditionCode < 900) {
    return 'wi:day-cloudy'
  } else {
    return 'wi:na'
  }
}

const getWeatherIconClass = () => {
  if (weatherData.value && weatherData.value.weather) {
    const conditionCode = weatherData.value.weather[0].id
    if (conditionCode >= 200 && conditionCode < 300) {
      return 'icon-lightning'
    } else if (conditionCode >= 300 && conditionCode < 600) {
      return 'icon-rain'
    } else if (conditionCode >= 600 && conditionCode < 700) {
      return 'icon-snow'
    } else if (conditionCode >= 700 && conditionCode < 800) {
      return 'icon-fog'
    } else if (conditionCode === 800) {
      return 'icon-sunny'
    } else if (conditionCode >= 801 && conditionCode < 900) {
      return 'icon-cloudy'
    } else {
      return 'icon-unknown'
    }
  }
}

const loading = ref<boolean>(false)

const fetchWeatherData = async () => {
  loading.value = true
  try {
    const position = await getUserLocation()
    const { latitude, longitude } = position.coords

    const response = await axios.get<WeatherData>(
      `/weather?lat=${latitude}&lon=${longitude}&appid=${apiKey}`
    )

    cityName.value = response.data.name
    country.value = response.data.sys.country
    temperature.value = (response.data.main.temp - 273.15).toFixed(1)
    dayName.value = new Date(response.data.dt * 1000).toLocaleString('en-US', {
      weekday: 'long',
    })
    currentDate.value = new Date(response.data.dt * 1000).toLocaleString('en-US', {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
    })

    weatherData.value = response.data
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}

const getUserLocation = (): Promise<GeolocationPosition> => {
  return new Promise((resolve, reject) => {
    if ('geolocation' in navigator) {
      navigator.geolocation.getCurrentPosition(resolve, reject)
    } else {
      reject(new Error('Geolocation is not available.'))
    }
  })
}

onMounted(() => {
  fetchWeatherData()
})
</script>
