<template>
  <div
    class="ball-root"
    :class="{active: isActive}"
    :style="rootStyle"
    @touchstart="onTouchStart"
    @touchend="onTouchEnd"
    @touchmove="onTouchMove"
    >
    <div class="circular"></div>
  </div>
</template>

<script>
export default {
  name: 'DebugBall',

  props: {
    defaultPosition: {
      type: Object,
      default () {
        return {
          left: 300,
          top: 300
        }
      }
    }
  },

  data () {
    return {
      isActive: false,
      position: {
        left: 0,
        top: 0
      },
      size: {
        width: 0,
        height: 0
      },
      containerSize: {
        width: 0,
        height: 0
      },
      transitionDuration: 0
    }
  },

  computed: {
    rootStyle () {
      const {left, top} = this.position
      return {
        left: left + 'px',
        top: top + 'px',
        transitionDuration: this.transitionDuration + 's'
      }
    }
  },
  mounted () {
    this.init()
  },

  methods: {
    init () {
      this.getContainerSize()
      this.getBallSize()
      this.resetPosition(this.defaultPosition)
    },
    onTouchStart (e) {
      this.closeTransition()
      this.active()
    },
    onTouchEnd (e) {
      this.openTransition()
      this.unActive()
      const hasChange = this.resetPosition(this.position)
      !hasChange && this.emitClick(e)
    },

    onTouchMove (e) {
      const touches = e.touches || e.changedTouches || []
      const touch = touches[0] || {}
      const {pageX, pageY} = touch
      let newLeft = pageX - this.size.width / 2
      let newTop = pageY - this.size.height / 2
      this.setPosition({left: newLeft, top: newTop})
    },

    active () {
      window.clearTimeout(this.unActiveTimeId)
      this.isActive = true
    },

    unActive () {
      this.unActiveTimeId = setTimeout(() => {
        this.isActive = false
      }, 1000)
    },

    getBallSize () {
      const {clientWidth, clientHeight} = this.$el
      this.size = {width: clientWidth, height: clientHeight}
    },

    resetPosition ({left, top}) {
      const {width, height} = this.size
      const {width: containerWidth, height: containerHeight} = this.containerSize

      const MIM_LENGTH = 10
      const minLeft = MIM_LENGTH
      const minTop = MIM_LENGTH
      const maxLeft = containerWidth - MIM_LENGTH - width
      const maxTop = containerHeight - MIM_LENGTH - height

      let newLeft = left
      let newTop = top
      let hasChange = false

      if (left < containerWidth / 2) {
        newLeft = minLeft
      } else {
        newLeft = maxLeft
      }

      if (top > maxTop) {
        newTop = maxTop
      }
      if (top < minTop) {
        newTop = minTop
      }

      if (this.position.left !== newLeft || this.position.top !== newTop) {
        this.position.left = newLeft
        this.position.top = newTop
        hasChange = true
      }
      return hasChange
    },
    setPosition ({left, top}) {
      this.position.left = left
      this.position.top = top
    },

    getContainerSize () {
      const {innerWidth, innerHeight} = window
      this.containerSize = {width: innerWidth, height: innerHeight}
    },

    setTransition () {
      this.positionTransition = true
      setTimeout(() => {
        this.positionTransition = false
      })
    },
    openTransition () {
      this.transitionDuration = 0.2
    },
    closeTransition () {
      this.transitionDuration = 0
    },

    emitClick () {
      const params = {
        position: this.position,
        size: this.size,
        center: {
          left: this.position.left + this.size.width / 2,
          top: this.position.top + this.size.height / 2
        }
      }
      this.$emit('click', params)
    }
  }
}
</script>

<style scoped>
  .ball-root {
    position: absolute;
    width: 13vw;
    height: 13vw;
    background: #000;
    opacity: 0.1;
    box-shadow: 0 0 4vw 2vw rgba(0,0,0,0.1);
    border-radius: 20%;
    transition: all ease 0.2s;
  }
  .ball-root.active {
    opacity: 0.3;
  }
  .circular {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 70%;
    height: 70%;
    background: #fff;
    opacity: 0.9;
    border-radius: 50%;
    border: 1vw solid rgba(0,0,0,0.5);
    box-sizing: content-box;
  }
</style>
