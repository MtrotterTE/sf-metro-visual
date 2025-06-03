<template>
  <div class="welcome">
    <h1 class="page-title">Tenco CityScale K Line Visualization</h1>
    <div ref="chart" class="svg-area"></div>
  </div>  
</template>

<script setup>
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';

const chart = ref(null);

onMounted(() => {
  const width = 800;
  const height = 2400;

  const svg = d3.select(chart.value)
    .append('svg')
    .attr('width', width)
    .attr('height', height);

  svg.append('rect')
    .attr('x', 380)
    .attr('y', 0)
    .attr('width', 10)
    .attr('height', height)
    .attr('fill', 'steelblue');

  svg.append('rect')
    .attr('x', 420)
    .attr('y', 0)
    .attr('width', 10)
    .attr('height', height)
    .attr('fill', 'steelblue');
})
</script>

<style scoped>
.page-title {
  text-align: center;
  font-size: 2rem;
  color: #C3BFBA;
  text-decoration: underline;
}

.svg-area {
  display: flex;
  justify-content: center;
  align-items: center;
  height: fit-content;
  border: 1px solid #ccc;
}
</style>