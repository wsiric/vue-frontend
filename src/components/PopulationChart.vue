<template>
  <v-btn @click="togglePlay" :icon="isPlaying ? 'mdi-pause' : 'mdi-play'" size="large"></v-btn>
  <v-btn @click="resetButton" icon='mdi-cancel' size="large"></v-btn>
  <div>
    <ul>
      <li v-for="(value, key) in chartState" :key="key">
        Key: {{ key }} - Value: <div>{{ value.slice(0, 10) }}</div>
      </li>
    </ul>
  </div>
</template>

<script setup>
import axios from 'axios';
import { onBeforeMount } from 'vue'
import { ref } from '@vue/reactivity';

const population = ref("")
const years = ref("")
const chartState = ref("")
const isPlaying = ref(false)
const currentIndex = ref(0)

onBeforeMount(async () => {
  population.value = await axios.get('http://localhost:5000/population/all');
  years.value = await axios.get('http://localhost:5000/population/years');

  initializeDisplayPopulation()
})

const togglePlay = async () => {
  isPlaying.value = !isPlaying.value;
  if (isPlaying.value) {
    await displayPopulation(currentIndex.value);
  }
}

const initializeDisplayPopulation = () => {
  const firstYear = years.value.data[0]
  chartState.value = {[firstYear] : population.value.data[firstYear]}
} 

const resetButton = async() => {
  isPlaying.value = false; 
  currentIndex.value = 0;
  initializeDisplayPopulation()
}

async function displayPopulation(startIndex) {
  for (let i = startIndex; i < years.value.data.length; i++) {
    currentIndex.value = i;
    if (!isPlaying.value) {
      break;
    }
    const year = years.value.data[i];
    chartState.value = { [year]: population.value.data[year] };
    await delay(100);
  }
  if (currentIndex.value === years.value.data.length - 1) {
    isPlaying.value = false; 
    currentIndex.value = 0;
  }
}


async function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
</script>