<template>
  <v-btn @click="test">
  Button
</v-btn>
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
import {onBeforeMount } from 'vue'
import { ref} from '@vue/reactivity';

const population = ref("")
const years = ref("")
const display = ref("")

onBeforeMount(async () => {
  population.value = await axios.get('http://localhost:5000/population/all');
  years.value = await axios.get('http://localhost:5000/population/years');
  await displayPopulation();
})

const test = () => {
  console.log(display.value)
}
async function displayPopulation() {
  for (const year of years.value.data) {
    await displayPopulationHelper(year);
  }
}

async function displayPopulationHelper(year) {
  display.value = { [year]: population.value.data[year] }
  await delay(700);
}

async function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
</script>