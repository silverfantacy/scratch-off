<template>
  <div style="position:relative; width: 300px;height: 300px;" class="scratch-box">
    <div class="secret-container">
      <template v-if="showContent">
        <slot name="secret">secret</slot>
      </template>
    </div>
    <canvas class="scratch" ref="scratch"/>
  </div>
  <button :disabled="!showContent" style="margin-top: 20px;" @click="reset()">重新刮</button>
</template>

<script>
import brushImage from '../assets/brush.png'
import coverImage from '../assets/cover.png'

export default {
  name: 'scratchOff',
  components: {},

  mounted: function () {
    this.$nextTick(() => {
      this.init()
    })
  },

  data () {
    return {
      isDrawing: false,
      lastPoint: null,
      ctx: null,
      showContent: false,
      showAlert: true,
      distCount: 0,
    }
  },

  methods: {
    init () {
      this.showContent = false
      const canvas = this.$refs.scratch

      canvas.width = canvas.parentElement.offsetWidth
      canvas.height = canvas.parentElement.offsetHeight
      console.log(canvas.width, canvas.height)

      canvas.addEventListener('mousedown', this.touchStart)
      canvas.addEventListener('touchstart', this.touchStart)
      canvas.addEventListener('mousemove', this.touchMove)
      canvas.addEventListener('touchmove', this.touchMove)
      canvas.addEventListener('mouseup', this.touchEnd)
      canvas.addEventListener('touchend', this.touchEnd)

      this.ctx = canvas.getContext('2d')

      this.brush = new Image()
      this.brush.src = brushImage

      this.cover = new Image()
      this.cover.src = coverImage
      this.cover.onload = () => {
        this.ctx.drawImage(this.cover, 0, 0, canvas.width, canvas.height)

        // 避免內容比圖片早顯示
        this.showContent = true
      }
    },

    getPosition (event) {
      let target = this.$refs.scratch
      let offsetX = 0
      let offsetY = 0

      if (target.offsetParent !== undefined) {
        while ((target = target.offsetParent)) {
          offsetX += target.offsetLeft
          offsetY += target.offsetTop
        }
      }

      const x = (event.pageX || event.touches[0].clientX) - offsetX
      const y = (event.pageY || event.touches[0].clientY) - offsetY
      return { x, y }
    },

    touchStart (event) {
      console.log(event.target)
      this.isDrawing = true
      this.lastPoint = this.getPosition(event)
      this.ctx.globalCompositeOperation = 'destination-out'
    },

    touchMove (event) {
      if (!this.isDrawing) return
      event.preventDefault()

      const ctx = this.ctx
      const a = this.lastPoint
      const b = this.getPosition(event)
      const dist = Math.sqrt(Math.pow(b.x - a.x, 2) + Math.pow(b.y - a.y, 2))
      const angle = Math.atan2(b.x - a.x, b.y - a.y)
      const offsetX = this.brush.width / 2
      const offsetY = this.brush.height / 2

      this.distCount += dist

      for (let x, y, i = 0; i < dist; i++) {
        x = a.x + Math.sin(angle) * i - offsetX
        y = a.y + Math.cos(angle) * i - offsetY
        ctx.drawImage(this.brush, x, y)
      }

      this.lastPoint = b
    },

    reset() {
      this.touchEnd()
      this.showAlert = true
      this.distCount = 0
      this.init()
    },

    touchEnd () {
      this.isDrawing = false
    },

    isOverlapped(element) {
      let document = element.ownerDocument;
      let {x, y, width, height} = element.getBoundingClientRect();
      x |= 0;
      y |= 0;
      width |= 0;
      height |= 0;
      let elements = [
        document.elementFromPoint(x, y),
        document.elementFromPoint(x + width, y),
        document.elementFromPoint(x, y + height),
        document.elementFromPoint(x + width, y + height)
      ];
      return elements.filter((el)=> el !== null).some((el)=> el !== element);
    }

  },

  watch: {
    distCount: function () {
      if (this.distCount > 1000 && this.showAlert) {
        alert('哭啊～沒中獎')
        this.touchEnd()
        this.showAlert = false
      }
    },
  },
}
</script>

<style>
.scratch-box {
  transition: .5s;
  animation: ping 8s linear infinite;
}

@keyframes ping {
  25%, 75% {
    box-shadow: 0 0 10px 3px rgba(255, 255, 255, 0.5);
  }
}

.secret-container {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: center;


}

.scratch {
  position: absolute;
  z-index: 2;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
</style>
