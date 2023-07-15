<template>
  <div class="flex flex-col flex-1 items-center">

    <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
      <p>You are currently previewing this city, click the "+" icon to start tracking this city.</p>
    </div>

    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-3xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-10">
        {{
          new Date(weatherData.currentTime).toLocaleDateString(
            "en-us",
            {
              weekday: "short",
              day: "2-digit",
              month: "long",
            }
          )
        }}
        {{
          new Date(weatherData.currentTime).toLocaleDateString(
            "en-us",
            {
              hour: "2-digit",
              minute: "2-digit",
            }
          )
        }}
      </p>
      <p class="text-6xl mb-6">
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>
      <p>
        Feels like
        {{ Math.round(weatherData.current.feels_like) }} &deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`" alt=""
        class="w-[150px] h-auto">
    </div>

    <hr class="border-white border-opacity-10 w-full border">

    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div v-for="hourData in weatherData.hourly" :key="hourData.dt" class="flex flex-col gap-4 items-center">
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleTimeString("en-us", {
                  hour: "numeric",
                })
              }}
            </p>
            <img :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`" alt=""
              class="h-[50px] w-auto object-cover">
            <p class="text-xl">
              {{ Math.round(hourData.temp) }} &deg;
            </p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 w-full border">

    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div v-for="day in weatherData.daily" :key="day.dt" class="flex items-center">
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("en-us", {
                weekday: "long",
              })
            }}
          </p>
          <img :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`" alt=""
            class="h-[50px] w-[50px] object-cover" />
          <div class="flex gap-2 flex-1 justify-end">
            <p>D:{{ Math.round(day.temp.max) }}</p>
            <p>N:{{ Math.round(day.temp.min) }}</p>
          </div>
        </div>
      </div>
    </div>

    <div class="flex items-center gap-2 py-12 text-white cursor-pointer suration-150 hover:text-red-500"
      @click="removeCity">
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(`https://api.openweathermap.org/data/3.0/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=88852d1e155b70cec7892b573b1f1eb3&units=metric`)

    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime =
        utc + 1000 * weatherData.data.timezone_offset;
    });

    return weatherData.data;
  } catch (err) {
    console.log(err)
  }
};
const weatherData = await getWeatherData();

//Flicker Delay
await new Promise((res) => setTimeout(res, 500));

const router = useRouter();
const removeCity = () => {
  const cities = JSON.parse(
    localStorage.getItem("savedCities")
  );
  const updatedCities = cities.filter(
    (city) => city.id !== route.query.id
  );
  localStorage.setItem(
    "savedCities", JSON.stringify(updatedCities)
  );
  router.push({
    name: "home",
  })
};
</script>