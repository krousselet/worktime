<template>
  <div>
    <h2>Résumé des semaines</h2>
    <div v-for="(week, i) in weeks" :key="i">
      <p>Semaine {{ i + 1 }} — Début : {{ week.date }}</p>
      <p>Matin : {{ week.matin.join(' / ') }}</p>
      <p>Après-midi : {{ week.aprem.join(' / ') }}</p>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const weeks = ref([])

onMounted(() => {
  const stored = localStorage.getItem('workWeeks')
  if (stored) weeks.value = JSON.parse(stored)
})
</script>

