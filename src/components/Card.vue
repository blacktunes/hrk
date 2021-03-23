<template>
  <div class="card-wrapper">
    <div class="card-text">
      <slot></slot>
    </div>
    <div class="card-name" v-if="$slots.name">——<slot name="name"></slot></div>
  </div>
</template>

<style lang="stylus" rel="stylesheet/stylus" scoped>
.card-wrapper
  display inline-block
  min-width 300px
  max-width 350px
  margin 10px 5px
  padding 10px
  border-radius 5px
  border 1px solid #F5C1BB
  background-color rgba(250, 219, 216, 0.6)
  color #303133
  transition box-shadow 0.25s, background-color 0.2s
  user-select none

  &:hover
    z-index 9
    background-color rgba(250, 219, 216, 0.8) !important
    box-shadow 2px 2px 5px rgba(0, 0, 0, 0.1)

  .card-text
    text-align center
    padding 10px

  .card-name
    text-align right
    padding 10px
</style>
