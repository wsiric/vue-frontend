<template>
  <v-btn @click="togglePlay" :icon="isPlaying ? 'mdi-pause' : 'mdi-play'" size="large"></v-btn>
  <div>
    <ul>
      <li v-for="(value, key) in display" :key="key">
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
const display = ref("")
const isPlaying = ref(false)
const currentIndex = ref(0)

onBeforeMount(async () => {
  population.value = await axios.get('http://localhost:5000/population/all');
  years.value = await axios.get('http://localhost:5000/population/years');

  const firstYear = years.value.data[0]
  display.value = {[firstYear] : population.value.data[firstYear]}
})

const togglePlay = async () => {
  isPlaying.value = !isPlaying.value;
  if (isPlaying.value) {
    await displayPopulation(currentIndex.value);
  }
}

async function displayPopulation(startIndex) {
  for (let i = startIndex; i < years.value.data.length; i++) {
    currentIndex.value = i;
    if (!isPlaying.value) {
      break;
    }
    const year = years.value.data[i];
    display.value = { [year]: population.value.data[year] };
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