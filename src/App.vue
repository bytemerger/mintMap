<template>
   <welcome-page/> 
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import WelcomePage from './components/WelcomePage.vue'

export default defineComponent({
  name: 'App',
  components: {
    WelcomePage
  }
});
</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
