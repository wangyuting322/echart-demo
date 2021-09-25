<template>
  <div class="about">
    <div class="wrapper">
      <div class="co">
        <Pie3d
          :pieData="pieData"
          class="my-pie"
          :internalDiameterRatio="internalDiameterRatio"
        ></Pie3d>
      </div>
      <div  class="co">
        <LineChart
          class="my-pie"
          :lineData="lineData"
          :xData="xData"
        ></LineChart>
      </div>
    </div>
  </div>
</template>
<script>
import * as echarts from 'echarts'
import Pie3d from '@/components/PieChart3d.vue'
import LineChart from '@/components/LineChart.vue'
export default {
  name: 'about',
  components: {
    Pie3d,
    LineChart
  },
  data () {
    return {
      pieData: [
        {
          name: '林地面积统计',
          value: 10000,
          itemStyle: {
            color: '#22c4ff'
          }
        },
        {
          name: '草地面积统计',
          value: 12116,
          itemStyle: {
            color: '#aaff00'
          }
        },
        {
          name: '耕地地面积统计',
          value: 16616,
          itemStyle: {
            color: '#ffaaff'
          }
        }
      ],
      internalDiameterRatio: 0.5,
      lineData: [
        {
          name: '指数',
          type: 'line',
          data: [4, 8, 16, 32, 64, '-', '-', '-', '-'],
          areaStyle: {
            opacity: 0.8,
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              {
                offset: 0,
                color: 'rgba(128, 255, 165)'
              },
              {
                offset: 1,
                color: 'rgba(1, 191, 236)'
              }
            ])
          }
        },
        {
          name: '指数',
          type: 'line',
          data: ['-', '-', '-', '-', 64, 78, 98, '-', '-'],
          areaStyle: {
            opacity: 0.8,
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              {
                offset: 0,
                color: 'rgba(225, 120, 165)'
              },
              {
                offset: 1,
                color: 'rgba(134, 191, 223)'
              }
            ])
          }
        },
        {
          name: '指数',
          type: 'line',
          data: ['-', '-', '-', '-', '-', '-', 98, 64, 45],
          itemStyle: {
            normal: {
              lineStyle: {
                width: 2,
                type: 'dotted' // 'dotted'虚线 'solid'实线
              }
            }
          }
        }
      ],
      xData: [
        '1',
        '2',
        '3',
        '4',
        '5',
        '6',
        '7', '8', '9'
      ]
    }
  }
}
</script>
<style lang="scss">
.wrapper {
  display: flex;
  height: 60vh;
  width: 100%;
}
.co{
  flex-grow: 1;
  height: 100%;
}
.my-pie {
 height: 100%;
}
</style>
