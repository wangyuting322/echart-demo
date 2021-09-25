<template>
  <div class="pie-3d-wrapper">
    <div class="pie-3d-chart" id="pie-3d-chart"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import 'echarts-gl'
export default {
  name: 'cityGreenLand',
  components: {},
  props: {
    // 数据
    pieData: {
      type: Array,
      default: () => []
    },
    // 圆弧中心的空心占比
    internalDiameterRatio: {
      type: Number,
      default: 0.5
    },
    // 自定义的配置
    customOption: {
      type: Object,
      default: () => ({})
    },
    // 3D饼图的最高高度
    Maxheight: {
      type: Number,
      default: 26
    }
  },
  data () {
    return {
      // 图表
      myChart: null,
      // 当前鼠标移至哪个位置（数字或''）
      hoveredIndex: ''
    }
  },
  computed: {
    // 数据总数 - 用于计算每个扇形的比例
    sumValue () {
      if (!this.pieData.length) return 0
      return this.pieData.reduce((pre, next) => {
        return pre + next.value
      }, 0)
    },
    // 对pieData进行数据格式化
    series () {
      let cacheSeries = []
      const sumValue = this.sumValue
      const pieData = this.pieData
      const internalDiameterRatio = this.internalDiameterRatio
      let startValue = 0
      let endValue = 0
      const k = 1 - internalDiameterRatio
      // 排序
      pieData.sort((a, b) => {
        return b.value - a.value
      })
      // 为每一个饼图数据，生成一个 series-surface 配置
      cacheSeries = pieData.map((item, index) => {
        const { name, itemStyle } = item
        return {
          name: name || `series${index}`,
          type: 'surface',
          parametric: true,
          wireframe: {
            show: false
          },
          pieData: item,
          pieStatus: {
            selected: false,
            hovered: false,
            k: k
          },
          center: ['10%', '50%'],
          itemStyle: {
            color: null,
            opacity: null,
            ...itemStyle
          }
        }
      })
      // 使用上一次遍历时，计算出的数据和 sumValue，调用 getParametricEquation 函数，
      // 向每个 series-surface 传入不同的参数方程 series-surface.parametricEquation，也就是实现每一个扇形。
      const seriesLen = cacheSeries.length
      for (let i = 0; i < seriesLen; i++) {
        endValue = startValue + cacheSeries[i].pieData.value
        cacheSeries[i].pieData.startRatio = startValue / sumValue
        cacheSeries[i].pieData.endRatio = endValue / sumValue
        cacheSeries[i].parametricEquation = this.getParametricEquation({
          startRatio: cacheSeries[i].pieData.startRatio,
          endRatio: cacheSeries[i].pieData.endRatio,
          isSelected: false,
          isHovered: false,
          k,
          value: cacheSeries[i].pieData.value
        })
        startValue = endValue
      }
      return cacheSeries
    },
    legendData () {
      const sumValue = this.sumValue
      return this.series.map(item => {
        return {
          name: item.name,
          value: this.fomatFloat(item.pieData.value / sumValue, 4)
        }
      })
    },
    // 3d饼图的高度
    boxHeight () {
      // 通过传参设定3d饼/环的最高高度，26代表26px
      return (this.Maxheight * 25) / this.series[0].pieData.value
    },
    option () {
      const legendData = this.legendData
      return {
        legend: {
          data: legendData,
          orient: 'horizontal',
          left: 10,
          top: 10,
          itemGap: 10,
          textStyle: {
            color: '#A1E2FF'
          },
          show: true,
          icon: 'circle',
          formatter: param => {
            const res = legendData.find(item => item.name === param)
            if (res) {
              const bfs = this.fomatFloat(res.value * 100, 2) + '%'
              return `${res.name}  ${bfs}`
            } else {
              return null
            }
          }
        },
        labelLine: {
          show: true,
          lineStyle: {
            color: '#7BC0CB'
          }
        },
        label: {
          show: true,
          position: 'outside',
          rich: {
            b: {
              color: '#7BC0CB',
              fontSize: 12,
              lineHeight: 20
            },
            c: {
              fontSize: 16
            }
          },
          formatter: '{b|{b} \n}{c|{c}}{b|  亩}'
        },
        tooltip: {
          formatter: params => {
            if (
              params.seriesName !== 'mouseoutSeries' &&
              params.seriesName !== 'pie2d'
            ) {
              const bfb = (
                (this.option.series[params.seriesIndex].pieData.endRatio -
                  this.option.series[params.seriesIndex].pieData.startRatio) *
                100
              ).toFixed(2)
              return (
                `${params.seriesName}<br/>` +
                `<span style="display:inline-block;margin-right:5px;border-radius:10px;width:10px;height:10px;background-color:${params.color};"></span>` +
                `<span>${bfb}%<span><br/>` +
                `<span>数量：${
                  this.series[params.seriesIndex].pieData.value
                }</span>`
              )
            }
          }
        },
        xAxis3D: {
          min: -1,
          max: 1
        },
        yAxis3D: {
          min: -1,
          max: 1
        },
        zAxis3D: {
          min: -1,
          max: 1
        },
        grid3D: {
          show: false,
          boxHeight: this.boxHeight, // 圆环的高度
          viewControl: {
            // 3d效果可以放大、旋转等，请自己去查看官方配置
            alpha: 40, // 角度
            distance: 300, // 调整视角到主体的距离，类似调整zoom
            rotateSensitivity: 1, // 设置为0无法旋转
            zoomSensitivity: 1, // 设置为0无法缩放
            panSensitivity: 0, // 设置为0无法平移
            autoRotate: false // 自动旋转
          }
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
    /**
     * 初始化
     */
    init () {
      // 构建3d饼状图
      this.myChart = echarts.init(document.getElementById('pie-3d-chart'))
      this.myChart.setOption(this.option)
      // 添加浏览器大小改变的监听
      window.addEventListener('resize', () => {
        this.myChart.resize()
      })
      // 添加图表的移入和移出事件监听
      this.bindListen(this.myChart)
    },
    /**
     * 生成扇形的曲面参数方程，用于 series-surface.parametricEquation
     */
    getParametricEquation ({
      startRatio,
      endRatio,
      isSelected,
      isHovered,
      k,
      value
    }) {
      // 计算
      const h = value
      const midRatio = (startRatio + endRatio) / 2
      const startRadian = startRatio * Math.PI * 2
      const endRadian = endRatio * Math.PI * 2
      const midRadian = midRatio * Math.PI * 2
      // 如果只有一个扇形，则不实现选中效果。
      if (startRatio === 0 && endRatio === 1) {
        isSelected = false
      }
      // 通过扇形内径/外径的值，换算出辅助参数 k（默认值 1/3）
      k = k || 1 / 3
      // 计算选中效果分别在 x 轴、y 轴方向上的位移（未选中，则位移均为 0）
      const offsetX = isSelected ? Math.cos(midRadian) * 0.1 : 0
      const offsetY = isSelected ? Math.sin(midRadian) * 0.1 : 0
      // 计算高亮效果的放大比例（未高亮，则比例为 1）
      const hoverRate = isHovered ? 1.05 : 1
      // 返回曲面参数方程
      return {
        u: {
          min: -Math.PI,
          max: Math.PI * 3,
          step: Math.PI / 32
        },
        v: {
          min: 0,
          max: Math.PI * 2,
          step: Math.PI / 20
        },
        x: (u, v) => {
          if (u < startRadian) {
            return (
              offsetX +
              Math.cos(startRadian) * (1 + Math.cos(v) * k) * hoverRate
            )
          }
          if (u > endRadian) {
            return (
              offsetX + Math.cos(endRadian) * (1 + Math.cos(v) * k) * hoverRate
            )
          }
          return offsetX + Math.cos(u) * (1 + Math.cos(v) * k) * hoverRate
        },
        y: (u, v) => {
          if (u < startRadian) {
            return (
              offsetY +
              Math.sin(startRadian) * (1 + Math.cos(v) * k) * hoverRate
            )
          }
          if (u > endRadian) {
            return (
              offsetY + Math.sin(endRadian) * (1 + Math.cos(v) * k) * hoverRate
            )
          }
          return offsetY + Math.sin(u) * (1 + Math.cos(v) * k) * hoverRate
        },
        z: (u, v) => {
          if (u < -Math.PI * 0.5) {
            return Math.sin(u)
          }
          if (u > Math.PI * 2.5) {
            return Math.sin(u) * h * 0.1
          }
          return Math.sin(v) > 0 ? 1 * h * 0.1 : -1
        }
      }
    },
    /**
     * 格式化数值（百分比后保留几位小数）
     */
    fomatFloat (num, n) {
      var f = parseFloat(num)
      if (isNaN(f)) {
        return false
      }
      return f.toFixed(n)
    },
    /**
     * 移入移出效果控制
     */
    setParametricEquation ({ index, ...args }) {
      const currentSeries = this.option.series[index]
      const { pieStatus, pieData } = currentSeries
      const { selected, hovered, k } = pieStatus
      const { startRatio, endRatio, value } = pieData
      const params = {
        startRatio,
        endRatio,
        isSelected: selected,
        isHovered: hovered,
        k,
        value,
        ...args
      }
      currentSeries.parametricEquation = this.getParametricEquation({
        ...params
      })
      pieStatus.hovered = params.isHovered
      pieStatus.selected = params.isSelected
    },
    /**
     * 添加移入和移出监听器
     */
    bindListen (myChart) {
      // 监听 mouseover，近似实现高亮（放大）效果
      myChart.on('mouseover', params => {
        if (!params) return
        // 准备重新渲染扇形所需的参数
        // 如果触发 mouseover 的扇形当前已高亮，则不做操作
        if (this.hoveredIndex === params.seriesIndex) {
          // 否则进行高亮及必要的取消高亮操作
        } else {
          // 如果当前有高亮的扇形，取消其高亮状态（对 option 更新）
          if (this.hoveredIndex !== '') {
            // 从 option.series 中读取重新渲染扇形所需的参数，将是否高亮设置为 false。
            // 对当前点击的扇形，执行取消高亮操作（对 option 更新）
            this.setParametricEquation({
              index: this.hoveredIndex,
              isHovered: false
            })
            // 将此前记录的上次选中的扇形对应的系列号 seriesIndex 清空
            this.hoveredIndex = ''
          }
          // 如果触发 mouseover 的扇形不是透明圆环，将其高亮（对 option 更新）
          if (
            params.seriesName !== 'mouseoutSeries' &&
            params.seriesName !== 'pie2d'
          ) {
            // 从 option.series 中读取重新渲染扇形所需的参数，将是否高亮设置为 true。
            const value = this.option.series[params.seriesIndex].pieData.value
            // 对当前点击的扇形，执行高亮操作（对 option 更新）
            this.setParametricEquation({
              index: params.seriesIndex,
              isHovered: true,
              value: value + 5
            })
            // 记录上次高亮的扇形对应的系列号 seriesIndex
            this.hoveredIndex = params.seriesIndex
          }
          // 使用更新后的 option，渲染图表
          myChart.setOption(this.option)
        }
      })
      // 修正取消高亮失败的 bug
      myChart.on('globalout', params => {
        if (!params) return
        // 准备重新渲染扇形所需的参数
        if (this.hoveredIndex !== '') {
          // 从 option.series 中读取重新渲染扇形所需的参数，将是否高亮设置为 true。
          // 对当前点击的扇形，执行取消高亮操作（对 option 更新）
          this.setParametricEquation({
            index: this.hoveredIndex,
            isHovered: false
          })
          // 将此前记录的上次选中的扇形对应的系列号 seriesIndex 清空
          this.hoveredIndex = ''
        }
        // 使用更新后的 option，渲染图表
        myChart.setOption(this.option)
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
.pie-3d-chart {
  height: 100%;
  width: 100%;
}
</style>
