<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

const { MAPBOXAPIKEY} = import.meta.env


const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null)

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${MAPBOXAPIKEY}&types=place`
        );

        mapboxSearchResults.value = result.data.features;
      }
      catch {
        searchError.value = true
      }
      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};

const router = useRouter()
const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(",")
  console.log(searchResult.geometry)
  router.push({
    name: "cityView",
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResult.geometry.coordinates[0],
      lng: searchResult.geometry.coordinates[1],
      preview: true
    },
  })
}


</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input v-model="searchQuery" @input="getSearchResults" type="text" placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004e71]" />
      <ul v-if="mapboxSearchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
        <p v-if="searchError">Sorry, Something Went Wrong Please Try Again</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">No results match your query, try a different term.</p>

        <template v-else>
          <li v-for="searchResult in mapboxSearchResults" :key="searchResult.id" class="py-2 cursor-pointer"
            @click="previewCity(searchResult)">
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>
