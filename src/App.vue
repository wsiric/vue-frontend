<template>
  <v-app>
    <v-main>
      <PopulationChart/>
    </v-main>
  </v-app>
</template>

<script>
import PopulationChart from './components/PopulationChart.vue'

export default {
  name: 'App',

  components: {
    PopulationChart,
  },

  data: () => ({
    //
  }),
}
</script>
