<template>
  <h1 class="title">Population growth per country, 1950 to 2021</h1>
  <v-container style="position: relative;">
    <div class="chart-container">
      <canvas id="myChart"></canvas>
      <div class="relative-year">{{ currentYear }}</div>
      <div class="relative-world-pop">Total: {{ totalPop }}</div>
    </div>

    <v-row class="pl-16">
      <v-col cols="1">
        <v-btn @click="togglePlay" :icon="isPlaying ? 'mdi-pause' : 'mdi-play'" size="large"></v-btn>
      </v-col>
      <v-col>
        <v-btn @click="resetButton" icon='mdi-cancel' size="large"></v-btn>
        <v-btn @click="nTopCountry" icon='mdi-cancel' size="large"></v-btn>
      </v-col>
    </v-row>
  </v-container>

</template>

<script setup>
import axios from 'axios';
import { onBeforeMount, shallowRef, nextTick, ref } from 'vue'
import Chart from 'chart.js/auto';
import ChartDataLabels from 'chartjs-plugin-datalabels';

const labels = ref([])
const chartData = {
  labels: labels.value,
  datasets: [{
    label: "Population growth per country, 1950 to 2021",
    borderColor: 'rgb(255,0 ,0)',
    data: [],
  },
  {
    label: 'Asia',
    borderColor: 'red',
  },
  {
    label: 'Europe',
    borderColor: 'red',
  },
  ]
}

const config = {
  type: 'bar',
  data: chartData,
  options: {
    indexAxis: 'y',
    plugins: {
      datalabels: {
        align: 'end',
        anchor: 'end',
        color: 'black',
        font: {
          size: 12
        },
        formatter: (value) => {
          return formatNumber(value);
        }
      },
      legend: {
        display: true,
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
            size: 12
          }
        }
      }
    }
  },
  plugins: [ChartDataLabels]
}

const nTop = 10
const currentYear = ref(null)
const totalPop = ref(null)
const population = ref(null)
const asiaPop = ref(null)
const euPop = ref(null)
const africaPop = ref(null)
const ocPop = ref(null)
const naPop = ref(null)
const saPop = ref(null)
const toggleAsia = ref(true)
const toggleEu = ref(true)
const toggleAfrica = ref(true)
const toggleOc = ref(true)
const toggleNa = ref(true)
const toggleSa = ref(true)
const years = ref(null)
const chartState = ref(null)
const isPlaying = ref(false)
const currentIndex = ref(0)
const myChart = ref(null)

onBeforeMount(async () => {
  await new Promise(resolve => {
    nextTick(() => {
      myChart.value = shallowRef(new Chart(
        document.getElementById('myChart'),
        config
      ));
      resolve();
    });
  });

  const cachedData = await sessionStorage.getItem('cachedData');
  if (cachedData) {
    const cachedJSONObject = JSON.parse(cachedData);
    population.value = cachedJSONObject.population
    asiaPop.value = cachedJSONObject.asiaPop;
    euPop.value = cachedJSONObject.euPop;
    africaPop.value = cachedJSONObject.africaPop;
    ocPop.value = cachedJSONObject.ocPop;
    naPop.value = cachedJSONObject.naPop;
    saPop.value = cachedJSONObject.saPop;
    years.value = cachedJSONObject.years;
  } else {
    console.log("dsa")
    const populationData = await axios.get('http://localhost:5000/population/asia');
    const asiaData = await axios.get('http://localhost:5000/population/asia');
    const euData = await axios.get('http://localhost:5000/population/europe');
    const africaData = await axios.get('http://localhost:5000/population/africa');
    const ocData = await axios.get('http://localhost:5000/population/oceania');
    const naData = await axios.get('http://localhost:5000/population/na');
    const saData = await axios.get('http://localhost:5000/population/sa');
    const yearsData = await axios.get('http://localhost:5000/population/years');

    population.value = populationData.data;
    asiaPop.value = asiaData.data;
    euPop.value = euData.data;
    africaPop.value = africaData.data;
    ocPop.value = ocData.data;
    naPop.value = naData.data;
    saPop.value = saData.data;
    years.value = yearsData.data;

    const cachingData = {
      population: populationData.data,
      asiaPop: asiaData.data,
      euPop: euData.data,
      africaPop: africaData.data,
      ocPop: ocData.data,
      naPop: naData.data,
      saPop: saData.data,
      years: yearsData.data
    };

    sessionStorage.setItem('cachedData', JSON.stringify(cachingData));
  }

  await initializeDisplayPopulation()
})

const togglePlay = async () => {
  isPlaying.value = !isPlaying.value;
  if (isPlaying.value) {
    await displayPopulation(currentIndex.value);
  }
}

function getRandomColor(text) {
  let seed = 0;
  for (let i = 0; i < text.length; i++) {
    seed += text.charCodeAt(i);
  }
  const randomColor = '#' + Math.floor(Math.abs(Math.sin(seed) * 14777215) % 16777215).toString(16);
  return randomColor;
}

const initializeDisplayPopulation = async () => {
  const firstYear = years.value[0]
  currentYear.value = firstYear
  totalPop.value = formatNumber(population.value[firstYear][0][1])
  chartState.value = { [firstYear]: population.value[firstYear] }
  computeNewChart(chartState, firstYear)
}

// bug that when user click reset button, its currentIndex get rewrite just after set to 0
const resetButton = async () => {
  isPlaying.value = false;
  currentIndex.value = 0;
  console.log("resetButton : " + currentIndex.value)
  initializeDisplayPopulation();
}

async function displayPopulation(startIndex) {
  for (let i = startIndex; i < years.value.length; i++) {
    currentIndex.value = i;
    if (!isPlaying.value) {
      break;
    }
    const year = years.value[i];
    currentYear.value = year
    totalPop.value = formatNumber(population.value[year][0][1])
    chartState.value = { [year]: population.value[year] };
    computeNewChart(chartState, year)
    await delay(200);
  }
  if (currentIndex.value === years.value.length - 1) {
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
  const newBackgroundColors = myChart.value.value.data.labels.map(label => getRandomColor(label));
  myChart.value.value.data.datasets[0].backgroundColor = newBackgroundColors;
  myChart.value.value.update();
}

async function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

function formatNumber(number) {
  return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

function nTopCountry() {
  currentYear.value = "2021"
  const asiaList = toggleAsia.value ? asiaPop.value.data[parseInt(currentYear.value)].slice(1, 11) : [];
  const euList = toggleEu.value ? euPop.value.data[parseInt(currentYear.value)].slice(1, 11) : [];
  const africaList = toggleAfrica.value ? africaPop.value.data[parseInt(currentYear.value)].slice(1, 11) : [];
  const ocList = toggleOc.value ? ocPop.value.data[parseInt(currentYear.value)].slice(1, 11) : [];
  const naList = toggleNa.value ? naPop.value.data[parseInt(currentYear.value)].slice(1, 11) : [];
  const saList = toggleSa.value ? saPop.value.data[parseInt(currentYear.value)].slice(1, 11) : [];

  const topNCountry = ref([]);

  const lists = [asiaList, euList, africaList, ocList, naList, saList];

  const n = 10;
  for (let i = 0; i < n; i++) {
    let maxPopulation = -Infinity;
    let maxIndex = -1;

    for (let j = 0; j < lists.length; j++) {
      const currentList = lists[j];
      if (currentList.length > 0 && currentList[0][1] > maxPopulation) {
        maxPopulation = currentList[0][1];
        maxIndex = j;
      }
    }

    if (maxIndex !== -1) {
      topNCountry.value.push(lists[maxIndex].shift());
    }
  }

  return topNCountry
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

.title {
  color: #87857e;
  text-align: center;
  margin-top: 40px;
}
</style>