<template>
   <div class="flex flex-col justify-center items-center font-Avenir text-center color-[#2c3e50] m-16"> 
    <div class="font-semibold text-2xl">
      👋 Welcome to the simple mapcreate tool
    </div>
    <div class="flex flex-col items-center">
      <div class="w-3 h-3 rounded-full border-red-300 border"></div>
      <div class="h-9 border-l border-red-300"></div>
      <div class="w-3 h-3 rounded-full border-red-300 border"></div>
    </div>
    <div class="font-medium text-base text-gray-500">
      This tool can create, edit and allow you to easily download maps... 
    </div>
    <div>
      <button class="border-2 border-red-300 py-2 px-6 mt-7 hover:bg-red-50">Create</button>
    </div>
   </div> 
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'WelcomePage',
  components: {
  }
});
</script>

<style>

</style>