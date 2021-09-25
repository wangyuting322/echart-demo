<template>
  <div class="pie-3d-wrapper">
    <div class="line-chart" id="line-chart"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
export default {
  name: 'cityGreenLand',
  components: {},
  props: {
    // 数据
    lineData: {
      type: Array,
      default: () => []
    },
    // x轴数据
    xData: {
      type: Array,
      default: () => []
    },
    // 自定义的配置
    customOption: {
      type: Object,
      default: () => ({})
    }
  },
  data () {
    return {
      // 图表
      myChart: null
    }
  },
  computed: {
    // 对lineData进行数据格式化
    series () {
      const cacheSeries = this.lineData
      return cacheSeries
    },
    option () {
      return {
        // title: {
        //   text: 'echarts3的折线图分段显示不同的颜色',
        //   left: 'center',
        //   link: 'http://phping.sinaapp.com'
        // },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c}'
        },
        legend: {
          left: 'left',
          data: ['指数']
        },
        xAxis: {
          type: 'category',
          name: 'x',
          splitLine: { show: false },
          data: this.xData
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        yAxis: {
          type: 'log',
          name: 'y',
          splitNumber: 60,
          max: 120,
          min: 0
        },
        series: this.series,
        ...this.customOption
      }
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    init () {
      // 构建3d饼状图
      this.myChart = echarts.init(document.getElementById('line-chart'))
      this.myChart.setOption(this.option)
      // 添加浏览器大小改变的监听
      window.addEventListener('resize', () => {
        this.myChart.resize()
      })
    },
    /**
     * 清空图表内存
     */
    disposeChart () {
      this.myChart && this.myChart.dispose()
    }
  },
  beforeDestory () {
    this.disposeChart()
  }
}
</script>
<style lang="scss" scoped>
.pie-3d-wrapper {
  /* width: 100vw;
  height: 100vh; */
}
.line-chart {
  height: 100%;
  width: 100%;
}
</style>
