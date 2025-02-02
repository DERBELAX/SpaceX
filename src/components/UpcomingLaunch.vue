<template>
  <div class="bg-gray-800 text-white p-4 rounded-lg shadow-lg">
    <h2 class="text-xl font-bold">Prochain Lancement</h2>
    <p v-if="launch">{{ launch.name }} - {{ formattedDate }}</p>
    <p>Décompte : {{ countdown }}s</p>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

const launch = ref<any>(null);
const countdown = ref(0);

onMounted(async () => {
  const res = await axios.get('https://api.spacexdata.com/v5/launches/next');
  launch.value = res.data;
  updateCountdown();
  setInterval(updateCountdown, 1000);
});

const updateCountdown = () => {
  if (!launch.value) return;
  const launchDate = new Date(launch.value.date_utc).getTime();
  countdown.value = Math.max(0, Math.floor((launchDate - Date.now()) / 1000));
};

const formattedDate = computed(() => new Date(launch.value?.date_utc).toLocaleString());
</script>
