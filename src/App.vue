<template>
  <welcome-page v-if="page === 'welcome'" @create-map = "goToMap"/> 
  <show-map v-if="page ==='map'"/>
</template>

<script setup lang="ts">
import { ref } from '@vue/reactivity'
import WelcomePage from './components/WelcomePage.vue'
import ShowMap from './components/ShowMap.vue'

const page = ref<'welcome' | 'map'>('welcome')

function goToMap(){
  page.value = "map"
}


</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
