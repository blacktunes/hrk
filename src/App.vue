<template>
  <Loading :ready="ready" />
  <transition name="in" appear>
    <div v-if="ready">
      <VSelect ref="select">
        <div @click="reset" v-if="isRandom || isShowAll" class="icon">
          Select
        </div>
        <div @click="setRandom" v-if="!isRandom" class="icon">Random</div>
        <div @click="showAll" v-if="!isShowAll" class="icon">All</div>
      </VSelect>
    </div>
  </transition>
  <div class="home" :class="[isHorizontal ? 'horizontal' : 'vertical']">
    <div>
      <img
        :style="{ filter: ready ? '' : 'blur(10px)' }"
        alt=""
        class="img"
        src="https://cdn.jsdelivr.net/gh/blacktunes/hrk/public/bg.jpg"
        @load.once="loaded"
      />
      <div
        v-if="!isRandom && !isShowAll"
        class="grids"
        :style="{
          width: isHorizontal ? 100 * (993 / 1105) + 'vh' : '100vw',
          height: isHorizontal ? '100vh' : 100 * (1105 / 993) + 'vw',
        }"
        @mouseleave="hide"
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
      <div
        @mouseover="moveIn(key)"
        @mouseleave="moveOut(key)"
        class="random-card"
        :style="{
          'z-index': item.index,
          top: item.y,
          left: item.x,
          opacity: item.show === 2 ? 1 : 0,
          transform:
            item.show !== 1
              ? 'translate(-50%, -50%) scale(1, 1)'
              : 'translate(-50%, -50%) scale(0, 0)',
        }"
      >
        <Card>
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
import VSelect from './components/Select.vue'
import Loading from './components/Loading.vue'

const initDefault = (ready) => {
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
    if (!ready.value) return
    isShow.value = true
    selectIndex.value = getIndex(key - 1)
  }

  const hide = () => {
    if (!ready.value) return
    isShow.value = false
  }

  return {
    selectIndex,
    isShow,
    show,
    hide
  }
}

/**
 * @param {number} num  生成的卡片数
 * @param {[number, number]} r 随机区间
 * @param {{value: boolean}} isHorizontal
 * @param {{start:number,end:number}} pos
 */
const initRandom = (num, r, isHorizontal, pos) => {
  const range = [[15, 85], [20, 80]]

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
      show: 0,
      x: isHorizontal.value ? random(pos.start, pos.end) : random(range[1][0], range[1][1]) + 'vw',
      y: isHorizontal.value ? random(range[1][0], range[1][1]) + 'vh' : random(pos.start, pos.end),
      name: data.name,
      text: data.text,
      index: i,
      timer: null
    }
  }

  const remove = () => {
    for (const item of showList.value) {
      clearTimeout(item.timer)
      item.timer = null
      item.show = false
    }
  }

  const setTimer = (key, times) => {
    showList.value[key].timer = setTimeout(() => {
      ++showList.value[key].show
      if (showList.value[key].show > 2) {
        showList.value[key].show = 0
      }
      if (showList.value[key].show === 2) {
        showList.value[key].index = 3

        showList.value[key].x = isHorizontal.value ? random(range[0][0], range[0][1]) + 'vw' : random(range[1][0], range[1][1]) + 'vh'
        showList.value[key].y = isHorizontal.value ? random(range[1][0], range[1][1]) + 'vh' : random(range[0][0], range[0][1]) + 'vh'
      } else if (showList.value[key].show === 1) {
        showList.value[key].x = isHorizontal.value ? random(pos.start, pos.end) + 'px' : random(range[1][0], range[1][1]) + 'vw'
        showList.value[key].y = isHorizontal.value ? random(range[1][0], range[1][1]) + 'vh' : random(pos.start, pos.end) + 'px'
      } else {
        showList.value[key].index = 2
      }
      showList.value[key].timer = null
      setTimer(key)
    }, showList.value[key].show === 1 ? 300 : random(times ? times[0] : r[0], times ? times[1] : r[1]))
  }

  const initCard = () => {
    for (let i = 0; i < num; i++) {
      setTimer(i, [0, 1500])
    }
  }

  const moveIn = (key) => {
    clearTimeout(showList.value[key].timer)
    showList.value[key].timer = null
  }

  const moveOut = (key) => {
    setTimer(key, [0, 1500])
  }

  return {
    isRandom,
    showList,
    setTimer,
    initCard,
    remove,
    moveIn,
    moveOut
  }
}

export default {
  components: {
    Card,
    VSelect,
    Loading
  },
  setup() {
    const ready = ref(false)
    const loaded = () => {
      setTimeout(() => {
        ready.value = true
      }, 300)
    }

    const select = ref()

    const isHorizontal = ref(true)
    const pos = {
      start: 50,
      end: 50
    }
    const getDirection = () => {
      isHorizontal.value = document.documentElement.clientWidth > document.documentElement.clientHeight
      if (isHorizontal.value) {
        const temp = (document.documentElement.clientWidth - (document.documentElement.clientHeight * (993 / 1105))) / 2
        pos.start = temp
        pos.end = document.documentElement.clientWidth - temp
      } else {
        const temp = (document.documentElement.clientHeight - document.documentElement.clientWidth * (1105 / 993)) / 2
        pos.start = temp
        pos.end = document.documentElement.clientHeight - temp
      }
    }
    onMounted(() => {
      getDirection()
      window.onresize = getDirection
    })

    const { selectIndex, isShow, show, hide } = initDefault(ready)
    const { isRandom, showList, initCard, remove, moveIn, moveOut } = initRandom(10, [1500, 5500], isHorizontal, pos)

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
      ready,
      loaded,
      isHorizontal,
      selectIndex, isShow, show, hide,
      isRandom, setRandom, showList, remove,
      isShowAll,
      showAll,
      reset,
      moveIn,
      moveOut
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
  position fixed
  top 0
  left 0
  height 100vh
  width 100vw
  display flex
  align-items center

  .img
    z-index -1
    transition filter 0.5s
    transition-delay 0.25s

.grids
  display grid
  grid-template-columns repeat(58, 1.724%)
  grid-template-rows repeat(86, 1.163%)
  grid-row-gap 0
  grid-column-gap 0
  position fixed
  top 50%
  left 50%
  transform translate(-50%, -50%)

  .grid
    box-sizing border-box
    border 1px solid transparent

    &:hover
      border-color red

.card
  position fixed
  top 50%
  left 50%
  transform translate(-50%, -50%)
  pointer-events none
  transition opacity 0.4s

.all-list
  z-index 2
  display flex
  flex-direction column
  align-items center

  .all-card
    z-index 3
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
    position fixed
    z-index 2
    opacity 0
    transition opacity 0.4s ease-in, transform 0.5s cubic-bezier(0.18, 0.89, 0.32, 1.1), left 0.5s cubic-bezier(0.18, 0.89, 0.32, 1.1), top 0.5s cubic-bezier(0.18, 0.89, 0.32, 1.1)
    transform translate(-50%, -50%) scale(0, 0)

    &:hover
      z-index 99 !important

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
</style>
