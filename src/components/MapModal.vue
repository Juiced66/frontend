<template>
    <mapbox-popup className="modal" :closeButton="true" @close="$emit('close')" :lngLat="lngLat">
        <div class="popup-title">Ajouter un point ?</div>
        <input 
            type="text"
            v-model="input" 
            placeholder="Entrez un nom svp"
        >
        <div 
            class="confirm" :class="{ disabled: !canValidate }" 
            :title="canValidate ? 'Valider' : 'Min: 3'"
            @click="canValidate && $emit('addPoint', input);input='';"
        >
            Confirm
        </div>
    </mapbox-popup>
</template>

<script>
import { MapboxPopup } from 'vue-mapbox-ts';

export default {
    name: 'App',
    components: {
        MapboxPopup,
    },
    data() {
        return {
            input: ''
        };
    },
    computed: {
        canValidate() {
        return this.input.length >= 3;
        }
    },
    props : {
        lngLat : {type : Array, required : true}
    }
};
</script>

<style>
.popup-title {
    height: 30px;
    font-size: 1.2em;
}
.confirm {
    width: 80px;
    border: 1px solid black;
    border-radius: 5px;
    cursor: pointer;
    background-color: rgb(30, 225, 30);
    padding: 5px;
    margin: 0 auto;
    margin-top: 10px;
}
.disabled {
    background-color: rgba(30, 225, 30, 0.5);
}
.modal {
    text-align: center;
}
</style>