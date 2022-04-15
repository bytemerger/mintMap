<template>
    <div class="flex justify-between z-10 relative pointer-events-none" v-if="!loading">
        <div class="ml-7 flex flex-col items-center w-10 pointer-events-auto">
            <div>
                <button class="border-2 border-gray-600 py-1 px-3 mt-7 hover:bg-red-300 bg-slate-300" @click="emit('exitMap')">Exit</button>
            </div>
            <div class="flex flex-col mt-40">
                <button class="border-2 border-gray-600 py-1 px-3 hover:bg-red-300 bg-slate-300" @click="zoomIn">+</button>
                <button class="border-2 border-gray-600 py-1 px-3 mt-5 hover:bg-red-300 bg-slate-300" @click="zoomOut">-</button>
            </div>
        </div>
        <div class="mr-7 mt-4 flex flex-col items-end">
            <!-- Search box -->
            <div class="pointer-events-auto">
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
            <!-- Other tools-->
            <div class="flex flex-col mt-32 pointer-events-auto">
                <div class="flex flex-row-reverse">
                    <div>
                        <button class="border-2 border-gray-600 py-1 px-2 hover:bg-red-300 bg-slate-300 text-xs" @click="()=>showTools === 'layers' ? showTools = null : showTools = 'layers'">Layers</button>
                    </div>
                    <div class="h-80 overflow-scroll bg-red-400 text-xs px-5" v-if="showTools === 'layers'">
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
                </div>
                <div class="mt-7 flex flex-row-reverse">
                    <div class="ml-1">
                        <button class="border-2 border-gray-600 py-1 px-2 hover:bg-red-300 bg-slate-300 text-xs" @click="()=>showTools === 'size' ? showTools = null : showTools = 'size'">Resize</button>
                    </div>
                    <div class="flex" v-if="showTools === 'size'">
                        <div>
                            <input type="number" placeholder="width" class="w-16" v-model="mapSize.width"/> 
                        </div> 
                        <div class="ml-3">
                            <input type="number" placeholder="height" class="w-16" v-model="mapSize.height"/> 
                        </div> 
                    </div>
                </div>
                <div class="mt-7 flex flex-row-reverse">
                    <div class="ml-1">
                        <button class="border-2 border-gray-600 py-1 px-2 hover:bg-red-300 bg-slate-300 text-xs" @click="()=>showTools === 'addText' | 'addIcon' ? showTools = null : showTools = 'addText'">Add</button>
                    </div>
                    <div class="flex flex-col bg-gray-500" v-if="(showTools === 'addText' || showTools === 'addIcon')">
                        <div class="p-2 flex justify-between">
                            <span @click="()=>showTools === 'addText' ? '' : showTools = 'addText'" class="bg-black text-white tex-xs p-1 cursor-pointer">Text</span>
                            <span @click="()=>showTools === 'addIcon' ? '' : showTools = 'addIcon'" class="bg-black text-white tex-xs p-1 cursor-pointer">Icon</span>
                        </div>
                        <div v-if="showTools === 'addText'" class="flex flex-col">
                            <div class="flex flex-col m-2">
                                <input type="text" placeholder="text" v-model="textToAdd.text" class="border border-red-400"/> 
                                <input type="number" placeholder="size in px" v-model="textToAdd.size" class="border border-red-400 my-2"/>
                                <input type="text" placeholder="color" v-model="textToAdd.color" class="border border-red-400"/>
                                <div class="mt-2">
                                    <button @click="addText" class="border-2 border-gray-600 py-1 px-2 hover:bg-red-300 bg-slate-300 text-xs">Add</button>
                                </div>
                            </div>
                        </div>
                        <div v-if="showTools === 'addIcon'" class="flex flex-col">
                            <div class="m-2">
                                <div class="flex">
                                    <div  v-for="(icon, key) in icons" :key="key" class="h-9 w-9">
                                        <img :src="icon" @click="addIcon(key)"/>
                                    </div>
                                </div>
                                <div class="bg-black text-white tex-xs p-1 cursor-pointer" @click="addIcon('default')">Default</div>
                            </div>
                        </div> 
                    </div>
                </div>
                <div class="mt-20 self-end">
                    <div>
                        <button class="border-2 border-gray-600 py-1 px-2 hover:bg-red-300 bg-slate-300 text-xs" @click="exportMap">Export</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div v-if="(mapSize.width && !loading)" class="absolute z-10 top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 border border-gray-600 pointer-events-none" :style="{'width': `${mapSize.width}px`, 'height': `${mapSize.height}px`}">
        
    </div>
    <!-- show clean ui when exporting -->
    <div class="fixed top-0 left-0 w-full h-screen bg-black z-20" v-if="loading"></div>
    <div v-if="loading" class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 z-50">
        <img src="../assets/Settings.gif" />
        <div class="text-white">Processing the export....</div>
    </div>
    <div id="map" class="top-0 absolute bottom-0 w-full"></div>
</template>
<script setup lang="ts">
import { onMounted, reactive, watch, watchEffect } from '@vue/runtime-core';
import { computed, defineEmits, ref } from "vue"
import mapboxgl,{ LngLatLike, Map, AnyLayer, Layer, Marker } from 'mapbox-gl';
import html2canvas from 'html2canvas';

const emit = defineEmits<{
  (e: 'exitMap'): void
}>()

const loading = ref(false);
let map : Map;
const layers =  ref<Layer[]>();
const showTools = ref<'layers'| 'size' | 'addText'| 'addIcon' | null>(null)
const mapSize = reactive({
    width: 0,
    height: 0
});
const textToAdd = reactive({
    text: '',
    size: 12,
    color: 'green'
});

// Watch the width change
watch(
  () => mapSize.width,
  (newValue, oldValue) => {
    if(isNaN(newValue)){
        alert("please pass width in pixel")
        mapSize.width = oldValue
    }
    if(newValue > window.screen.width){
        alert("The set pixel is above you screen width")
        mapSize.width = oldValue
    }
  }
)
watch(
  () => mapSize.height,
  (newValue, oldValue) => {
    if(isNaN(newValue)){
        alert("please pass height in pixel")
        mapSize.height = oldValue
    }
    if(newValue > window.screen.height){
        alert("The set pixel is above you screen width")
        mapSize.height = oldValue
    }
  }
)

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
        zoom: 9,
        preserveDrawingBuffer: true
    }); 
    map.on('load', ()=>{
        const mapLayers: any = map.getStyle().layers
        // Set layer state
        for(let layers of mapLayers){
            layerState[layers.id] = false;
        }
        layers.value = mapLayers;
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

function setSize(type: 'width' | 'height', value: number){
    console.log(window.screen)
    if(!isNaN(value)){
        alert("please pass " + type + " in pixel")
        return
    }
    if(type === 'width'){
        if(value> window.screenX){
            alert("The set pixel is above you screen width")
            return
        }
    }
    if(type === 'height'){
        if(value> window.screenY){
            alert("The set pixel is above you screen height")
            return
        }
    }
    mapSize[type] = value
}

//Add ICON and TEXT
const icons: { [key: string]: any } = {
    "profileSvg": require("./../assets/icons/basic-profile-ui-svgrepo-com.svg"),
    "touchSvg": require("../assets/icons/basic-touch-ui-svgrepo-com.svg"),
    "homeSvg": require("../assets/icons/basic-home-svgrepo-com.svg")
}
function addText(){
    if(textToAdd.text !== ""){
        let text = document.createElement('span')
        text.innerHTML = `<span class="text-[${textToAdd.size}px]" style="color:${textToAdd.color}; font-size:${textToAdd.size}px">${textToAdd.text}</span>`
        const deleteButton = document.createElement('div')
        deleteButton.innerHTML = `<button class="bg-red-400 font-medium rounded-md">Delete</button>`
        const {lng, lat} = map.getCenter();
        const marker = new mapboxgl.Marker({
                            element: text,
                            draggable: true
                        });
        deleteButton.addEventListener('click',(e)=>{
            marker.remove()
            setTimeout(() => {
                alert('Deleted successfully')
            }, 300);
        })
        marker.setPopup(new mapboxgl.Popup().setDOMContent(deleteButton))
            .setLngLat([lng, lat])
            .addTo(map);

        textToAdd.text = '';
        showTools.value = null;
        setTimeout(() => {
            alert("You can drag the element to the preferred location | To delete text click")
        }, 300);
    }
}
function addIcon(key: string){
    let marker: Marker;
    if(key === 'default'){
         marker = new mapboxgl.Marker({
                        draggable: true
                    })
    }else{
        let icon = document.createElement('div')
        icon.innerHTML = `<img src="${icons[key]}" class="w-9 h-9"/>`
        marker = new mapboxgl.Marker({
                        element: icon,
                        draggable: true
                    })
    }
    const {lng, lat} = map.getCenter();

    //set delete button
    const deleteButton = document.createElement('div')
    deleteButton.innerHTML = `<button class="bg-red-400 font-medium rounded-md">Delete</button>`
    deleteButton.addEventListener('click',(e)=>{
        marker.remove()
        setTimeout(() => {
            alert('Deleted successfully')
        }, 300);
    })
    // add to map
    marker.setPopup(new mapboxgl.Popup().setDOMContent(deleteButton))
            .setLngLat([lng, lat])
            .addTo(map);

    showTools.value = null
    setTimeout(() => {
        alert("You can drag the element to the preferred location | To delete icon click")
    }, 300);
}
function exportCanvasAsPNG(canvasElement: any, fileName: string) {
    const MIME_TYPE = "image/png";

    const imgURL = canvasElement.toDataURL(MIME_TYPE);

    const dlLink = document.createElement('a');
    dlLink.download = fileName;
    dlLink.href = imgURL;
    dlLink.dataset.downloadurl = [MIME_TYPE, dlLink.download, dlLink.href].join(':');

    document.body.appendChild(dlLink);
    dlLink.click();
    document.body.removeChild(dlLink);
}
function exportMap(){
    if(mapSize.width === 0 || mapSize.height === 0){
        alert('please set the map export size')
        showTools.value = 'size'
        return;
    }
    let container = map.getContainer();
    let formerwidth  = window.screen.availWidth;
    let formerheight = window.screen.availHeight;

    container.style.height = `${mapSize.height}px`
    container.style.width = `${mapSize.width}px`

    let center = map.getCenter();
    loading.value = true
    map.resize()

    map.once('idle',()=>{
        html2canvas(container).then((element)=>{
            exportCanvasAsPNG(element, 'newMap')
            container.style.height = `${formerheight}px`;
            container.style.width = `${formerwidth}px`;
            map.resize()
            map.setCenter(center);
        }).finally(()=> {
            setTimeout(()=>{
                loading.value = false
            },1000)
        })
    })
}
</script>
<style scoped>
#map { position: absolute; top: 0; bottom: 0; width: 100%; }
</style>
