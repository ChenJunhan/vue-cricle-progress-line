/** 
import progressLine from 'components/attendance/progressLine'
components: {
  progressLine
},
<progress-line :size="size" :value="stuData.value" :rate="stuData.rate" layerColor="#FFA900" :color="txNum === 1 ? '#2876F8' : '#00C3AC'" class="progress-line"></progress-line>
*/
<template>
  <div class="circle-progress" :style="{width: `${size + strokeWidth}px`, height: `${size + strokeWidth}px`}">
    <canvas id="canvas" :width="size * ratio" :height="size * ratio" :style="{width: `${size}px`, height: `${size}px`}"></canvas>
  </div>
</template>

<script>
export default {
  data () {
    return {
      canvas: '',
      ctx: '',
      circleX: '', // 圆心x坐标
      circleY: '', // 圆心y坐标
      radius: '', // 半径
      process: '', // 进度
      circleLoading: null,
      ratio: 0
    }
  },
  props: {
    value: { // 当前进度
      type: Number,
      default: 0
    },
    rate: { // 目标进度
      type: Number,
      default: 100
    },
    size: { // 圆环直径
      type: Number,
      default: () => 100
    },
    color: { // 进度条颜色
      type: String,
      default: '#1989fa'
    },
    layerColor: { // 轨道颜色
      type: String,
      default: '#eee'
    },
    strokeWidth: { // 进度条宽度
      type: Number,
      default: 10
    }
  },
  watch: {
    value: {
      handler (val, oldVal) {
        this.toCanvas('canvas')
      },
      deep: true
    },
    rate: {
      handler (val, oldVal) {
        this.toCanvas('canvas')
      },
      deep: true
    },
    size: {
      handler (val, oldVal) {
        this.toCanvas('canvas')
      },
      deep: true
    }
  },
  mounted () {
  },
  methods: {
    whiteBorder (cx, cy, r, anti) { // 弧线尾部白边
      let startAngle = 0.86 * Math.PI + (anti / this.rate) * (1.3 * Math.PI)
      let endAngle = 0.88 * Math.PI + (anti / this.rate) * (1.3 * Math.PI)
      this.ctx.beginPath()
      this.ctx.lineWidth = this.strokeWidth * this.ratio
      this.ctx.strokeStyle = this.createLinearObj(startAngle, endAngle) // 颜色
      this.ctx.lineCap = 'butt' // 圆弧两端的样式
      this.ctx.arc(cx * this.ratio, cy * this.ratio, r * this.ratio, startAngle, endAngle, false)
      this.ctx.stroke()
    },
    createLinearObj (angle1, angle2) { // 创建渐变对象  计算渐变开始点和渐变结束点
      let x1 = (this.size / 2 + this.radius * Math.cos(angle1)) * this.ratio
      let y1 = (this.size / 2 + this.radius * Math.sin(angle1)) * this.ratio
      let x2 = (this.size / 2 + this.radius * Math.cos(angle2)) * this.ratio
      let y2 = (this.size / 2 + this.radius * Math.sin(angle2)) * this.ratio
      let g = this.ctx.createLinearGradient(x1, y1, x2, y2)
      g.addColorStop(0, '#fff') // 添加颜色点
      g.addColorStop(0.5, '#fff')
      g.addColorStop(0.5, this.layerColor)
      g.addColorStop(1, this.layerColor)
      return g
    },
    circle (cx, cy, r) { // 画目标进度圆
      this.ctx.beginPath()
      this.ctx.lineWidth = this.strokeWidth * this.ratio
      this.ctx.strokeStyle = this.layerColor
      this.ctx.lineCap = 'round' // 半圆两端的样式
      this.ctx.arc(cx * this.ratio, cy * this.ratio, r * this.ratio, 0.85 * Math.PI, 0.15 * Math.PI)
      this.ctx.stroke()
    },
    sector (cx, cy, r, startAngle, endAngle, anti) { // 画当前进度圆弧
      this.ctx.beginPath()
      // ctx.moveTo(cx, cy + r) // 从圆形底部开始画
      this.ctx.lineWidth = this.strokeWidth * this.ratio
      this.ctx.strokeStyle = this.color // 进度条颜色
      this.ctx.lineCap = 'round' // 圆弧两端的样式
      this.ctx.arc(cx * this.ratio, cy * this.ratio, r * this.ratio, startAngle, startAngle + (anti / this.rate) * (endAngle - startAngle), false) // 圆弧
      this.ctx.stroke()
    },
    loading (progress) { // 刷新
      // 当前进度若超过目标进度，限制进度条不得超过目标进度
      if (this.process >= this.value || this.process >= this.rate) {
        clearInterval(this.circleLoading)
      }
      this.ctx.clearRect(0, 0, this.circleX * 2 * this.ratio, this.circleY * 2 * this.ratio) // 清除canvas内容
      this.circle(this.circleX, this.circleY, this.radius) // 目标进度圆环
      this.sector(this.circleX, this.circleY, this.radius, 0.85 * Math.PI, 2.15 * Math.PI, this.process) // 当前进度圆弧
      if (this.value && this.value < this.rate) { // 当前进度为0或者等于目标进度不显示白边
        this.whiteBorder(this.circleX, this.circleY, this.radius, this.process)
      }
      // 控制结束时动画的速度
      if (this.process / this.value > 0.9 * this.rate) {
        this.process += this.rate * 0.003
      } else if (this.process / this.value > 0.8 * this.rate) {
        this.process += this.rate * 0.01
      } else if (this.process / this.value > 0.7 * this.rate) {
        this.process += this.rate * 0.02
      } else {
        this.process += this.rate * 0.03
      }
      if (this.process > this.rate) this.process = this.rate
    },
    toCanvas (id) { // 生成环形进度条
      clearInterval(this.circleLoading)
      this.canvas = document.getElementById(id)
      this.ctx = this.canvas.getContext('2d')
      this.ratio = this.getPixelRatio(this.ctx)
      this.circleX = this.size / 2
      this.circleY = this.size / 2
      this.radius = this.size / 2 - this.strokeWidth // 圆环半径
      this.process = 0.0
      this.circleLoading = window.setInterval(() => {
        this.loading()
      }, 20)
    },
    getPixelRatio (context) { // 获取屏幕像素比
      let backingStore = context.backingStorePixelRatio ||
        context.webkitBackingStorePixelRatio ||
        context.mozBackingStorePixelRatio ||
        context.msBackingStorePixelRatio ||
        context.oBackingStorePixelRatio ||
        context.backingStorePixelRatio || 1
      return (window.devicePixelRatio || 1) / backingStore
    }
  }
}
</script>
<style lang="less">
.circle-progress {
  display: inline-block;
}
</style>
