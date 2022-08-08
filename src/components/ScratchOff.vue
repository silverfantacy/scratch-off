<template>
  <div style="position:relative; width: 300px;height: 300px;" class="scratch-box">
    <div class="secret-container">
      <template v-if="state.showContent">
        <slot name="secret">secret</slot>
      </template>
    </div>
    <canvas class="scratch" ref="scratch"/>
  </div>
  <button :disabled="!state.showContent" style="margin-top: 20px;" @click="reset()">重新刮</button>
</template>

<script>
import {
  reactive,
  ref,
  onMounted,
  nextTick,
  watch,
} from 'vue'
import brushImage from '../assets/brush.png'
import coverImage from '../assets/cover.png'

export default {
  name: 'scratchOff',
  components: {},

  setup (props) {
    const scratch = ref(null)
    const state = reactive({
      isDrawing: false,
      lastPoint: null,
      ctx: null,
      brush: null,
      cover: null,
      showContent: false,
      showAlert: true,
      distCount: 0,
    })

    onMounted(() => {

    })
    nextTick(() => {
      init()
    })

    watch(
        () => state.distCount, (curVal, preVal) => {
          if (curVal > 1000 && state.showAlert) {
            alert('哭啊～沒中獎')
            touchEnd()
            state.showAlert = false
          }
        }, { immediate: true },
    )

    function init () {
      state.showContent = false
      const canvas = scratch.value

      canvas.width = canvas.parentElement.offsetWidth
      canvas.height = canvas.parentElement.offsetHeight

      canvas.addEventListener('mousedown', touchStart)
      canvas.addEventListener('touchstart', touchStart)
      canvas.addEventListener('mousemove', touchMove)
      canvas.addEventListener('touchmove', touchMove)
      canvas.addEventListener('mouseup', touchEnd)
      canvas.addEventListener('touchend', touchEnd)

      state.ctx = canvas.getContext('2d')

      state.brush = new Image()
      state.brush.src = brushImage

      state.cover = new Image()
      state.cover.src = coverImage
      state.cover.onload = () => {
        state.ctx.drawImage(state.cover, 0, 0, canvas.width, canvas.height)

        // 避免內容比圖片早顯示
        state.showContent = true
      }
    }

    function getPosition (event) {
      let target = scratch.value
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
    }

    function touchStart (event) {
      state.isDrawing = true
      state.lastPoint = getPosition(event)
      state.ctx.globalCompositeOperation = 'destination-out'
    }

    function touchMove (event) {
      if (!state.isDrawing) return
      event.preventDefault()

      let ctx = state.ctx
      let a = state.lastPoint
      let b = getPosition(event)
      let dist = Math.sqrt(Math.pow(b.x - a.x, 2) + Math.pow(b.y - a.y, 2))
      let angle = Math.atan2(b.x - a.x, b.y - a.y)
      let offsetX = state.brush.width / 2
      let offsetY = state.brush.height / 2

      state.distCount += dist

      for (let x, y, i = 0; i < dist; i++) {
        x = a.x + Math.sin(angle) * i - offsetX
        y = a.y + Math.cos(angle) * i - offsetY
        ctx.drawImage(state.brush, x, y)
      }

      state.lastPoint = b
    }

    function reset () {
      touchEnd()
      state.showAlert = true
      state.distCount = 0
      init()
    }

    function touchEnd () {
      state.isDrawing = false
    }

    return {
      scratch,
      state,
      reset,
    }
  },

  methods: {
    isOverlapped (element) {
      let document = element.ownerDocument
      let { x, y, width, height } = element.getBoundingClientRect()
      x |= 0
      y |= 0
      width |= 0
      height |= 0
      let elements = [
        document.elementFromPoint(x, y),
        document.elementFromPoint(x + width, y),
        document.elementFromPoint(x, y + height),
        document.elementFromPoint(x + width, y + height),
      ]
      return elements.filter((el) => el !== null).some((el) => el !== element)
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
