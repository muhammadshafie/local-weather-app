<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search here for any city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="openweatherSearchResults"
      >
        <p v-if="searchError">Sorry, something went wrong, please try again.</p>
        <p v-if="!searchError && openweatherSearchResults.length === 0">
          No results match your query, please try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in openweatherSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.name }},
            <!-- if no data for state inside the array then no extra comma provided -->
            <span v-if="searchResult.state">{{ searchResult.state }}, </span>
            {{ searchResult.country }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";

const router = useRouter();
const previewCity = (searchResult) => {
  // CHECK OUTPUT
  // console.log(searchResult);
  const name = searchResult.name;
  const country = searchResult.country;
  // CHECK OUTPUT
  // console.log(name);
  // console.log(country);

  router.push({
    name: "cityView",
    params: {
      name: name,
      country: country,
    },
    query: {
      lat: searchResult.lat,
      long: searchResult.lon,
      preview: true,
    },
  });
};

const OpenWeatherAPIKey = "274d5ae6ac771c32e4033efcbee977bd";
const searchQuery = ref("");
const queryTimeout = ref(null);
const openweatherSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `http://api.openweathermap.org/geo/1.0/direct?q=${searchQuery.value}&limit=5&appid=${OpenWeatherAPIKey}`
        );
        openweatherSearchResults.value = result.data;
        // CHECK OUTPUT
        // console.log(openweatherSearchResults.value);
      } catch {
        searchError.value = true;
      }
      return;
    }
    openweatherSearchResults.value = null;
  }, 300);
};
</script>
