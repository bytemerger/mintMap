<template>
    <div class="ml-10 flex flex-col items-center w-10">
        <div>
            <button class="border-2 border-gray-600 py-2 px-6 mt-7 hover:bg-red-300 bg-slate-300" @click="emit('exitMap')">Exit</button>
        </div>
        <div class="flex flex-col mt-40">
            <button class="border-2 border-gray-600 py-1 px-3 hover:bg-red-300 bg-slate-300" @click="zoomIn">+</button>
            <button class="border-2 border-gray-600 py-1 px-3 mt-5 hover:bg-red-300 bg-slate-300" @click="zoomOut">-</button>
        </div>
    </div>
    <div id="map" class="top-0 absolute bottom-0 w-full"></div>
</template>
<script setup lang="ts">
import { onMounted } from '@vue/runtime-core';
import { defineEmits, ref } from "vue"
import mapboxgl,{ Map } from 'mapbox-gl';

const emit = defineEmits<{
  (e: 'exitMap'): void
}>()

const loading = ref(true);
let map : Map;
mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_TOKEN;

onMounted(()=>{
    map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/streets-v11', 
        center: [-74.5, 40],
        zoom: 9 
    }); 
    map.on('load',()=>{
        console.log(map)
        loading.value = false;
    })
})

function zoomIn(){
    map.zoomIn();
}
function zoomOut(){
    map.zoomOut();
}

</script>
<style scoped>
#map { position: absolute; top: 0; bottom: 0; width: 100%; z-index: -10; }
</style>
