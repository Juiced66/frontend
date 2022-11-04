<template>
    <MapboxMap 
        :class="{ displayNone: loading, center: !loading }" class="map" 
        :auto-resize="true" 
        height="400px" width="100%"
        accessToken="pk.eyJ1IjoianVpY2VkIiwiYSI6ImNsYTBwdGdkMzByZDgzeWx0cHBvY3JocWYifQ.P2VZk8UGbSdXYyMEb5SiMg"
        :center="[3.900841959551621, 43.60123651620898]" :zoom="11" :flyToOptions="{ maxDuration: 2000, speed: 1.2 }"
        @loaded="$emit('loaded')"
        @click="handleClick"
    >
        <MapMarkerComponent
            v-for="point in markers" :key="point._id" 
            @delete="event => deleteMarker(event, point._id)" 
            :lngLat="point.geoPoint" 
            :color="markerColor"
        />
        <MapModal 
            v-if="popup"
            :lngLat="[popup.lng, popup.lat]"
            @close="handlePopupClose" 
            @add-point="name => handleAddPoint(name)"
        />
    </MapboxMap>
</template>

<script>
import {MapboxMap} from 'vue-mapbox-ts';
import MapMarkerComponent from './MapMarkerComponent.vue';
import MapModal from './MapModal.vue';

export default {
    name: 'App',
    components: {
        MapboxMap,
        MapMarkerComponent,
        MapModal
    },
    props : {
        markers : Array,
        default : [],
    },
    data() {
        return {
            modalShowAddPoint: false,
            loading: true,
            popup: null,
            // TODO : handle multiples colors
            //availableColors : [],
            markerColor : 'red',
        };
    },
    methods: {
        handlePopupClose() {
            this.popup = null;
        },
        handleAddPoint(name) {
            this.$emit('sendPoint',{name : name, geoPoint: [this.popup.lng, this.popup.lat]})
            this.handlePopupClose(); 
        },
        async handleClick(e) {
            if (this.popup) {
                await this.handlePopupClose()
            }
            this.popup = { lng: e.lngLat.lng, lat: e.lngLat.lat, name: '' };
        },
        deleteMarker(event, _id) {
            event.stopPropagation();
            for (const point of this.markers) {
                if (point._id === _id) {
                    this.$emit('deletePoint', point._id)
                    break
                }
            }
        },
    }
};
</script>
