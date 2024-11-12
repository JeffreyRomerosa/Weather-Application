<template>
  <v-app>
    <v-main class="bg-blue-lighten-4">
      <v-container fluid class="fill-height">
        <v-row justify="center" align="center">
          <v-col cols="12" sm="8" md="6" lg="4">
            <v-card class="mx-auto pa-4" elevation="8" rounded="lg">
              <v-card-title class="text-h4 font-weight-bold text-center mb-6">
                Weather App
              </v-card-title>
              <v-form @submit.prevent="fetchWeather">
                <v-text-field
                  v-model="city"
                  label="Enter city name"
                  variant="outlined"
                  append-inner-icon="mdi-magnify"
                  @click:append-inner="fetchWeather"
                  @keyup.enter="fetchWeather"
                ></v-text-field>
              </v-form>
              <v-alert
                v-if="error"
                type="error"
                variant="tonal"
                class="mb-4"
              >
                {{ error }}
              </v-alert>
              <v-slide-y-transition group>
                <v-card
                  v-for="weather in weatherList"
                  :key="weather.id"
                  :class="getBackgroundColor(weather.main.temp)"
                  class="mb-4"
                >
                  <v-card-text>
                    <v-row no-gutters>
                      <v-col>
                        <div class="text-h5 font-weight-bold">{{ weather.name }}</div>
                        <div class="text-subtitle-1">{{ weather.weather[0].description }}</div>
                      </v-col>
                      <v-col cols="auto">
                        <div class="text-h3">{{ getWeatherIcon(weather.weather[0].main) }}</div>
                      </v-col>
                    </v-row>
                    <v-row class="mt-4">
                      <v-col cols="6" class="d-flex align-center">
                        <v-icon color="red" class="mr-2">mdi-thermometer</v-icon>
                        <span>{{ Math.round(weather.main.temp) }}°C</span>
                      </v-col>
                      <v-col cols="6" class="d-flex align-center">
                        <v-icon color="blue" class="mr-2">mdi-water-percent</v-icon>
                        <span>{{ weather.main.humidity }}%</span>
                      </v-col>
                      <v-col cols="6" class="d-flex align-center">
                        <v-icon color="grey" class="mr-2">mdi-weather-windy</v-icon>
                        <span>{{ weather.wind.speed }} m/s</span>
                      </v-col>
                    </v-row>
                  </v-card-text>
                  <v-card-actions>
                    <v-btn
                      color="error"
                      variant="tonal"
                      @click="deleteWeather(weather.id)"
                    >
                      Delete
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-slide-y-transition>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue'
import { useTheme } from 'vuetify'

const theme = useTheme()

const city = ref('')
const weatherList = ref([])
const error = ref('')

const fetchWeather = async () => {
  if (city.value.trim() === '') {
    error.value = 'Please enter a city name.'
    return
  }
  try {
    const apiKey = '4563ae76186065ac885c54ea0f81ea99'
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&units=metric&appid=${apiKey}`
    )
    const data = await response.json()
    if (data.cod === 200) {
      weatherList.value.unshift({ ...data, id: Date.now() })
      error.value = ''
      city.value = ''
    } else {
      error.value = 'City not found.'
    }
  } catch (err) {
    error.value = 'Unable to fetch weather data.'
  }
}

const deleteWeather = (id) => {
  weatherList.value = weatherList.value.filter(weather => weather.id !== id)
}

const getBackgroundColor = (temp) => {
  if (temp > 30) {
    return theme.global.current.value.dark ? 'bg-red-darken-4' : 'bg-red-lighten-4'
  } else if (temp < 15) {
    return theme.global.current.value.dark ? 'bg-blue-darken-4' : 'bg-blue-lighten-4'
  } else {
    return theme.global.current.value.dark ? 'bg-grey-darken-3' : 'bg-grey-lighten-3'
  }
}

const getWeatherIcon = (main) => {
  switch (main.toLowerCase()) {
    case 'clear':
      return '☀️'
    case 'clouds':
      return '☁️'
    case 'rain':
      return '🌧️'
    case 'snow':
      return '❄️'
    default:
      return '🌤️'
  }
}
</script>

<style scoped>
.clouds {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  overflow: hidden;
}

.cloud {
  position: absolute;
  width: 150px;
  height: 90px;
  background-size: contain;
  background-repeat: no-repeat;
  opacity: 0.5;
  animation: float 20s infinite ease-in-out;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-20px);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
</style>
