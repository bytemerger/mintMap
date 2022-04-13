<template>
    <div class="flex justify-between">
        <div class="ml-7 flex flex-col items-center w-10">
            <div>
                <button class="border-2 border-gray-600 py-1 px-3 mt-7 hover:bg-red-300 bg-slate-300" @click="emit('exitMap')">Exit</button>
            </div>
            <div class="flex flex-col mt-40">
                <button class="border-2 border-gray-600 py-1 px-3 hover:bg-red-300 bg-slate-300" @click="zoomIn">+</button>
                <button class="border-2 border-gray-600 py-1 px-3 mt-5 hover:bg-red-300 bg-slate-300" @click="zoomOut">-</button>
            </div>
        </div>
        <div class="mr-7 mt-4 flex flex-col items-end">
            <div>
                <input type="text" class="border-red-400 border-2 focus:outline-none py-1 px-1 focus:px-3 w-14 text-xs rounded focus:w-full focus:text-base peer" placeholder="Search ..." v-model="searchQuery"/>
                <div class="w-13 hidden peer-focus:block">
                    <div class="bg-white border mt-2">
                        <ul>
                            <li v-for="result in searchResult.data.features" :key="result.id" class="hover:bg-gray-400 p-2 cursor-pointer" @mousedown.prevent="goToMap(result.center)">{{result.place_name}}</li>
                        </ul>
                    </div>
                    <hr>
                </div>
            </div>
            <div class="flex mt-32">
                <div class="h-80 overflow-scroll bg-red-400 text-xs px-5" v-if="showLayers">
                    <ul>
                        <li v-for="(layerTypes, type) in sortedLayers" :key="type" class="group hover:font-medium hover:text-sm">{{type}}
                            <ul class="hidden group-hover:block ml-4">
                                <li v-for="layer in layerTypes" :key="layer.id" class="hover:font-bold cursor-pointer" @click="toggleLayer(layer.id)">
                                    - {{layer.id}} | <span class="text-xs bg-green-400" v-if="layerState[layer.id]">show</span> <span v-else class="text-xs bg-red-700">hide</span>
                                </li>
                            </ul>
                        </li>
                    </ul>
                </div>
                <div>
                    <button class="border-2 border-gray-600 py-1 px-2 hover:bg-red-300 bg-slate-300 text-xs" @click="()=>showLayers = !showLayers">Layers</button>
                </div>
            </div>
        </div>
    </div>
    <div id="map" class="top-0 absolute bottom-0 w-full"></div>
</template>
<script setup lang="ts">
import { onMounted, reactive, watchEffect } from '@vue/runtime-core';
import { computed, defineEmits, ref } from "vue"
import mapboxgl,{ LngLatLike, Map, AnyLayer, Layer } from 'mapbox-gl';

const emit = defineEmits<{
  (e: 'exitMap'): void
}>()

const loading = ref(true);
let map : Map;
const layers =  ref<Layer[]>();
const showLayers = ref(false)

mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_TOKEN;


const sortedLayers = computed(() => {
  let sortedStyles: Record<string, Layer[]> = {}
        if(layers.value){
            for(let mapStyle of layers.value){
                if(!mapStyle['source-layer']){
                    sortedStyles['general'] ? sortedStyles['general'].push(mapStyle): sortedStyles['general'] = [mapStyle]
                    continue
                    /* for(const key in Object.keys(sortedStyles)){
                        console.log(key)
                        let regex = new RegExp(mapStyle.id,'g')
                        if(key.match(regex)){
                            if(!sortedStyles[key]){
                                sortedStyles[key] = [mapStyle];
                                break;
                            }
                            sortedStyles[key].push(mapStyle);
                            break;
                        }
                    } */
                }
                if(!sortedStyles[mapStyle['source-layer']]){
                    sortedStyles[mapStyle['source-layer']] = [mapStyle]
                    continue
                }
                sortedStyles[mapStyle['source-layer']].push(mapStyle)
            }
        }
        return sortedStyles;
})

type LayerState = {
    [index : Layer["id"]]: true | false
}
let layerState:LayerState = reactive({}) ;

onMounted(()=>{
    map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/streets-v11', 
        center: [-74.5, 40],
        zoom: 9 
    }); 
    map.on('load', ()=>{
        const mapLayers: any = map.getStyle().layers
        // Set layer state
        for(let layers of mapLayers){
            layerState[layers.id] = false;
        }
        layers.value = mapLayers;
        loading.value = false;
    })
})

function zoomIn(){
    map.zoomIn();
}
function zoomOut(){
    map.zoomOut();
}

// search for a particular location
let timeOut: number|null = null;
const searchQuery = ref('');
let searchResult = reactive({
    data:{}
});
watchEffect(() => {
    if(searchQuery.value !== ""){
        if(timeOut !== null){
            return;
        }
       timeOut = setTimeout(()=>{
            console.log(searchQuery.value)
            searchLocation(searchQuery.value)
            timeOut = null
       },400)
    }
})
async function searchLocation(q: string){
    let url = `https://api.mapbox.com/geocoding/v5/mapbox.places/${q}.json?access_token=${process.env.VUE_APP_MAPBOX_TOKEN}`;
    const response = await fetch(url);
    searchResult.data = await response.json();
    console.log(searchResult.data)
}
function goToMap(location: LngLatLike){
    map.flyTo({
        center: location
    })
}

function toggleLayer(layer: Layer['id']){
    const currentState = layerState[layer]
    // if currentState = false then it is visible
    const setAs = currentState ? 'visible' : 'none'
    map.setLayoutProperty(layer,'visibility',setAs)
    layerState[layer] = !currentState
}
</script>
<style scoped>
#map { position: absolute; top: 0; bottom: 0; width: 100%; z-index: -10; }
</style>
