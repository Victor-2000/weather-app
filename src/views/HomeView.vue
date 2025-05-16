<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input 
      type="text" 
      v-model="searchQuery"
      @input="getSearchResults"
      placeholder="Search for a city"
      class="py-2 px-1 w-full bg-transparent border-b 
      focus:border-weather-secondary focus:outline-none 
      focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResults">
        <p v-if="searchError"> 
          Sorry, something went wrong, please try again. 
        </p>
        <p v-if="!serverError && mapboxSearchResults.length === 0 "> 
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li v-for="searchResult in mapboxSearchResults" 
          :key="searchResult.id"
          class="py-2 cursor-pointer"
          @click="previewCity(searchResult)">
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
        <Suspense>
          <Transition name="citycardview">
            <CityList v-slot="{Component}">
              <component :is="Component"/>
            </CityList>
          </Transition>
          <template #fallback>
            <CityCardSkeleton />
          </template>
        </Suspense>
      </div>
  </main>
</template>

<script setup>
 import {ref} from 'vue';
 import axios from 'axios';
 import config from 'config';
 import {useRouter} from 'vue-router';
 import CityList from '../components/CityList.vue';
 import CityCardSkeleton from '../components/CityCardSkeleton.vue';
 
 const router = useRouter();

 //Loads a city preview depending on a search result
 const previewCity = (searchResult) => {
   console.log(searchResult);
   const [city, state] = searchResult.place_name.split(",");
   router.push({
     name: 'cityView',
     params: {
       state: state,
       city: city,
     },
     query: {
        lat: searchResult.geometry.coordinates[1],
        lon: searchResult.geometry.coordinates[0],
        preview: true,
     }
   });
 }

 const mapboxAPIKey = config.MAPBOX_API_KEY;
 const searchQuery = ref("");
 const queryTimeout = ref(null);
 const mapboxSearchResults = ref(null);

 const getSearchResults = () => {
    clearTimeout(queryTimeout.value);
    queryTimeout.value = setTimeout(async () => {
      if(searchQuery.value !== ""){
        try {
          const result = await axios.get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
          );
          mapboxSearchResults.value = result.data.features;
        } catch {
          searchError.value = true;
        }
      }else{
        mapboxSearchResults.value = null;
      }
    }, 300);
 }
</script>

<style>
  .citycardview-enter-active {
    transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02) 0.15s;
  }

  .citycardview-leave-active {
    transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
  }

  .citycardview-enter-from {
    opacity: 0;
    transition: scale(0.8);
  }

  .citycardview-leave-to {
    opacity: 1;
    transition: scale(0.8);
  }
</style>
