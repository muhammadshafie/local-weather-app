<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        You are currently previewing this city, click the '+' icon to start
        tracking this city.
      </p>
    </div>
    <!-- Weather Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">
        {{ route.params.name }}, {{ route.params.country }}
      </h1>
      <p>
        Local Time:
        {{
          new Date(weatherData.currentTime).toLocaleTimeString("en-us", {
            hour: "2-digit",
            minute: "2-digit",
          })
        }}
      </p>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString("en-us", {
            weekday: "long",
            day: "2-digit",
            month: "long",
          })
        }}
      </p>
      <p
        class="text-8xl mb-8 border border-solid rounded-lg p-4 bg-gradient-to-l from-weather-primary to-weather-secondary hover:from-weather-secondary hover:to-weather-primary"
      >
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>

      <p>Feels like {{ Math.round(weatherData.current.feels_like) }}&deg;</p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`"
      />
      <p class="text-md">
        <i class="fa-solid fa-arrow-up"></i> High:
        {{ Math.round(weatherData.daily[0].temp.max) }}&deg; |
        <i class="fa-solid fa-arrow-down"></i> Low:
        {{ Math.round(weatherData.daily[0].temp.min) }}&deg;
      </p>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Hourly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleTimeString("en-us", {
                  hour: "numeric",
                })
              }}
            </p>
            <img
              class="w-[50px] object-cover"
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
            />
            <p class="text-xl">{{ Math.round(hourData.temp) }}&deg;</p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Weekly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">8-Day Forecast</h2>
        <div
          v-for="day in weatherData.daily"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("en-us", {
                weekday: "short",
                day: "2-digit",
                month: "short",
              })
            }}
          </p>
          <img
            class="w-[50px] h-[50px] object-cover"
            :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
          />
          <p>
            {{ Math.round(day.temp.max) }}&deg; /
            {{ Math.round(day.temp.min) }}&deg;
          </p>
          <div class="flex gap-2 flex-1 justify-end">
            <p class="capitalize">{{ day.weather[0].description }}</p>
          </div>
        </div>
      </div>
    </div>

    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
    >
      <i class="fa-solid fa-trash-can"></i>
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
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.long}&exclude={part}&appid=274d5ae6ac771c32e4033efcbee977bd&units=metric`
    );

    // CALCULATE CURRENT DATE AND TIME
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // CALCULATE HOURLY WEATHER OFFSET
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
    });

    //Flicker Delay
    await new Promise((res) => setTimeout(res, 1000));

    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};
const weatherData = await getWeatherData();

const router = useRouter();
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"));
  const updatedCities = cities.filter((city) => city.id !== route.query.id);
  localStorage.setItem("savedCities", JSON.stringify(updatedCities));
  router.push({
    name: "home",
  });
};
// CHECK OUTPUT
// console.log(weatherData);
</script>
