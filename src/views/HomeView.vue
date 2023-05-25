<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        v-model="searchQuery"
        @input="getSearchResults"
        type="text"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        v-if="mapBoxSearchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="searchError">Sorry... Something went wrong...</p>
        <p v-if="!searchError && mapBoxSearchResults.length == 0">
          No item matched your search...
        </p>
        <template v-else
          ><li
            v-for="searchResult in mapBoxSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li></template
        >
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
  console.log(searchResult);
  const [city, state] = searchResult.place_name.split(",");

  router.push({
    name: "cityView",
    params: { city: city, state: state.replaceAll(" ", "") },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  });
};

const searchQuery = ref("");
const queryTimeout = ref(null);
const mapBoxSearchResults = ref(null);
const searchError = ref(false);

const mapBoxAPIKey =
  "pk.eyJ1IjoibWFzb29tZSIsImEiOiJjanNjNTFvdmswNWtjM3lueGpka3R3dW00In0.n5sgz9b_gAKcTerosaOUPg";

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value != "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapBoxAPIKey}&type=place`
        );
        mapBoxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }
      return;
    }
    mapBoxSearchResults.value = null;
  }, 300);
};
</script>
