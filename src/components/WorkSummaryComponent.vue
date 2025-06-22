<template>
  <div class="summary-container">
    <h2>Résumé des semaines</h2>
    <div v-for="(week, i) in weeks" :key="i" class="week-card">
      <h3>Semaine {{ i + 1 }}</h3>
      <p><strong>Début :</strong> {{ week.date }}</p>
      <div class="time-section">
        <p><strong>Matin :</strong></p>
        <ul>
          <li v-for="(slot, idx) in week.matin" :key="'matin-' + idx">{{ days[idx] }} : {{ slot || '—' }}</li>
        </ul>
      </div>
      <div class="time-section">
        <p><strong>Après-midi :</strong></p>
        <ul>
          <li v-for="(slot, idx) in week.aprem" :key="'aprem-' + idx">{{ days[idx] }} : {{ slot || '—' }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const weeks = ref([])
const days = ['Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi']

onMounted(() => {
  const stored = localStorage.getItem('workWeeks')
  if (stored) weeks.value = JSON.parse(stored)
})
</script>

<style scoped>
.summary-container {
  max-width: 800px;
  margin: 40px auto;
  padding: 0 20px;
  font-family: Arial, sans-serif;
}

h2 {
  text-align: center;
  margin-bottom: 30px;
  font-size: 24px;
  color: #333;
}

.week-card {
  border: 1px solid #ccc;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 20px;
  background-color: #f9f9f9;
  box-shadow: 2px 2px 6px rgba(0, 0, 0, 0.05);
}

.week-card h3 {
  margin-top: 0;
  color: #2a4d69;
}

.time-section {
  margin-top: 10px;
}

.time-section ul {
  padding-left: 20px;
  margin: 5px 0;
}

.time-section li {
  margin-bottom: 4px;
}

ul, li {
    list-style: none;
}
</style>
