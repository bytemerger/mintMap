<template>
  <welcome-page v-if="page === 'welcome'" @create-map = "goToMap('map')"/> 
  <keep-alive>
    <show-map v-if="page ==='map'" @exit-map = "goToMap('welcome')" />
  </keep-alive>
</template>

<script setup lang="ts">
import { ref } from '@vue/reactivity'
import WelcomePage from './components/WelcomePage.vue'
import ShowMap from './components/ShowMap.vue'

type pageType = 'welcome' | 'map';

const page = ref<pageType>('welcome')

function goToMap(value: pageType){
  page.value = value
}


</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
