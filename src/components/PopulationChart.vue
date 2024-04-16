<template>
  <v-container class="mt-16" style="position: relative;">
    <div class="chart-container">
      <canvas id="myChart"></canvas>
      <div class="relative-year">{{ currentYearDisplay }}</div>
      <div class="relative-world-pop">Total: {{ currentWorldPopulationDisplay }}</div>
    </div>

    <v-row class="pl-16">
      <v-col cols="1">
        <v-btn @click="togglePlay" :icon="isPlaying ? 'mdi-pause' : 'mdi-play'" size="large"></v-btn>
      </v-col>
      <v-col>
        <v-btn @click="resetButton" icon='mdi-cancel' size="large"></v-btn>
      </v-col>
    </v-row>
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
    backgroundColor: [
      'rgba(255, 99, 132, 0.8)',
      'rgba(255, 159, 64, 0.8)',
      'rgba(255, 205, 86, 0.8)',
      'rgba(75, 192, 192, 0.8)',
      'rgba(54, 162, 235, 0.8)',
      'rgba(153, 102, 255, 0.8)',
      'rgba(201, 203, 207, 0.8)'
    ],
  }]
}

const config = {
  type: 'bar',
  data: chartData,
  options: {
    indexAxis: 'y',
    plugins: {
      legend: {
        labels: {
          // This more specific font property overrides the global property
          font: {
            size: 32
          }
        }
      }
    },
    scales: {
      y: {
        ticks: {
          font: {
            size: 18
          }
        },
        grid: {
          display: false
        }
      },
      x: {
        ticks: {
          font: {
            size: 18
          }
        }
      }
    }
  }
}

const nTop = 10
const currentYearDisplay = ref("")
const currentWorldPopulationDisplay = ref("")
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

const togglePlay = async () => {
  isPlaying.value = !isPlaying.value;
  if (isPlaying.value) {
    await displayPopulation(currentIndex.value);
  }
}

const initializeDisplayPopulation = async () => {
  const firstYear = years.value.data[0]
  currentYearDisplay.value = firstYear
  currentWorldPopulationDisplay.value = formatNumber(population.value.data[firstYear][0][1])
  chartState.value = { [firstYear]: population.value.data[firstYear] }
  computeNewChart(chartState, firstYear)
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
    currentYearDisplay.value = year
    currentWorldPopulationDisplay.value = formatNumber(population.value.data[year][0][1])
    chartState.value = { [year]: population.value.data[year] };
    computeNewChart(chartState, year)
    await delay(250);
  }
  if (currentIndex.value === years.value.data.length - 1) {
    isPlaying.value = false;
    currentIndex.value = 0;
  }
}

function computeNewChart(chartState, year) {
  const nTopCountryList = chartState.value[year].slice(1, nTop);
  const countryNames = nTopCountryList.map(item => item[0]);
  const countryPopulation = nTopCountryList.map(item => item[1]);
  changeChartData(countryNames, countryPopulation)
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

function formatNumber(number) {
  return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

</script>

<style>
.chart-container {
  position: relative;
}

.relative-year {
  position: absolute;
  color: #87857e;
  top: 66%;
  left: 81%;
  font-size: 64px;
  font-weight: bold;
}

.relative-world-pop {
  position: absolute;
  color: #87857e;
  top: 80%;
  left: 65%;
  font-size: 38px;
}
</style>