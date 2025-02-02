<script setup lang="ts">
import { ref } from 'vue';
import UpcomingLaunch from './components/UpcomingLaunch.vue';

import LaunchList from './components/LaunchList.vue';


const filter = ref<'all' | 'success' | 'failure'>('all');
const selectedLaunch = ref(null);


const openModal = (launch: any) => {
  selectedLaunch.value = launch;
};
</script>

<template>
  <div class="container mx-auto p-4">
    <h1 class="text-4xl font-bold text-center mb-6"> SpaceX Launches 🚀</h1>
    <UpcomingLaunch />
    <LaunchList :filter="filter" @selectLaunch="openModal" />
  </div>
</template>


<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
