<template>
  <div class="cartcontrol">
    <transition name="move">
      <div class="cart-decrease icon-remove_circle_outline" v-show="food.count>0" @click.stop.prevent="decreaseCart"></div>
    </transition>
    <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
    <div class="cart-add icon-add_circle" @click.stop.prevent="addCart($event)" ref="mybox"></div>
  </div>
</template>

<script type="text/ecmascript-6">
import Vue from 'vue'
export default {
  props: {
    food: {
      type: Object
    }
  },
  methods: {
    addCart(event) {
      if (!event._constructed) {
        return
      }
      if (!this.food.count) {
        // 这里很重要，food是good组件传过来对象，我们给这个对象添加count属性并且设置为1
        // 这不是改变了父组件传过来的值的吗，并且响应式的影响到父组件
        Vue.set(this.food, 'count', 1)
      } else {
        this.food.count++
      }
      this.$emit('cartadd', event.target)
    },
    decreaseCart(event) {
      if (!event._constructed) {
        return
      }
      if (this.food.count) {
        this.food.count--
      }
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .cartcontrol
    width: 100px
    height: 30px
    font-size: 0
    .cart-decrease
      display: inline-block
      position: absolute
      width: 24px
      height: 30px
      font-size: 24px
      line-height: 30px
      text-align: center
      right: 48px
      color: rgb(0, 160, 220)
      opacity: 1
      transform: rotateZ(0)
      &.move-enter-active, &.move-leave-active
        transition: all 0.4s linear
      &.move-enter, &.move-leave-to
        opacity: 0
        right: 0px
        transform: rotateZ(180deg)
    .cart-count
      display: inline-block
      position: absolute
      width: 24px
      height: 30px
      right: 24px
      font-size: 10px
      line-height: 30px
      text-align: center
    .cart-add
      display: inline-block
      position: absolute
      right: 0
      height: 30px
      flex: 0 0 30px
      font-size: 24px
      line-height: 30px
      text-align: center
      color: rgb(0, 160, 220)
</style>
