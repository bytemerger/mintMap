<template>
    <div class="ml-10">
        <div class="text-red-700 font-bold text-2xl">Show the map here</div>
        <div>
            <button class="border-2 border-gray-600 py-2 px-6 mt-7 hover:bg-red-300 bg-slate-300" @click="emit('exitMap')">Exit</button>
        </div>
    </div>
    <div id="map" class="top-0 absolute bottom-0 w-full"></div>
</template>
<script setup lang="ts">
import { onMounted } from '@vue/runtime-core';
import { defineEmits } from "vue"
import mapboxgl from 'mapbox-gl';

const emit = defineEmits<{
  (e: 'exitMap'): void
}>()

mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_TOKEN;

onMounted(()=>{
    const map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/streets-v11', 
        center: [-74.5, 40],
        zoom: 9 
    }); 
})
</script>
<style scoped>
#map { position: absolute; top: 0; bottom: 0; width: 100%; z-index: -10; }
</style>
