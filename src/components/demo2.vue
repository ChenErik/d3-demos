<script>
import { onMounted, ref, watch } from 'vue'
import * as d3 from 'd3'

export default {
  name: 'HmiDial',
  props: {
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 160,
    },
    standard: {
      type: Number,
      default: 100,
    },
    text: {
      type: String,
      default: '',
    },
    modelValue: {
      type: Number,
      default: 0,
    },
  },
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    const dial = ref(null)

    let keduListB = [] // （大）刻度的数组
    let keduListS = [] // （小）刻度的数组

    const getKeduListB = () => {
      const arr = []
      const step = (props.max - props.min) / 8
      for (let i = 0; i < 9; i++)
        arr.push(step * i + props.min)

      return arr
    }

    const getKeduListS = () => {
      const arr = []
      const step = (props.max - props.min) / 40
      for (let i = 1; i < 40; i++) {
        if (i % 5 === 0)
          continue
        arr.push(step * i + props.min)
      }

      return arr
    }

    const height = 230 // 画布高度
    const width = 280 // 画布宽度
    const outerRadius = 106 // 仪表盘外半径
    const innerRadius = 64 // 仪表盘内半径
    const arc = d3.arc().innerRadius(innerRadius).outerRadius(outerRadius) // 编写弧生成器并传入内外半径

    const radiusScaleB = d3
      .scaleLinear()
      .domain([0, 1])
      .range([Math.PI * (-128 / 180), (Math.PI * 128) / 180]) // 表盘线性比例尺
    let radiusScaleS = null // 刻度线性比例尺
    let rotate_standard = null
    let rotate_current = null
    let value_current = null
    const draw = (el) => {
      el.innerHTML = null

      const svg = d3
        .select(el)
        .append('svg')
        .attr('width', width)
        .attr('height', height)
        .attr('class', 'svg')

      const group = svg
        .append('g')
        .attr('class', 'group')
        .attr('transform', `translate(${width * 0.5}, ${height * 0.6})`) // 添加分组并将分组中心移至画布中心

      group.append('g').attr('class', 'pan') // 绘制仪表盘的圆弧分组
      group.append('g').attr('class', 'kedu_b') // 绘制大刻度分组
      group.append('g').attr('class', 'kedu_s') // 绘制小刻度分组

      if (rotate_standard) {
        group
          .append('g')
          .attr('class', 'zhizhen_s')
          .attr('transform', `rotate(${rotate_standard})`) // 绘制标准指针分组
          .append('polygon')
          .attr('points', '-8,62,8,62,0,80,0,80')
          .attr('fill', '#4A84FF')
      }

      group
        .append('g')
        .attr('class', 'zhizhen')
        .attr('transform', `rotate(${rotate_current})`) // 绘制指针分组
        .append('polygon')
        .attr('points', '-5,108,5,108,0,94,0,94')
        .attr('fill', '#000000')

      group.append('g').attr('class', 'shuzi') // 绘制刻度数字分组
      group.append('g').attr('class', 'show') // 绘制速度框和数字分组
      group
        .append('g')
        .attr('class', 'button')
        .attr('transform', `rotate(${rotate_current})`) // 绘制按钮框和数字分组

      group
        .select('.pan')
        .append('path')
        .attr(
          'd',
          arc({ startAngle: radiusScaleB(0), endAngle: radiusScaleB(1) }),
        )
        .attr('fill', '#f2f2f2')
        .attr('stroke', '#cccccc')
        .attr('stroke-width', 3)

      group
        .select('.kedu_b')
        .selectAll('line')
        .data(keduListB)
        .enter()
        .append('line')
        .attr('x1', d => 68 * Math.sin(radiusScaleS(d)))
        .attr('y1', d => 68 * Math.cos(radiusScaleS(d)) * -1)
        .attr('x2', d => 82 * Math.sin(radiusScaleS(d)))
        .attr('y2', d => 82 * Math.cos(radiusScaleS(d)) * -1)
        .attr('stroke', '#000000')
        .attr('stroke-width', 2)

      group
        .select('.kedu_s')
        .selectAll('line')
        .data(keduListS)
        .enter()
        .append('line')
        .attr('x1', d => 68 * Math.sin(radiusScaleS(d)))
        .attr('y1', d => 68 * Math.cos(radiusScaleS(d)) * -1)
        .attr('x2', d => 77 * Math.sin(radiusScaleS(d)))
        .attr('y2', d => 77 * Math.cos(radiusScaleS(d)) * -1)
        .attr('stroke', '#000000')
        .attr('stroke-width', 1)

      group
        .select('.shuzi')
        .selectAll('text')
        .data(keduListB)
        .enter()
        .append('text')
        .attr('x', d => 94 * Math.sin(radiusScaleS(d)))
        .attr('y', d => 94 * Math.cos(radiusScaleS(d)) * -1)
        .attr('text-anchor', 'middle')
        .attr('font-size', '10px')
        .attr('dy', '0.7em')
        .attr('fill', '#000000')
        .text(d => d)

      function dragged(event) {
        const p = d3.pointer(event, svg.node())

        const x = p[0] - width * 0.5
        const y = p[1] - height * 0.6

        let a = (Math.atan2(y, x) / Math.PI) * 180 + 270
        while (a > 360) a -= 360

        if (a < 60)
          a = 60
        if (a > 300)
          a = 300

        group.selectAll('.zhizhen').attr('transform', `rotate(${a})`)
        group.selectAll('.button').attr('transform', `rotate(${a})`)

        value_current = Math.floor(
          ((a - 60) / 240) * (props.max - props.min) + props.min,
        )
        group.select('#speed-val').text(value_current)
      }

      function dragEnd(event) {
        emit('update:modelValue', value_current)
      }

      group
        .select('.button')
        .append('rect')
        .attr('x', -24)
        .attr('y', 107)
        .attr('width', 48)
        .attr('height', 30)
        .attr('rx', 5)
        .attr('fill', '#f2f2f2')
        .attr('stroke', '#cccccc')
        .call(d3.drag().on('drag', dragged).on('end', dragEnd))

      group
        .select('.button')
        .append('rect')
        .attr('x', -1)
        .attr('y', 112)
        .attr('width', 2)
        .attr('height', 20)
        .attr('fill', '#000000')
        .attr('style', 'pointer-events: none;')

      group
        .select('.button')
        .append('rect')
        .attr('x', -9)
        .attr('y', 112)
        .attr('width', 3)
        .attr('height', 20)
        .attr('fill', '#000000')
        .attr('style', 'pointer-events: none;')

      group
        .select('.button')
        .append('polygon')
        .attr('points', '-13,118,-13,124,-17,121')
        .attr('fill', '#000000')
        .attr('style', 'pointer-events: none;')

      group
        .select('.button')
        .append('polygon')
        .attr('points', '13,118,13,124,17,121')
        .attr('fill', '#000000')
        .attr('style', 'pointer-events: none;')

      group
        .select('.button')
        .append('rect')
        .attr('x', 6)
        .attr('y', 112)
        .attr('width', 3)
        .attr('height', 20)
        .attr('fill', '#000000')
        .attr('style', 'pointer-events: none;')

      group
        .select('.show')
        .append('line')
        .attr('x1', 3)
        .attr('y1', -8)
        .attr('x2', -3)
        .attr('y2', 18)
        .attr('stroke', '#000000')
        .attr('stroke-width', 2)

      group
        .select('.show')
        .append('rect')
        .attr('x', -56)
        .attr('y', -10)
        .attr('width', 48)
        .attr('height', 30)
        .attr('fill', '#f2f2f2')
        .attr('stroke', '#cccccc')
        .attr('stroke-width', 1)

      group
        .select('.show')
        .append('rect')
        .attr('x', 6)
        .attr('y', -10)
        .attr('width', 48)
        .attr('height', 30)
        .attr('fill', '#f2f2f2')
        .attr('stroke', '#cccccc')
        .attr('stroke-width', 1)

      group
        .select('.show')
        .append('text')
        .attr('text-anchor', 'middle')
        .attr('x', -31)
        .attr('y', -20)
        .attr('dy', '0.35em')
        .attr('font-size', '12px')
        .text('实际值')

      group
        .select('.show')
        .append('text')
        .attr('text-anchor', 'middle')
        .attr('x', 29)
        .attr('y', -20)
        .attr('dy', '0.35em')
        .attr('font-size', '12px')
        .text('标准值')

      group
        .select('.show')
        .append('text')
        .attr('id', 'speed-val')
        .attr('text-anchor', 'middle')
        .attr('x', -31)
        .attr('y', -10)
        .attr('dy', '23px')
        .attr('font-size', '23px')
        .attr('fill', '#4A84FF')
        .text(value_current)

      group
        .select('.show')
        .append('text')
        .attr('text-anchor', 'middle')
        .attr('x', 29)
        .attr('y', -10)
        .attr('dy', '23px')
        .attr('font-size', '23px')
        .text(props.standard)

      group
        .select('.show')
        .append('text')
        .attr('text-anchor', 'middle')
        .attr('x', 0)
        .attr('y', 50)
        .attr('dy', '23px')
        .attr('font-size', '16px')
        .text(props.text)
    }
    watch(
      () => [
        props.min,
        props.max,
        props.standard,
        props.text,
        props.modelValue,
      ],
      () => {
        keduListB = getKeduListB()
        keduListS = getKeduListS()
        radiusScaleS = d3
          .scaleLinear()
          .domain([props.min, props.max])
          .range([Math.PI * (-2 / 3), Math.PI * (2 / 3)])

        if (props.standard >= props.min && props.standard <= props.max) {
          rotate_standard
            = ((props.standard - props.min) / (props.max - props.min)) * 240 + 60
        }
        else {
          rotate_standard = null
        }

        if (props.modelValue >= props.min && props.modelValue <= props.max) {
          rotate_current
            = ((props.modelValue - props.min) / (props.max - props.min)) * 240
            + 60
          value_current = props.modelValue
        }
        else {
          rotate_current = 60
          value_current = props.min
        }

        if (dial.value)
          draw(dial.value)
      },
      {
        immediate: true,
      },
    )

    onMounted(() => {
      if (dial.value)
        draw(dial.value)
    })

    return {
      dial,
    }
  },
}
</script>

<template>
  <div ref="dial" />
</template>
