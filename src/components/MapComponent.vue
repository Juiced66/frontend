<template>
    <MapboxMap 
        :class="{ displayNone: loading, center: !loading }" class="map" 
        :auto-resize="true" 
        height="400px" width="100%"
        :accessToken="process.env.MAPBOX_TOKEN"
        :center="[3.900841959551621, 43.60123651620898]" :zoom="11" :flyToOptions="{ maxDuration: 2000, speed: 1.2 }"
        @loaded="$emit('loaded')"
        @click="handleClick"
    >
        <MapMarkerComponent
            v-for="point in markers" :key="point._id" 
            @update-status="event => updateStatus(event, point._id)" 
            :lngLat="point.geoPoint" 
            :color="getColor(point.status)"
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
        };
    },
    methods: {
        handlePopupClose() {
            this.popup = null;
        },
        handleAddPoint(name) {
            this.$emit('sendPoint',{name : name, geoPoint: [this.popup.lng, this.popup.lat], status: 'created'})
            this.handlePopupClose(); 
        },
        getColor(status){
            switch (status){
                case 'created' :
                    return 'yellow'
                case 'visited':
                    return 'green'
                default : 
                    return 'yellow'
            }
        },
        async handleClick(e) {
            if (this.popup) {
                await this.handlePopupClose()
            }
            this.popup = { lng: e.lngLat.lng, lat: e.lngLat.lat, name: '' };
        },
        updateStatus(event, _id) {
            event.stopPropagation();
            for (const point of this.markers) {
                if (point._id === _id) {
                    this.$emit('updateStatus', _id)
                    break
                }
            }
        },
    }
};
</script>
