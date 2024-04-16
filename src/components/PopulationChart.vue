<template>
  <v-btn @click="togglePlay" :icon="isPlaying ? 'mdi-pause' : 'mdi-play'" size="large"></v-btn>
  <v-btn @click="resetButton" icon='mdi-cancel' size="large"></v-btn>
  <v-btn @click="test" icon='mdi-cancel' size="large"></v-btn>
  <div>
    <ul>
      <li v-for="(value, key) in chartState" :key="key">
        Key: {{ key }} - Value: <div>{{ value.slice(0, 10) }}</div>
      </li>
    </ul>
  </div>
  <v-container>
    <div>
      <canvas id="myChart"></canvas>
    </div>
  </v-container>

</template>

<script setup>
import axios from 'axios';
import { onBeforeMount, onMounted } from 'vue'
import { ref } from '@vue/reactivity';
import Chart from 'chart.js/auto';
import { shallowRef } from 'vue';

const labels = ref([])

const chartData = {
  labels: labels.value,
  datasets: [{
    label: "Population growth per country, 1950 to 2021",
    borderColor: 'rgb(255,0 ,0)',
    data: [],
  }]
}

const config = {
  type: 'bar',
  data: chartData,
  options: {
    indexAxis: 'y',
  }
}

const nTop = 10
const population = ref("")
const years = ref("")
const chartState = ref("")
const isPlaying = ref(false)
const currentIndex = ref(0)
const myChart = ref(null)

onMounted(() => {
  myChart.value = shallowRef(new Chart(
    document.getElementById('myChart'),
    config
  ))
})
onBeforeMount(async () => {
  population.value = await axios.get('http://localhost:5000/population/all');
  years.value = await axios.get('http://localhost:5000/population/years');

  await initializeDisplayPopulation()
})

const test = () => {
  changeChartData(["h",'a'], [0,1])
  // myChart.value.update();
}
const togglePlay = async () => {
  isPlaying.value = !isPlaying.value;
  if (isPlaying.value) {
    await displayPopulation(currentIndex.value);
  }
}

const initializeDisplayPopulation = async () => {
  const firstYear = years.value.data[0]
  chartState.value = { [firstYear]: population.value.data[firstYear] }
  const nTopCountryList = chartState.value[firstYear].slice(0, nTop);
  const countryNames = nTopCountryList.map(item => item[0]);
  const countryPopulation = nTopCountryList.map(item => item[1]);
  changeChartData(countryNames, countryPopulation)
}

// bug that when user click reset button, its currentIndex get rewrite just after set to 0
const resetButton = async () => {
  isPlaying.value = false;
  await delay(300)
  currentIndex.value = 0;
  console.log("resetButton : " + currentIndex.value)
  initializeDisplayPopulation();
}

async function displayPopulation(startIndex) {
  for (let i = startIndex; i < years.value.data.length; i++) {
    currentIndex.value = i;
    console.log("displayPop : " + currentIndex.value)
    if (!isPlaying.value) {
      break;
    }
    const year = years.value.data[i];
    chartState.value = { [year]: population.value.data[year] };
    await delay(250);
  }
  if (currentIndex.value === years.value.data.length - 1) {
    isPlaying.value = false;
    currentIndex.value = 0;
  }
}

function changeChartData(label, newData) {
  myChart.value.value.data.labels = label
  myChart.value.value.data.datasets[0].data = newData
  myChart.value.value.update();
}

function removeChartData() {
  myChart.value.value.data.labels = []
  myChart.value.value.data.datasets[0].data = []
  myChart.value.value.update();
}

async function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
</script>