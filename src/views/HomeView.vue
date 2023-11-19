<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        placeholder="Search for a city or state..."
        v-model="searchQuery"
        @input="getSearchResults"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <Transition name="results">
        <ul
          class="position absolute bg-weather-secondary text--white w-full shadow-md py-2 px-1 top-[66px]"
          v-if="mapboxSearchResults"
        >
          <p v-if="searchError">Something went wrong... Please try again.</p>
          <p v-if="!serverError && mapboxSearchResults.length === 0">
            No city found... Please enter another name.
          </p>
          <template v-else>
            <li
              v-for="searchResult in mapboxSearchResults"
              :key="searchResult.id"
              class="py-2 cursor-pointer"
              @click="previewCity(searchResult)"
            >
              {{ searchResult.place_name }}
            </li>
          </template>
        </ul>
      </Transition>
    </div>
    <div class="flex flex-col gap-4">
      <suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

import CityList from "../components/CityList.vue";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";

const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxAPIKEY =
  "pk.eyJ1IjoiYmliaW5oaW1zZWxmIiwiYSI6ImNsb2xjdWs3MDJjeGUya212MWFkdHEzeDYifQ._7PJm2-fgovTxO0S4wW6Ag";

const mapboxSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKEY}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }
      return; //if the above call is true, so it returns
    }

    mapboxSearchResults.value = null; //if the above call is not true
  }, 300);
};




const router = useRouter();

const previewCity = (searchResult) =>{
  console.log(searchResult);
  const [city, state, country] = searchResult.place_name.split(","); //[city, state, country]  this should be as the same order as search results (console.log data)

  router.push({
    name: "cityView",
    params: {
      state: state.replaceAll(" ", ""),
      city: city,
      country: country.replaceAll(" ", "")
    },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true
    }
  });
};





</script>

<style>
.results-enter-active {
  transition: all 0.3s cubic-bezier(0.02, 0.02, 0.19, 1.02);
}

.results-leave-active {
  transition: all 0.2s cubic-bezier(0.02, 0.02, 0.19, 1.02) 0s;
}

.results-enter-from {
  opacity: 0;
  transform: scale(0.8);
}

.results-leave-to {
  opacity: 0;
  transform: scale(0.8);
}
</style>
