<template>
    <mapbox-marker @click="$emit('updateStatus', $event)" :lngLat="lngLat" :color="color" />
</template>

<script>
import  {MapboxMarker} from 'vue-mapbox-ts'; 

export default {
    name: 'MapMarkerComponent',
    components: {
        MapboxMarker,
    },
    props: {
    name: String,
    color: {
      type: String,
      required: true,
    },
    lngLat: Array
  }
}


</script>

<style scoped>

</style>