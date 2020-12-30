<template>
  <view class="min-countdown" :class="countdownClass">
    <rich-text :nodes="time"></rich-text>
  </view>
</template>

<script>
export default {
  name: 'min-countdown',
  props: {
    targetTime: {
      type: Number,
      default: 0
    },
    format: {
      type: String,
      default: '{%h}:{%m}:{%s}'
    },
    countdownClass: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      time: '00:00:00'
    }
  },
  methods: {
    init () {
      setTimeout(() => {
        this.getTime()
      }, 1000)
    },
    getTime () {
      let time = {}
      let format = this.format
      function formatNumber (num) {
        return num > 9 ? `${num}` : `0${num}`
      }
      const gapTime = Math.ceil((this.targetTime - new Date().getTime()) / 1000)
      if (gapTime >= 0) {
        time.d = formatNumber(parseInt(gapTime / 86400))
        let lastTime = gapTime % 86400;
        time.h = formatNumber(parseInt(lastTime / 3600))
        lastTime = lastTime % 3600;
        time.m = formatNumber(parseInt(lastTime / 60))
        time.s = formatNumber(lastTime % 60)
        ;['d', 'h', 'm', 's'].forEach(item => {
          const day = time[item].split('');
          format = format.replace('{%' + item + '}', time[item])
          format = format.replace('{%' + item + '0}', day[0])
          format = format.replace('{%' + item + '1}', day[1])
          format = format.replace('{%' + item + '2}', day[2] ? day[2] : '0')
        })
        this.time = format
        this.init()
      } else {
        this.$emit('callback')
      }
    }
  },
  mounted () {
    this.getTime()
  }
}
</script>

<style scoped>
.min-countdown {
  display: inline-flex;
  justify-content: center;
  align-items: center;
}
</style>
