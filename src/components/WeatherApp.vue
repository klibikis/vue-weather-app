<!-- eslint-disable vue/no-parsing-error -->
<template>
  <div class="app__wrapper">
    <div v-if="isLoading" class="weather-app__wrapper">
    Loading....
    </div>
    <div v-else class="weather-app__wrapper">
      <div class="toggler__wrapper">
        <p class="temperature-units--text">°C | °K</p>
        <label class="switch">
          <input type="checkbox" v-model="isBoxChecked">
          <span class="slider round"></span>
        </label>
      </div>
      <div class="title__wrapper">
        <h1 class="location--title">{{ weatherData.city }}, {{ weatherData.country }}</h1>
      </div>
      <div class="additional-info__wrapper">
        <p>💨 Time zone: {{ weatherData.timeZone }}</p>
        <p>💨 Wind direction: {{ weatherData.windDirection }}</p>
      </div>
      <div class="main-info__wrapper">
        <div class="date-time__wrapper">
          <p>🌡️ &nbsp {{ temperature }}</p>
          <p>📅 &nbsp {{ date }}</p>
          <p>🕑 &nbsp {{ time }}</p>
          <p>🌅 &nbsp {{ weatherData.sunriseTime }}</p>
          <p>🌄 &nbsp {{ weatherData.sunsetTime }}</p>
        </div>
        <div class="weather-icon__wrapper">
          <img :src="weatherData.icon" :alt="weatherData.description" />
          <p>{{ weatherData.description }}</p>
        </div>
      </div>
      <div>
        <form class="search-form" @submit="(e) => {
          e.preventDefault()
          handleFormSubmit()
        }">
          <input type="text" placeholder="City" class="search-input" v-model="searchValue" required>
          <button type="submit" class="search-button">Get weather</button>
        </form>
      </div>
    </div>
  </div>
  
</template>

<script lang="ts">
import { computed, onMounted, ref } from 'vue'
import axios from 'axios'
const API_KEY = '34b03b556fee4c3da5d7b732371f1b8b'

type WeatherData = {
  city: string;
  country: string;
  description: string;
  temperature: number;
  icon: string;
  currentTime: string;
  sunriseTime: string;
  sunsetTime: string;
  windDirection: string;
  timeZone: string;
}

export default {
  setup() {
    const weatherData = ref<WeatherData>({
      city: '',
      country: '',
      description: '',
      temperature: 0,
      icon: '',
      currentTime: '',
      sunriseTime: '',
      sunsetTime: '',
      windDirection: '',
      timeZone: ''
    })
    const city = ref('Riga')
    const searchValue = ref('')
    const isLoading = ref(false)
    const isBoxChecked = ref(false)

    const date = computed(() => {
      return weatherData.value.currentTime.split(' ')[0]
    })
    const time = computed(() => {
      return weatherData.value.currentTime.split(' ')[1]
    })
    const temperature = computed(() => {
      if(isBoxChecked.value){
        const temperatureInFarenhaits = weatherData.value.temperature*1.8+32
        return `${temperatureInFarenhaits} °K`
      }else{
        return `${weatherData.value.temperature} °C`
      }
    })

    const fetchWeatherData = () => {

      const url = `https://api.weatherbit.io/v2.0/current?city=${city.value}&key=${API_KEY}&include=minutely`
      isLoading.value = !isLoading.value
      
      axios.get(url).then(({ data }) => {

        weatherData.value = {
          city: data.data[0].city_name,
          temperature: data.data[0].app_temp,
          country: data.data[0].country_code,
          description: data.data[0].weather.description,
          icon: `https://www.weatherbit.io/static/img/icons/${data.data[0].weather.icon}.png`,
          currentTime: data.data[0].ob_time,
          sunriseTime: data.data[0].sunrise,
          sunsetTime: data.data[0].sunset,
          windDirection: data.data[0].wind_cdir_full,
          timeZone: data.data[0].timezone
        }

        isLoading.value = !isLoading.value

      })
    }

    const handleFormSubmit = () => {
      city.value = searchValue.value
      fetchWeatherData()
      searchValue.value=''
    }

    onMounted(() => {
      fetchWeatherData()
    })
    return {
      weatherData,
      temperature,
      isLoading,
      searchValue,
      handleFormSubmit,
      date,
      time,
      isBoxChecked
    }
  }
}
</script>

<style scoped>
.app__wrapper{
  width: 100%;
  min-height: 100vh;
  margin: 0 auto;
  display: flex;
  justify-content: center;
  align-items: center;
}
.weather-app__wrapper{
  width: 25%;
  min-height: 400px;
  background-color: rgba(255, 255, 255, 0.398);
  padding: 50px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  position: relative;
}
.search-form{
  padding: 10px 0;
  height: 60px;
  display: flex;
  justify-content: center;
  gap: 5px;
}
.search-input{
  all: unset;
  height: 100%;
  background-color: rgba(255, 255, 255, 0.519);
  padding: 0 15px;
  border-radius: 5px;
}
.search-button{
  all: unset;
  height: 100%;
  background-color: #feffddbf;
  padding: 0 15px;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s ease-in;
  box-shadow: 0 0 20px #f3f4d6e0;
}
.search-button:hover{
  background-color: #f0f2ad;
  box-shadow: 0 0 70px #f0f2ad;
}
.title__wrapper{
  font-weight: 600;
  font-size: 20px;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px;
  background-color: #fdffc22b;
  box-shadow: 0 0 70px #fdffc285;
  border-radius: 60px;
}
.date-time__wrapper{
  padding: 15px 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.main-info__wrapper{
  width: 100%;
  display: flex;
  justify-content: space-around;
}
.additional-info__wrapper{
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 5px;
  padding: 10px 0 10px 20px;
}
.toggler__wrapper{
  position: absolute;
  top: 10px;
  right: 10px;
  display: flex;
  align-items: center;
  gap: 10px;
}
.switch {
  position: relative;
  display: inline-block;
  width: 38px;
  height: 22px;
}

/* Hide default HTML checkbox */
.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

/* The slider */
.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  -webkit-transition: .4s;
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 2px;
  bottom: 2px;
  background-color: white;
  -webkit-transition: .4s;
  transition: .4s;
}

input:checked + .slider {
  background-color: #96f3f0;
}

input:focus + .slider {
  box-shadow: 0 0 1px #f3f4d6;
}

input:checked + .slider:before {
  -webkit-transform: translateX(16px);
  -ms-transform: translateX(16px);
  transform: translateX(16px);
}

/* Rounded sliders */
.slider.round {
  border-radius: 44px;
}

.slider.round:before {
  border-radius: 50%;
}
.temperature-units--text{
  font-size: 14px;
}
.weather-icon__wrapper{
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
