<template>
  <div v-for="city in savedCities" :key="city.id">
  <CityCard :city="city" @click="goToCityView(city)"/>
  </div>

  <p v-if="savedCities.length === 0">No locations added. Search in the above field to start tracking a location.</p>
</template>

<script setup>
import {ref} from 'vue';
import axios from 'axios';

import CityCard from './CityCard.vue';
import { useRouter } from 'vue-router';


const savedCities = ref([]);

const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));// for changing the data ito JS object
    //retrieving cities from local storage

    const requests = [];
    savedCities.value.forEach((city) => {
      requests.push(
        axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=63cd3def2667c4a6a4dfb49b8f40d8ed&units=imperial`
        )
      );
    });
    //getting current weather for particular city selected


    const weatherData = await Promise.all(requests);

    //flicker delay/skeleton
    await new Promise((res) => setTimeout(res, 1000));

    weatherData.forEach((value, index) => {  //first index =0 so 1st value of array
        savedCities.value[index].weather = value.data;
    }); 
    //matching up the data in api and the cities in savedCities array. since the cities in array and data from api is in same order we can use index


  }
};

await getCities(); //running the function


//for redirecting to cityview page(CityView Component) that contains whole data
const router = useRouter();
const goToCityView = (city) => {
    router.push({
        name: "cityView",
        params: {
            country: city.country,
            state: city.state,
            city: city.city 
        },
        query: {
            id: city.id,
            lat: city.coords.lat,
            lng: city.coords.lng
        }
    })
}

</script>
