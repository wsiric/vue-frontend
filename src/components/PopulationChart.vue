<template>
  <div>
    <ul>
      <li v-for="(value, key) in population.data" :key="key">
        Key: {{ key }} - Value: {{ value.slice(0, 10) }}
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


onBeforeMount(async () => {
  population.value = await axios.get('http://localhost:5000/population/all');
  years.value = await axios.get('http://localhost:5000/population/years')
})


</script>