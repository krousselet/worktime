<template>
  <div>
    <div class="summary">
      Total des heures prévues : {{ totalTheoretical }}h — Heures réelles : {{ totalActual.toFixed(2) }}h
    </div>

    <button @click="addNewWeek">Ajouter une semaine</button>

    <div v-for="(week, index) in weeks" :key="index" class="week-container">
      <div>
        <strong>Date de début :</strong>
        <input type="date" v-model="week.date" @change="saveToStorage" />
      </div>

      <button @click="removeWeek(index)">Supprimer la semaine</button>

      <div>
        <strong>Semaine {{ index + 1 }} :</strong> Théorique : 30h / Réel : {{ getWeekHours(week).toFixed(2) }}h
      </div>

      <table>
        <tr>
          <th></th>
          <th v-for="day in days" :key="day">{{ day }}</th>
        </tr>

        <tr v-for="period in ['matin', 'aprem']" :key="period">
          <td>{{ period === 'matin' ? 'Matin' : 'Après-midi' }}</td>
          <td
            v-for="i in 6"
            :key="i"
            :class="{ edited: isEdited(week, period, i - 1) }"
          >
            <div class="original">{{ week.original?.[period]?.[i - 1] || '' }}</div>
            <input
              class="modified"
              type="text"
              v-model="week[period][i - 1]"
              @input="handleTextInput(week, period, i - 1)"
              placeholder="Ex: 8h30 - 12h"
            />
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script setup>
import { reactive, onMounted, computed } from 'vue'

const days = ['Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi']

const defaultWeek = () => ({
  date: new Date().toISOString().split('T')[0],
  matin: ['', '', '', '', '', ''],
  aprem: ['', '', '', '', '', ''],
  original: {
    matin: ['', '', '', '', '', ''],
    aprem: ['', '', '', '', '', '']
  },
  justCreated: true
})

const weeks = reactive([])

function saveToStorage() {
  localStorage.setItem('workWeeks', JSON.stringify(weeks))
}

function loadFromStorage() {
  const data = localStorage.getItem('workWeeks')
  if (data) {
    const parsed = JSON.parse(data)
    parsed.forEach(w => {
      if (!w.original) {
        w.original = JSON.parse(JSON.stringify({ matin: w.matin, aprem: w.aprem }))
      }
      if (typeof w.justCreated === 'undefined') {
        w.justCreated = false
      }
      weeks.push(w)
    })
  } else {
    const firstWeek = {
      date: new Date().toISOString().split('T')[0],
      matin: ['8h45 - 12h00', 'Repos', '9h - 14h15', '9h15 - 12h15', 'Repos', '9h - 13h30'],
      aprem: ['Repos', 'Repos', 'Repos', '14h15 - 19h30', '14h15 - 19h45', '16h - 19h15'],
      original: {},
      justCreated: false
    }
    firstWeek.original = JSON.parse(JSON.stringify(firstWeek))
    weeks.push(firstWeek)
  }
}

function timeToHours(timeStr) {
  if (timeStr.trim().toLowerCase() === 'repos' || timeStr.trim() === '') return 0
  let total = 0
  const ranges = timeStr.split(',')
  for (let range of ranges) {
    const [start, end] = range.trim().split('-')
    if (!start || !end) continue
    const [h1, m1] = start.trim().replace('h', ':').split(':').map(Number)
    const [h2, m2] = end.trim().replace('h', ':').split(':').map(Number)
    total += (h2 + m2 / 60) - (h1 + m1 / 60)
  }
  return Math.max(0, total)
}

function addNewWeek() {
  const newWeek = defaultWeek()
  weeks.push(newWeek)
  saveToStorage()
}

function removeWeek(index) {
  if (confirm('Supprimer cette semaine ?')) {
    weeks.splice(index, 1)
    saveToStorage()
  }
}

function handleTextInput(week, period, dayIdx) {
  const value = week[period][dayIdx]

  if (week.justCreated) {
    week.original[period][dayIdx] = value

    const isFilled = ['matin', 'aprem'].every(type =>
      week[type].every((v, i) => v === week.original[type][i] && v.trim() !== '')
    )

    if (isFilled) {
      week.justCreated = false
    }
  }

  saveToStorage()
}

function isEdited(week, period, index) {
  const original = week.original?.[period]?.[index] ?? ''
  const current = week[period][index] ?? ''
  return current !== original
}

function getWeekHours(week) {
  let hours = 0
  for (let i = 0; i < 6; i++) {
    hours += timeToHours(week.matin[i]) + timeToHours(week.aprem[i])
  }
  return hours
}

const totalTheoretical = computed(() => weeks.length * 30)
const totalActual = computed(() => weeks.reduce((sum, week) => sum + getWeekHours(week), 0))

onMounted(() => {
  loadFromStorage()
})
</script>

<style scoped>
table {
  margin: 25px auto;
  border-collapse: separate;
  border-spacing: 0;
}
th, td {
  width: 150px;
  padding: 5px;
  border: 1px solid black;
  text-align: center;
}
.summary {
  text-align: center;
  margin-bottom: 20px;
}
.week-container {
  border: 2px solid #444;
  margin: 20px auto;
  padding: 10px;
  width: fit-content;
}
button {
  display: block;
  margin: 20px auto;
}
.edited {
  background-color: #fff3cd;
}
.original {
  font-size: 0.8em;
  color: gray;
  margin-bottom: 2px;
}
.modified {
  font-size: 1em;
  width: 100%;
  box-sizing: border-box;
}
</style>
