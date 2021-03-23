<template>
  <Select ref="select">
    <div @click="reset" v-if="isRandom || isShowAll" class="icon">Select</div>
    <div @click="setRandom" v-if="!isRandom" class="icon">Random</div>
    <div @click="showAll" v-if="!isShowAll" class="icon">All</div>
  </Select>
  <div class="home" :class="[isHorizontal ? 'horizontal' : 'vertical']">
    <div>
      <div
        v-if="!isRandom && !isShowAll"
        class="grids"
        :style="{
          width: isHorizontal ? 100 * (993 / 1105) + 'vh' : '100vw',
          height: isHorizontal ? '100vh' : 100 * (1105 / 993) + 'vw',
        }"
      >
        <div
          class="grid"
          v-for="i in 4988"
          :key="i"
          @mouseenter="show(i)"
        ></div>
      </div>
    </div>
  </div>
  <div
    class="card"
    :style="{ opacity: isShow ? 1 : 0 }"
    v-if="!isRandom && !isShowAll"
  >
    <Card class="mid-card">
      {{ Data[selectIndex].text }}
      <template #name>{{ Data[selectIndex].name }}</template>
    </Card>
  </div>
  <template v-else-if="isShowAll">
    <transition name="fade" appear>
      <div class="all-list">
        <Card v-for="(card, key) in Data" :key="'card' + key" class="all-card">
          {{ card.text }}
          <template #name>{{ card.name }}</template>
        </Card>
      </div>
    </transition>
  </template>
  <div
    class="random-list"
    :style="{ 'z-index': isRandom ? 1 : -1 }"
    v-else-if="isRandom"
  >
    <template v-for="(item, key) in showList" :key="'random' + key">
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
import { onMounted, ref } from 'vue'
import Data from './assets/hrk.json'
import Card from './components/Card.vue'
import Select from './components/Select.vue'

const initDefault = () => {
  /** 是否显示选择卡片 */
  const isShow = ref(false)
  const selectIndex = ref(0)

  const getIndex = (key) => {
    let temp = key
    if (temp > (Data.length - 1)) {
      temp -= (Data.length - 1)
      return getIndex(temp)
    } else {
      return temp
    }
  }

  const show = (key) => {
    isShow.value = true
    selectIndex.value = getIndex(key - 1)
  }

  return {
    selectIndex,
    isShow,
    show
  }
}

/**
 * @param {number} num  生成的卡片数
 * @param {[number, number]} r 随机区间
 */
const initRandom = (num, r) => {
  const isRandom = ref(false)

  const random = (min, max) => {
    if (min === undefined || max === undefined) {
      return Math.random()
    }
    return Math.floor(Math.random() * (Number(max) - Number(min) + 1)) + Number(min)
  }

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
    for (const item of showList.value) {
      item.show = false
    }
    timers = {}
  }

  const setTimer = (key, times) => {
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
        setTimer(key)
      }, random(times ? times[0] : r[0], times ? times[1] : r[1]))
    }
  }

  const initCard = () => {
    for (let i = 0; i < num; i++) {
      setTimer(i, [500, 1000])
    }
  }

  return {
    isRandom,
    showList,
    initCard,
    remove
  }
}

export default {
  components: {
    Card,
    Select
  },
  setup() {
    const select = ref()

    const isHorizontal = ref(true)
    const getDirection = () => {
      isHorizontal.value = document.documentElement.clientWidth > document.documentElement.clientHeight
    }
    onMounted(() => {
      window.onresize = getDirection
    })

    const { selectIndex, isShow, show } = initDefault()
    const { isRandom, showList, initCard, remove } = initRandom(10, [2000, 8000])

    const reset = () => {
      isShow.value = false
      isRandom.value = false
      remove()
      isShowAll.value = false
      select.value.showItem()
    }

    const setRandom = () => {
      isShowAll.value = false
      isRandom.value = true
      initCard()
      select.value.showItem()
    }

    const isShowAll = ref(false)
    const showAll = () => {
      isRandom.value = false
      remove()
      isShowAll.value = true
      select.value.showItem()
    }



    return {
      select,
      Data,
      isHorizontal,
      selectIndex, isShow, show,
      isRandom, setRandom, showList, remove,
      isShowAll,
      showAll,
      reset
    }
  }
}
</script>

<style lang="stylus">
body
  margin 0

.icon
  height 40px
  line-height 40px
  user-select none
  cursor pointer
  text-align center
  font-size 12px
  font-weight bold

.home
  z-index -1
  position fixed
  top 0
  left 0
  height 100vh
  width 100vw
  background url('/bg.jpg') center center no-repeat
  background-size contain

.grids
  display grid
  grid-template-columns repeat(58, 1.724%)
  grid-template-rows repeat(86, 1.163%)
  grid-row-gap 0px
  grid-column-gap 0px
  position fixed
  top 50%
  left 50%
  transform translate(-50%, -50%)

  .grid:hover
    box-sizing border-box
    border 1px solid red

.card
  position fixed
  top 50%
  left 50%
  transform translate(-50%, -50%)
  pointer-events none
  transition opacity 0.3s

.all-list
  z-index 2
  display flex
  flex-direction column
  align-items center

  .all-card
    max-width 80%

.random-list
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

.horizontal
  flex-direction column

  .img
    height 100vh

.vertical
  flex-direction row

  .img
    width 100vw

@media only screen and (min-width 900px)
  .mid-card
    max-width 600px !important

@media only screen and (max-width 500px)
  .random-card
    font-size 10px !important

@media only screen and (min-width 900px)
  .all-card
    max-width 600px !important

.fade-enter-active, .fade-leave-active
  transition opacity 0.3s

.fade-enter-from, .fade-leave-to
  opacity 0

.fade-enter-to, .fade-leave-from
  opacity 1
</style>
