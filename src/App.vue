<template>
  <div class="index"></div>
  <template v-if="isShow">
    <transition name="fade" appear>
      <div class="card-list">
        <Card v-for="(card, key) in Data" :key="'card' + key" class="card">
          {{ card.text }}
          <template #name>{{ card.name }}</template>
        </Card>
      </div>
    </transition>
  </template>
  <div class="list" v-else @click.once="showAll">
    <template v-for="(item, index) in showList" :key="index">
      <div class="random-card" :style="{ opacity: item.show ? 1 : 0 }">
        <Card style="position: fixed" :style="{ top: item.y, left: item.x }">
          {{ item.text }}
          <template #name>{{ item.name }}</template>
        </Card>
      </div>
    </template>
  </div>
</template>

<script>
import { onDeactivated, onMounted, ref } from 'vue'
import Card from './components/Card.vue'
import Data from './assets/hrk.json'

export default {
  components: { Card },
  setup() {
    const isShow = ref(false)
    const showAll = () => {
      isShow.value = true
      remove()
    }

    const random = (min, max) => {
      if (min === undefined || max === undefined) {
        return Math.random()
      }
      return Math.floor(Math.random() * (Number(max) - Number(min) + 1)) + Number(min)
    }

    // 需要生成的卡片数
    const num = 10
    // 随机区间
    const r = [2000, 8000]

    const showList = ref([])
    for (let i = 0; i < num; i++) {
      const data = Data[random(0, Data.length - 1)]
      showList.value[i] = {
        show: false,
        x: random(0, 80) + 'vw',
        y: random(0, 80) + 'vh',
        name: data.name,
        text: data.text
      }
    }

    let timers = {}
    const remove = () => {
      for (const i in timers) {
        clearTimeout(timers[i])
        timers[i] = null
      }
      timers = null
    }

    const testTimer = (key) => {
      if (timers) {
        const time = Date.now()
        timers[time] = setTimeout(() => {
          showList.value[key].show = !showList.value[key].show
          if (showList.value[key].show) {
            showList.value[key].x = random(0, 80) + 'vw'
            showList.value[key].y = random(0, 80) + 'vh'
          }
          if (timers) {
            delete timers[time]
          }
          testTimer(key)
        }, random(r[0], r[1]))
      }
    }

    onMounted(() => {
      for (let i = 0; i < num; i++) {
        testTimer(i)
      }
    })

    onDeactivated(() => {
      remove()
    })

    return {
      isShow,
      showAll,
      Data,
      showList,
      random
    }
  }
}
</script>

<style lang="stylus">
.index
  z-index -1
  position fixed
  top 0
  left 0
  height 100vh
  width 100vw
  background url('/bg.jpg') center center no-repeat
  background-size contain

.card-list
  z-index 2
  display flex
  flex-direction column
  align-items center

  .card
    max-width 80%

.list
  z-index 1
  overflow hidden
  position fixed
  top 0
  left 0
  height 100vh
  width 100vw

  .random-card
    opacity 0
    transition opacity 0.3s

@media only screen and (min-width 900px)
  .card
    max-width 600px !important

.fade-enter-active, .fade-leave-active
  transition opacity 0.3s

.fade-enter-from, .fade-leave-to
  opacity 0

.fade-enter-to, .fade-leave-from
  opacity 1

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
</style>
