<template>
  <div class="loading" v-show="isShow && !isLoad">
    <div>loading</div>
  </div>
  <div class="index" @click="show">
    <img @load="load" v-if="isShow" alt="hiiro" src="/Hiiro.jpg" />
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const isShow = ref(false)
    const show = () => {
      isShow.value = !isShow.value
    }

    const isLoad = ref(false)
    const load = () => {
      isLoad.value = true
    }

    return {
      isShow,
      show,
      isLoad,
      load
    }
  }
}
</script>

<style lang="stylus">
body
  margin 0
  -webkit-tap-highlight-color rgba(0, 0, 0, 0)

::-webkit-scrollbar
  width 7px
  height 7px

::-webkit-scrollbar-track
  box-shadow inset 0 0 6px rgba(0, 0, 0, 0.3)
  -webkit-box-shadow inset 0 0 6px rgba(0, 0, 0, 0.3)
  background-color #F5F0F2

::-webkit-scrollbar-thumb
  box-shadow inset 0 0 6px rgba(0, 0, 0, 0.1)
  -webkit-box-shadow inset 0 0 6px rgba(0, 0, 0, 0.1)
  background-color #F5C1BB

::-webkit-scrollbar-thumb:active
  background-color #FADBD8

.index
  height 100vh
  width 100vw
  background url('/bg.jpg') center center no-repeat
  background-size contain

.loading
  position fixed
  top 0
  left 0
  height 100vh
  width 100vw
  background rgba(0, 0, 0, 0.5)

  div
    position fixed
    top 50%
    left 50%
    transform translate(-50%, -50%)
</style>
