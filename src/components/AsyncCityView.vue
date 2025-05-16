<template>
  <div class="flex flex-col flex-1 items-center">
    <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
      <p>
        You are currently previewing this city, click the "+"
        icon in the top right to track this city.
      </p>
    </div>

    <div v-if="weatherData" class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{ weatherData.currentTime.toLocaleDateString("en-us", {
          weekday: "short",
          day: "2-digit",
          month: "long"
        }) }}
        {{ weatherData.currentTime.toLocaleTimeString("en-us", {
          timeStyle: "short"
        }) }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(weatherData.current.temperature) }}&deg;C
      </p>
      <p>
        Wind Speed: {{ weatherData.current.windspeed }} km/h
      </p>
       <p><img :src="`http://openweathermap.org/img/wn/${weatherCodeToOWMIcon[weatherData.current.weathercode]}@2x.png`" alt="Weather icon"></p>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <div v-if="weatherData" class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll custom-scroll-bar">
          <div v-for="hour in weatherData.hourlyData" :key="hour.time" class="flex flex-col gap-4 items-center">
            <p class="whitespace-nowrap text-md">
              {{ hour.time.toLocaleTimeString("en-us", { hour: "numeric" }) }}
            </p>
            <p>{{ hour.temperature }}&deg;C</p>
            <p><img :src="`http://openweathermap.org/img/wn/${weatherCodeToOWMIcon[hour.weatherCode]}@2x.png`" alt="Weather icon"></p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <div v-if="weatherData" class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div v-for="day in weatherData.dailyData" :key="day.date" class="flex items-center">
          <p class="flex-1">
            {{ day.date.toLocaleDateString("en-us", { weekday: "long" }) }}
          </p>
          <div class="flex gap-2 flex-1 justify-end">
            <p>H: {{ Math.round(day.max) }}</p>
            <p>L: {{ Math.round(day.min) }}</p>
            <p><img :src="`http://openweathermap.org/img/wn/${weatherCodeToOWMIcon[day.weatherCode]}@2x.png`" alt="Weather icon"></p>
          </div>
        </div>
      </div>
    </div>

    <div class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity">
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();
const weatherData = ref(null);

const weatherCodeToOWMIcon = {
        0: '01d',
        1: '02d',
        2: '03d',
        3: '04d',
        45: '50d',
        48: '50d',
        51: '09d',
        53: '09d',
        55: '09d',
        56: '09d',
        57: '09d',
        61: '10d',
        63: '10d',
        65: '10d',
        66: '10d',
        67: '10d',
        71: '13d',
        73: '13d',
        75: '13d',
        77: '13d',
        80: '09d',
        81: '09d',
        82: '09d',
        85: '13d',
        86: '13d',
        95: '11d',
        96: '11d',
        99: '11d',
    };

const getWeatherData = async () => {
  try {
    const { lat, lon } = route.query;

    const res = await axios.get(
      `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true&hourly=temperature_2m,weathercode&daily=temperature_2m_max,temperature_2m_min,weathercode&timezone=auto`
    );

    const data = res.data;

    const parsed = {
      current: data.current_weather,
      currentTime: new Date(data.current_weather.time),
      hourlyData: data.hourly.time.map((time, index) => ({
        time: new Date(time),
        temperature: data.hourly.temperature_2m[index],
        weatherCode: data.hourly.weathercode[index]
      })),
      dailyData: data.daily.time.map((time, index) => ({
        date: new Date(time),
        max: data.daily.temperature_2m_max[index],
        min: data.daily.temperature_2m_min[index],
        weatherCode: data.daily.weathercode[index]
      }))
    };

    return parsed;
  } catch (err) {
    console.error('Error fetching weather data:', err);
    return null;
  }
};

onMounted(async () => {
  weatherData.value = await getWeatherData();
});

const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem('savedCities'));
  const updatedCities = cities.filter(city => city.id !== route.query.id);

  localStorage.setItem('savedCities', JSON.stringify(updatedCities));
  router.push({ name: 'home' });
};
</script>
