<template>
  <transition name="fade" appear>
    <div class="loading" v-if="!ready || show">
      <div class="progress" :style="{ width: width }">
        <div>
          <span>L</span>
          <span>o</span>
          <span>a</span>
          <span>d</span>
          <span>i</span>
          <span>n</span>
          <span>g</span>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import { ref, watch } from 'vue'

export default {
  props: {
    ready: Boolean
  },
  setup(props) {
    const show = ref(true)
    const width = ref('')

    watch(props, () => {
      if (props.ready) {
        width.value = '100% !important'
        setTimeout(() => {
          show.value = false
        }, 500)
      }
    })

    return {
      show,
      width
    }
  }
}
</script>

<style lang="stylus" scoped>
.loading
  z-index 99
  position fixed
  top 0
  left 0
  height 100vh
  width 100vw
  display flex
  flex-direction column
  align-items center
  justify-content center
  background rgba(250, 219, 216, 0.3)
  user-select none
  font-size 30px

  .progress
    display flex
    align-items center
    justify-content center
    width 0
    height 5px
    border-radius 5px
    background rgb(250, 219, 216)
    transition width 0.2s
    transition-delay 0.5s
    animation progress 0.5s
    animation-fill-mode forwards
    box-shadow 0px 0px 10px 2px rgba(250, 219, 216, 0.9)

    div
      padding-bottom 9px

      span
        display inline-block
        animation shake 2s infinite

        &:nth-child(1)
          animation-delay 0.5s

        &:nth-child(2)
          animation-delay 0.6s

        &:nth-child(3)
          animation-delay 0.7s

        &:nth-child(4)
          animation-delay 0.8s

        &:nth-child(5)
          animation-delay 0.9s

        &:nth-child(6)
          animation-delay 1s

        &:nth-child(7)
          animation-delay 1.1s
</style>
