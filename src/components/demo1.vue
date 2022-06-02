<script lang="ts" setup>
import { onMounted, ref, watch, withDefaults } from 'vue'
import * as d3 from 'd3'
interface Props {
  start: number
  end: number
}
const props = withDefaults(defineProps<Props>(), {
  start: 0,
  end: 100,
})
const emits = defineEmits<{
  (e: 'update:start', value: number): void
  (e: 'update:end', value: number): void
}>()
const startValue = ref(0)
const endValue = ref(100)
const min = 0
const max = 10
const demoRef = ref<HTMLDivElement | null>(null)
const height = 130 // 画布高度
const width = 460 // 画布宽度
const wxs = 0.85
const startPosition = (width - (width * wxs)) * 0.5 - 11 /* 指针开始位置 */
const draw = (el: HTMLDivElement) => {
  el.innerHTML = ''
  let rotate_current = 391 * (props.start / 100)
  let rotate_end = 391 * (props.end / 100) + (7 * 4) // 4代表指针之间的间距,end开始位置少了7个,所以4 * 7
  const getKeduListS = () => {
    const arr = []
    const step = (max - min) / 100
    for (let i = 1; i < 100; i++) {
      if (i % 10 === 0)
        continue
      arr.push(step * i + min)
    }

    return arr
  }
  const svg = d3
    .select(el)
    .append('svg')
    .attr('width', width)
    .attr('height', height)
    .attr('class', 'svg')
  const ruler = svg.append('g').attr('class', 'ruler')
  ruler.append('g').attr('class', 'rulerBox')
  ruler.append('g').attr('class', 'rulerKd').attr('transform', `translate(${(width - (width * wxs)) * 0.5},0)`)
  ruler.append('g').attr('class', 'rulerKds').attr('transform', `translate(${(width - (width * wxs)) * 0.5},0)`)
  ruler.append('g').attr('class', 'rulerNum').attr('transform', `translate(${(width - (width * wxs)) * 0.5},0)`)
  ruler.append('g').attr('class', 'rulerPointer').attr('transform', `translate(${rotate_current},0)`)
  ruler.append('g').attr('class', 'rulerPointerRight').attr('transform', `translate(${rotate_end},0)`)
  const rectScaleLine = d3
    .scaleLinear()
    .domain([min, max])
    .range([0, width * wxs])
  ruler
    .select('.rulerBox')
    .append('rect')
    .attr('width', width)
    .attr('height', height * 0.5)
    .attr('fill', '#F2F2F7')
    .attr('stroke', '#CECED6')
    .attr('stroke-width', 3)
  ruler
    .select('.rulerKd')
    .selectAll('line')
    .data(Array.from({ length: max + 1 }).map((_, i) => i))
    .enter()
    .append('line')
    .attr('x1', d => rectScaleLine(d))
    .attr('x2', d => rectScaleLine(d))
    .attr('y1', (height * 0.6) * 0.5 - 10)
    .attr('y2', 50)
    .attr('stroke', '#CECED6')
    .attr('stroke-width', 2)
  ruler
    .select('.rulerNum')
    .selectAll('text')
    .data(Array.from({ length: max + 1 }).map((_, i) => i))
    .enter()
    .append('text')
    .attr('x', d => rectScaleLine(d))
    .attr('y', (height * 0.6) * 0.5 - 25)
    .attr('text-anchor', 'middle')
    .attr('font-size', '16px')
    .attr('font-weight', 'bold')
    .attr('dy', '0.7em')
    .attr('fill', '#2C3556')
    .text(d => d)
  ruler
    .select('.rulerKds')
    .selectAll('line')
    .data(getKeduListS())
    .enter()
    .append('line')
    .attr('x1', d => rectScaleLine(d))
    .attr('x2', d => rectScaleLine(d))
    .attr('y1', (height * 0.6) * 0.5 - 10)
    .attr('y2', 45)
    .attr('stroke', '#000000')
    .attr('stroke-width', 1)
  const pathData: [number, number][] = [[startPosition, 70], [startPosition + 10, 55], [startPosition + 10, 29], [startPosition + 12, 29], [startPosition + 12, 70]]
  const lineGenerator = d3.line()
  ruler.select('.rulerPointer').append('path').attr('d', lineGenerator(pathData)).attr('fill', '#1890FF')
  ruler
    .select('.rulerPointer')
    .append('rect')
    .attr('x', 0)
    .attr('y', 70)
    .attr('width', 40)
    .attr('height', 60)
    .attr('rx', 5)
    .attr('fill', '#F5F6F9')
    .attr('stroke', '#cccccc')
    .attr('style', 'cursor:move')
    .call(d3.drag().on('drag', e => dragged(e, 'start')).on('end', e => dragEnd(e, 'start')) as any)
  ruler
    .select('.rulerPointer')
    .append('rect')
    .attr('x', 10)
    .attr('y', 100)
    .attr('width', 2)
    .attr('height', 20)
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')

  ruler
    .select('.rulerPointer')
    .append('rect')
    .attr('x', 20)
    .attr('y', 100)
    .attr('width', 3)
    .attr('height', 20)
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')
  ruler
    .select('.rulerPointer')
    .append('polygon')
    .attr('points', '8,113 5,110 8,107')
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')

  ruler
    .select('.rulerPointer')
    .append('polygon')
    .attr('points', '34,113, 34,107, 37,110')
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')
  ruler
    .select('.rulerPointer')
    .append('rect')
    .attr('x', 30)
    .attr('y', 100)
    .attr('width', 2)
    .attr('height', 20)
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')
  ruler
    .select('.rulerPointer')
    .append('rect')
    .attr('x', 4)
    .attr('y', 73)
    .attr('width', 33)
    .attr('height', 23)
    .attr('fill', '#FFF7E6')
    .attr('stroke', '#FA8C16 ')
    .attr('style', 'pointer-events: none;')
  ruler
    .select('.rulerPointer')
    .append('text')
    .attr('id', 'start_value')
    .attr('x', 20)
    .attr('y', 92)
    .attr('font-size', '20px')
    .attr('fill', '#FA8C16')
    .attr('text-anchor', 'middle')
    .attr('style', 'pointer-events: none;')
    .text(props.start)
    /* 右边 */
  const rightPathData: [number, number][] = [[startPosition - 6, 70], [startPosition - 16, 55], [startPosition - 16, 29], [startPosition - 18, 29], [startPosition - 18, 70]]
  ruler.select('.rulerPointerRight').append('path').attr('d', lineGenerator(rightPathData)).attr('fill', '#1890FF')
  ruler
    .select('.rulerPointerRight')
    .append('rect')
    .attr('x', 0)
    .attr('y', 70)
    .attr('width', 40)
    .attr('height', 60)
    .attr('rx', 5)
    .attr('fill', '#F5F6F9')
    .attr('stroke', '#cccccc')
    .attr('style', 'cursor:move')
    .call(d3.drag().on('drag', e => dragged(e, 'end')).on('end', e => dragEnd(e, 'end')) as any)
  ruler
    .select('.rulerPointerRight')
    .append('rect')
    .attr('x', 10)
    .attr('y', 100)
    .attr('width', 2)
    .attr('height', 20)
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')

  ruler
    .select('.rulerPointerRight')
    .append('rect')
    .attr('x', 20)
    .attr('y', 100)
    .attr('width', 3)
    .attr('height', 20)
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')
  ruler
    .select('.rulerPointerRight')
    .append('polygon')
    .attr('points', '8,113 5,110 8,107')
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')

  ruler
    .select('.rulerPointerRight')
    .append('polygon')
    .attr('points', '34,113, 34,107, 37,110')
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')
  ruler
    .select('.rulerPointerRight')
    .append('rect')
    .attr('x', 30)
    .attr('y', 100)
    .attr('width', 2)
    .attr('height', 20)
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')
  ruler
    .select('.rulerPointerRight')
    .append('rect')
    .attr('x', 4)
    .attr('y', 73)
    .attr('width', 33)
    .attr('height', 23)
    .attr('fill', '#F2F2F7')
    .attr('stroke', '#CECED6 ')
    .attr('style', 'pointer-events: none;')
  ruler
    .select('.rulerPointerRight')
    .append('text')
    .attr('id', 'end_value')
    .attr('text-anchor', 'middle')
    .attr('x', 20)
    .attr('y', 92)
    .attr('font-size', '20px')
    .attr('fill', '#2C3556')
    .attr('style', 'pointer-events: none;')
    .text(props.end)
  function dragged(event: DragEvent, type: 'start' | 'end') {
    const p = d3.pointer(event, svg.node())
    const [x, _y] = p // _代表没有使用解构的y值,防止ts警告
    const px = x - startPosition
    if (type === 'start') {
      startValue.value = parseInt(Math.round(((px / 391) * 100)).toString())
      rotate_current = px
      if (px < 0) {
        startValue.value = 0
        rotate_current = 0
      }
      if (px > 391) {
        startValue.value = 100
        rotate_current = 391
      }
      ruler.select('#start_value').text(startValue.value)
      ruler.select('.rulerPointer').attr('transform', `translate(${rotate_current},0)`)
    }
    else {
      endValue.value = parseInt(Math.round(((px / 391) * 100)).toString()) - 7
      rotate_end = px
      if (px <= 28.5) {
        endValue.value = 0
        rotate_end = 28.5
      }
      if (px > 419) {
        endValue.value = 100
        rotate_end = 419
      }
      ruler.select('#end_value').text(endValue.value)
      ruler.select('.rulerPointerRight').attr('transform', `translate(${rotate_end},0)`)
    }
  }

  function dragEnd(event: DragEvent, type: 'start' | 'end') {
    if (type === 'start')
      emits('update:start', startValue.value)
    else
      emits('update:end', endValue.value)
  }
}
watch(() => [props.start, props.end], () => {
  if (demoRef.value)
    draw(demoRef.value)
}, { immediate: true })
onMounted(() => {
  if (demoRef.value)
    draw(demoRef.value)
})
</script>

<template>
  <div ref="demoRef" />
</template>
