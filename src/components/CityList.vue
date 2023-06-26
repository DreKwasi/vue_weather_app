<script setup>
import axios from 'axios';
import CityCard from './CityCard.vue';
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const { VITE_WEATHERAPIKEY } = import.meta.env;
const savedCities = ref([])
const router = useRouter()

const getCities = async () => {
    if (localStorage.getItem('savedCities')) {
        savedCities.value = JSON.parse(localStorage.getItem('savedCities'));

        const requests = [];
        savedCities.value.forEach((city) => {
            requests.push(
                axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=${VITE_WEATHERAPIKEY}&units=imperial`)
            );
        });
        const weatherData = await Promise.all(requests);

        // Flicker Delay
        await new Promise((res) => setTimeout(res, 1000))

        weatherData.forEach((value, index) => {
            savedCities.value[index].weather = value.data
        })
    }
}

await getCities();


const getCityView = (city) => {
    router.push({
        name: 'cityView',
        params: { state: city.state, city: city.city },
        query: { id: city.id, lat: city.coords.lat, lng: city.coords.lng },
    })
}

</script>

<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="getCityView(city)" />
    </div>
    <p v-if="savedCities.length === 0">
        No Locations Added.
    </p>
</template>


