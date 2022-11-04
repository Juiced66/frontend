<template>
  <div class="container">
    <h1>Kuzzle Interactive POI (Point of Interests) Map</h1>
    <div class="instructions">
      <h2>Instructions : </h2>
      <ul>
        <li>Click on map to add a Point of Interest you wanna visit (yellow)</li>
        <li>Click Point of Interest you have visited to change his color (green)</li>
        <li>Click click again to delete it</li>
        <!-- True to say that at this point, everybody freely can interact with everyone's point of interest !.. -->
      </ul>
    </div>
    <div class="loader" v-if="loading">
      <PulseLoader />
    </div>
    <MapComponent 
      :markers="points" 
      @send-point="point => sendPoint(point)"
      @update-status="_id => updateStatus(_id)"
      @loaded="loading = false" />
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
        name: document._source.name,
        status: document._source.status
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
          status: point.status
        }
      );
      this.points = [
        {
          geoPoint: [point.geoPoint[0], point.geoPoint[1]],
          name: point.name,
          _id: document._id,
          status : 'created'
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
        for (const [i, point] of this.points.entries()) {
          if (point._id === _id) {
            this.points.splice(i, 1);
          }
        }
      } catch (error) {
        console.error(error.message);
      }
    },
    async updatePoint(p) {
      try {
        const document = await kuzzle.document.update('map-interactions', 'points-of-interest', p._id, {
          name: p.name,
          geo_point: p.geo_point,
          status: 'visited'
        });
        for (const [i, point] of this.points.entries()) {
          if (point._id === p._id) {
            this.points[i] = {
              ...point,
              status: 'visited'
            };
          }
        }
      } catch (error) {
        console.error(error.message);
      }
    },
    async updateStatus(_id){
      for (const [i, point] of this.points.entries()) {
          if (point._id === _id) {
            switch(point.status){
              case 'created': 
                return this.updatePoint(point);
              case 'visited': 
                return this.deletePoint(_id);
              default : return;
            }
          }
        }
    },
    async subscribePoints() {
      this.subId = await kuzzle.realtime.subscribe(
        "map-interactions",
        "points-of-interest",
        {},
        notification => {
          if (notification.type !== "document") return;
          if (!["create", "delete", "update"].includes(notification.action)) return;
          if (notification.action === "create") {
            this.points = [
              this.getPoint(notification.result),
              ...this.points
            ];
          } else if(notification.action === "update"){
            for (const [i, point] of this.points.entries()) {
              if (point._id === this.getPoint(notification.result)._id) {
                this.points[i].status = 'visited';
              }
            }
          }
          else {
            for (const [i, point] of this.points.entries()) {
              if (point._id === this.getPoint(notification.result)._id) {
                this.points.splice(i, 1);
              }
            }
          }
        }
      );
    },
  },
  mounted() {
    this.valid();
  }
};
</script>

<style scoped>
.container {
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  align-content: center;
  height: 100%;
  width: 90%;
  margin: 0 auto
}
.instructions {
  align-self: flex-start;
}
</style>
<style>
  body {
    background:linear-gradient( to bottom, #bcc6cc, #eee, #bcc6cc);
    font-family: sans-serif;
    min-height: 100vh;
    min-width: 100vw;
    overflow: hidden;
  }
</style>