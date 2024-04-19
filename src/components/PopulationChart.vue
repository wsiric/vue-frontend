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
  datasets: [
    {
      label: 'All',
    },
    {
      label: 'Asia',
    },
    {
      label: 'Europe',
    },
    {
      label: 'Africa',
    },
    {
      label: 'North America',
    },
    {
      label: 'South America',
    },
    {
      label: 'Oceania',
    },
  ]
}

const config = {
  type: 'bar',
  data: chartData,
  options: {
    indexAxis: 'y',
    plugins: {
      legend: {
        onClick: legendClickHandler,
      },
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
    },
    datasets: {
      bar: {
        barPercentage: 20,
        categoryPercentage: 0.1
      }
    }
  },
  plugins: [ChartDataLabels]
}

const nTop = 10
const currentYear = ref(null)
const totalPop = ref(null)
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
    asiaPop.value = cachedJSONObject.asiaPop;
    euPop.value = cachedJSONObject.euPop;
    africaPop.value = cachedJSONObject.africaPop;
    ocPop.value = cachedJSONObject.ocPop;
    naPop.value = cachedJSONObject.naPop;
    saPop.value = cachedJSONObject.saPop;
    years.value = cachedJSONObject.years;
  } else {
    const url = "https://sgn-service-5voezw3rqq-as.a.run.app/"
    const asiaData = await axios.get(`${url}/population/asia`);
    const euData = await axios.get(`${url}/population/europe`);
    const africaData = await axios.get(`${url}/population/africa`);
    const ocData = await axios.get(`${url}/population/oceania`);
    const naData = await axios.get(`${url}/population/na`);
    const saData = await axios.get(`${url}/population/sa`);
    const yearsData = await axios.get(`${url}/population/years`);

    asiaPop.value = asiaData.data;
    euPop.value = euData.data;
    africaPop.value = africaData.data;
    ocPop.value = ocData.data;
    naPop.value = naData.data;
    saPop.value = saData.data;
    years.value = yearsData.data;

    sessionStorage.setItem('cachedData', JSON.stringify({
      asiaPop: asiaData.data,
      euPop: euData.data,
      africaPop: africaData.data,
      ocPop: ocData.data,
      naPop: naData.data,
      saPop: saData.data,
      years: yearsData.data
    }));
  }
  await initializeData()
})

const togglePlay = async () => {
  isPlaying.value = !isPlaying.value;
  if (isPlaying.value) {
    await displayPopulation(currentIndex.value);
  }
}

const initializeData = async () => {
  const firstYear = years.value[0]
  currentYear.value = firstYear
  updatedChart(firstYear)
}

const reset = ref(false)
const resetButton = async () => {
  reset.value = true
  isPlaying.value = false;
  initializeData();
}

async function displayPopulation(startIndex) {
  for (let i = startIndex; i < years.value.length; i++) {
    currentIndex.value = i;
    if (!isPlaying.value) {
      if (reset.value === true) {
        currentIndex.value = 0;
        reset.value = false
      }
      break;
    }
    const year = years.value[i];
    currentYear.value = year
    updatedChart(year)
    await delay(200);
  }
  if (currentIndex.value === years.value.length - 1) {
    isPlaying.value = false;
    currentIndex.value = 0;
  }
}

function updatedChart(year) {
  totalPop.value = formatNumber(computeTotalPop(year))
  chartState.value = computeNTopPopCountry(year, nTop).value
  const countryNames = chartState.value.map(item => item[0]);
  const countryPopulation = chartState.value.map(item => item[1]);
  const barColor = chartState.value.map(item => item[2])
  changeChartData(countryNames, countryPopulation, barColor)
}

function changeChartData(label, newData, barColor) {
  myChart.value.value.data.labels = label
  myChart.value.value.data.datasets[0].data = newData
  myChart.value.value.data.datasets[0].backgroundColor = barColor;
  myChart.value.value.update();
}

function computeTotalPop(year) {
  const asiaTotalPop = toggleAsia.value ? asiaPop.value[year][0][1] : 0
  const euTotalPop = toggleEu.value ? euPop.value[year][0][1] : 0
  const africaTotalPop = toggleAfrica.value ? africaPop.value[year][0][1] : 0
  const ocTotalPop = toggleOc.value ? ocPop.value[year][0][1] : 0
  const naTotalPop = toggleNa.value ? naPop.value[year][0][1] : 0
  const saTotalPop = toggleSa.value ? saPop.value[year][0][1] : 0

  return asiaTotalPop + euTotalPop + africaTotalPop + ocTotalPop + naTotalPop + saTotalPop
}


function computeNTopPopCountry(year, n) {
  const asiaList = toggleAsia.value ? asiaPop.value[year].slice(1, 11) : [];
  const euList = toggleEu.value ? euPop.value[year].slice(1, 11) : [];
  const africaList = toggleAfrica.value ? africaPop.value[year].slice(1, 11) : [];
  const naList = toggleNa.value ? naPop.value[year].slice(1, 11) : [];
  const saList = toggleSa.value ? saPop.value[year].slice(1, 11) : [];
  const ocList = toggleOc.value ? ocPop.value[year].slice(1, 11) : [];

  const topNCountry = ref([]);

  const lists = [asiaList, euList, africaList, naList, saList, ocList];

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
    // asia 'rgba(255, 99, 132, 0.5)'
    // eu 'rgba(255, 159, 64, 0.5)'
    // afr 'rgba(255, 205, 86, 0.5)'
    // na 'rgba(75, 192, 192, 0.5)'
    // sa 'rgba(153, 102, 255, 0.5)'
    // oc 'rgba(201, 203, 207, 0.5)'
    if (maxIndex !== -1) {
      const country = lists[maxIndex].shift()
      switch (maxIndex) {
        case 0:
          country.push('rgba(255, 99, 132, 0.5)')
          break;
        case 1:
          country.push('rgba(255, 159, 64, 0.5)')
          break;
        case 2:
          country.push('rgba(255, 205, 86, 0.5)')
          break;
        case 3:
          country.push('rgba(75, 192, 192, 0.5)')
          break;
        case 4:
          country.push('rgba(153, 102, 255, 0.5)')
          break;
        case 5:
          country.push('rgba(201, 203, 207, 0.5)')
          break;
      }
      topNCountry.value.push(country);
    }
  }

  return topNCountry
}

function legendClickHandler(e, legendItem) {
  const index = legendItem.datasetIndex
  const ci = myChart.value.value
  const meta = ci.getDatasetMeta(index)

  meta.hidden = meta.hidden === null ? !ci.data.datasets[index].hidden : null;


  const name = legendItem.text;
  if (name === 'Asia') {
    toggleAsia.value = !toggleAsia.value
  } else if (name === 'Europe') {
    toggleEu.value = !toggleEu.value
  } else if (name === 'Africa') {
    toggleAfrica.value = !toggleAfrica.value
  } else if (name === 'North America') {
    toggleNa.value = !toggleNa.value
  } else if (name === 'South America') {
    toggleSa.value = !toggleSa.value
  } else if (name === 'Oceania') {
    toggleOc.value = !toggleOc.value
  }

  updatedChart(currentYear.value)
}

// ===================== HELPER ==============================

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

.title {
  color: #87857e;
  text-align: center;
  margin-top: 40px;
}
</style>