<template>
  <div class="app-header">
    <h1>Kuzzle Interactive POI (Point of Interests) Map</h1>
  </div>
  <div class="instructions">
    <h2>Instructions : </h2>
    <ul>
      <li>Click on map to add a Point of Interest</li>
      <li>Click on Point of interest to delete it</li>
    </ul>
  </div>
  <div class="container">
    <div class="loader" v-if="loading">
      <PulseLoader />
    </div>
    <MapComponent @loaded="loading = false" />
  </div>

</template>
 
<script>
import MapComponent from './components/MapComponent.vue';
import PulseLoader from 'vue-spinner/src/PulseLoader.vue';
import kuzzle from "./services/KuzzleService.ts";

export default {
  name: 'App',
  components: {
    MapComponent,
    PulseLoader
  },
  data() {
    return {
      points: [],
      loading: true,
      modalShowAddPoint: false,
      popup: null,
      subId: "", // Id of the realtime subscription
    };
  },
  methods: {
    /* Kuzzle methods */
    async valid() {

      await kuzzle.connect();

      if (! await kuzzle.index.exists("map-interactions")) {
        await kuzzle.index.create("map-interactions");
        await kuzzle.collection.create("map-interactions", "points-of-interest");
      }
      // await this.fetchMessages();
      // await this.subscribeMessages();
      this.validate = true;
    }
  }
};
</script>

<style scoped>
.container,
.app-header {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  align-content: center;
  height: 100%;
  width: 90%;
  margin: 0 auto;
}
</style>
