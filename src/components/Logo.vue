<script setup lang="ts">
import * as d3 from 'd3'
import { onMounted, ref } from 'vue'
const logoRef = ref<HTMLDivElement | null>(null)
const width = 170
const height = 354
const rectWidth = width - 34
const rectHeight = 34
function draw(el: HTMLDivElement) {
  el.innerHTML = ''
  const svg = d3
    .select(el)
    .append('svg')
    .attr('width', width)
    .attr('height', height)
    .attr('class', 'logo')
  svg
    .append('g')
    .attr('class', 'container')
  for (let i = 0; i < 6; i++) {
    svg
      .select('.container')
      .append('rect')
      .attr('width', i === 1 ? width : rectWidth)
      .attr('height', rectHeight)
      .attr('fill', '#252B3D ')
      .attr('opacity', 0.4)
      .attr('x', i === 1 ? 0 : 34)
      .attr('y', (rectHeight + 30) * i)
  }
}
onMounted(() => {
  draw(logoRef.value!)
})
</script>

<template>
  <div ref="logoRef" />
</template>
