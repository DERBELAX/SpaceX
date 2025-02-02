<template>
  <div>
    <h2 class="text-xl font-bold">Derniers Lancements</h2>
    
    <select v-model="selectedFilter" class="mb-4 p-2 border rounded">
      <option value="all">Tous les lancements</option>
      <option value="success">Lancements réussis</option>
      <option value="failure">Lancements échoués</option>
    </select>
    
    <ul>
      <li v-for="launch in filteredLaunches" :key="launch.id" @click="openModal(launch)">
        {{ launch.name }} - {{ new Date(launch.date_utc).toLocaleDateString() }}
      </li>
    </ul>
    
    <h3 class="text-lg font-semibold mt-4">Top 10 des derniers lancements</h3>
    <ul>
      <li v-for="(launch, index) in latestLaunches" :key="launch.id">
        {{ index + 1 }}. {{ launch.name }} - {{ new Date(launch.date_utc).toLocaleDateString() }}
      </li>
    </ul>
    
    <div v-if="selectedLaunch" class="modal">
      <div class="modal-content">
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
        <button @click="selectedLaunch = null">Fermer</button>
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
    launchpadName: l.launchpad || 'Lieu inconnu', // Vérifier si l.launchpad est défini
    payloads: l.payloads ? l.payloads.map(p => p.name) : [], // Vérifier si l.payloads est défini
    customers: l.payloads ? l.payloads.flatMap(p => p.customers || []) : [] // Éviter les erreurs si customers est undefined
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
  background: white;
  padding: 20px;
  border-radius: 5px;
  width: 500px;
}
</style>