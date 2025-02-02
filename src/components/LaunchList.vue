<template>
  <div>
    <h2 class="text-xl font-bold">Derniers Lancements</h2>
    
    <select v-model="selectedFilter" class="mb-4 p-2 border rounded">
      <option value="all">Tous les lancements</option>
      <option value="success">Lancements réussis</option>
      <option value="failure">Lancements échoués</option>
    </select>
    
    <ul class="list-none p-0 m-0" style="list-style-type: none !important;">
  <li v-for="launch in filteredLaunches" :key="launch.id" @click="openModal(launch)">
    {{ launch.name }} - {{ new Date(launch.date_utc).toLocaleDateString() }}
  </li>
</ul>
    

<h3 class="text-lg font-semibold mt-4">Top 10 des derniers lancements</h3>
<ul class="list-none"  style="list-style-type: none !important;">
  <li v-for="(launch, index) in latestLaunches" :key="launch.id" @click="openModal(launch)">
    {{ index + 1 }}. {{ launch.name }} - {{ new Date(launch.date_utc).toLocaleDateString() }}
  </li>
</ul>



    <div v-if="selectedLaunch" class="modal fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center transition-opacity duration-300">
  <div class="modal-content bg-white p-6 rounded-lg shadow-lg w-96 transform scale-95 opacity-0 transition-all duration-300 ease-out" 
    :class="{'opacity-100 scale-100': selectedLaunch, 'opacity-0 scale-95': !selectedLaunch}" @click.stop>
    <h3 class="text-xl font-bold">{{ selectedLaunch.name }}</h3>
    <p>Date: {{ new Date(selectedLaunch.date_utc).toLocaleDateString('fr-FR') }}</p>
    <p>Description: {{ selectedLaunch.details || 'Aucune description disponible' }}</p>
    <img :src="selectedLaunch.links.patch.small" alt="Mission Image" class="my-2" />
    <p><a :href="selectedLaunch.links.article" target="_blank" class="text-blue-500">Article de présentation</a></p>
    <label>
      <input type="checkbox" v-model="showVideo" /> Voir la vidéo de la mission
    </label>
    <div v-if="showVideo">
      <iframe :src="`https://www.youtube.com/embed/${selectedLaunch.links.youtube_id}`" frameborder="0" allowfullscreen></iframe>
    </div>
    <p>Lieu de lancement: {{ selectedLaunch.launchpadName }}</p>
    <p>Chargements envoyés: {{ selectedLaunch.payloads.join(', ') }}</p>
    <p>Clients: {{ selectedLaunch.customers.join(', ') }}</p>
    <button @click="selectedLaunch = null" class="mt-4 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600">Fermer</button>
  </div>
</div>

  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

const launches = ref<any[]>([]);
const selectedFilter = ref<'all' | 'success' | 'failure'>('all');
const selectedLaunch = ref<any | null>(null);
const showVideo = ref(false);

onMounted(async () => {
  const res = await axios.get('https://api.spacexdata.com/v5/launches/past?limit=10');
  launches.value = res.data.map(l => ({
    ...l,
    launchpadName: l.launchpad || 'Lieu inconnu', 
    payloads: l.payloads ? l.payloads.map(p => p.name) : [], 
    customers: l.payloads ? l.payloads.flatMap(p => p.customers || []) : [] 
  }));
});


const filteredLaunches = computed(() => {
  if (selectedFilter.value === 'all') return launches.value;
  return launches.value.filter(l => (selectedFilter.value === 'success' ? l.success : !l.success));
});
const latestLaunches = computed(() => {
  return launches.value.slice(0, 10);
});

const openModal = (launch: any) => {
  selectedLaunch.value = launch;
  showVideo.value = false;
};
</script>

<style>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal-content {
  background: rgba(244, 229, 229, 0.876);
  padding: 20px;
  border-radius: 5px;
  width: 500px;
}
</style>