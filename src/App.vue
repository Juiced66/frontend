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
    <MapComponent :markers="points" @delete-point="_id => deletePoint(_id)" @send-point="point => sendPoint(point)" @loaded="loading = false" />
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
      points : [],
      loading: true,
      popup: null,
      subId: "",
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
      await this.fetchPoints();
      await this.subscribePoints();
    },
    getPoint(document) {
      const point = {
        _id: document._id,
        geoPoint: document._source.geo_point,
        name: document._source.name
      };
      return point;
    },
    async fetchPoints() {
      const results = await kuzzle.document.search(
        "map-interactions",
        "points-of-interest",
        { sort: ["_kuzzle_info.createdAt"] },
        { size: 100 }
      );
      // Add each message to our array
      results.hits.map(hit => {
        this.points = [this.getPoint(hit), ...this.points];
      });
    },
    async sendPoint(point) {
      const document = await kuzzle.document.create(
        "map-interactions",
        "points-of-interest",
        {
          geo_point: [point.geoPoint[0], point.geoPoint[1]],
          name: point.name,
        }
      );
      this.points= [
      {
                geoPoint: [point.geoPoint[0], point.geoPoint[1]],
                name: point.name,
                _id: document.result._id
      },
      ...this.points
    ];
    },
    async deletePoint(_id) {
      try {
        const id = await kuzzle.document.delete('map-interactions', 'points-of-interest', _id);
        if (id === _id) {
          console.log('Success');
        }
        for (const [i, point] of this.points.entries()){
          if(point._id === _id){
            this.points.splice(i, 1)
          }
        }
      } catch (error) {
        console.error(error.message);
      }
    },
    async subscribePoints() {
      this.subId = await kuzzle.realtime.subscribe(
        "map-interactions",
        "points-of-interest",
        {},
        notification => {
          if (notification.type !== "document") return;
          if (notification.action !== "create") return;
          this.points = [
            this.getPoint(notification.result),
            ...this.points
          ];
        }
      );
    },
  },
  mounted() {
    this.valid()
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
