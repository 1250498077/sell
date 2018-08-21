<template>
  <div class="shopcart">
    <div class="content" @click="toggleList(1)">
      <div class="content-left">
        <div class="logo-wrapper" ref="balllogo">
          <div class="logo" :class="{'highlight':totalCount>0}">
            <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
          </div>
          <div class="num" v-show="totalCount>0">{{totalCount}}</div>
        </div>
        <div class="price" :class="{'highlight':totalCount>0}">￥{{totalPrice}}</div>
        <div class="desc">另需配送费{{totalPrice}}元</div>
      </div>
      <div class="content-right">
        <div class="pay" :class="payClass">
          {{payDesc}}
        </div>
      </div>
    </div>
    <div class="ball-container">
      <div v-for="(ball, key) in balls" :key="key">
        <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
          <div class="ball" v-show="ball.show">
            <div class="inner inner-hook"></div>
          </div>
        </transition>
      </div>
    </div>
    <transition name="fold">
      <div class="shopcart-list" v-show="listShow">
        <div class="list-header">
          <h1 class="title">购物车</h1>
          <span class="empty" @click="empty">清空</span>
        </div>
        <div class="list-content" ref="listContent">
          <ul ref="listed">
            <li class="food" v-for="(food, key) in selectFoods" :key="key">
              <span class="name">{{food.name}}</span>
              <div class="price">
                <span>￥{{food.price*food.count}}</span>
              </div>
              <div class="cartcontrol-wrapper">
                <cartcontrol :food="food"></cartcontrol>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
import cartcontrol from 'components/cartcontrol/cartcontrol'
import BScroll from 'better-scroll'
export default {
  components: {
    cartcontrol
  },
  props: {
    selectFoods: {
      type: Array,
      default() {
        return [
          {}
        ]
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    },
    rect: {
      type: DOMRect
    }
  },
  data() {
    return {
      balls: [
        {
          show: false,
          el: null
        },
        {
          show: false,
          el: null
        },
        {
          show: false,
          el: null
        },
        {
          show: false,
          el: null
        },
        {
          show: false,
          el: null
        }
      ],
      dropBalls: [],
      fold: false,
      counted: 0
    }
  },
  computed: {
    // 使用vue一定要利用数据的响应式，通过修改数据触发连续响应
    // 就可以避免过度耦合，各个组件各个逻辑都基于数据响应，就可以避免组件与组件之间的耦合
    totalPrice() {
      let total = 0
      this.selectFoods.forEach((food) => {
        total += food.price * food.count
      })
      return total
    },
    totalCount() {
      let count = 0
      this.selectFoods.forEach((food) => {
        count += food.count
      })
      return count
    },
    payDesc() {
      if (this.totalPrice === 0) {
        return `￥${this.minPrice}元起送`
      } else if (this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice
        return `还差￥${diff}元起送`
      } else {
        return '去结算'
      }
    },
    payClass() {
      if (this.totalPrice < this.minPrice) {
        return 'not-enough'
      } else {
        return 'enough'
      }
    },
    listShow: {
      // 在计算属性里面不要修改data属性的值
      get: function () {
        if (this.totalCount === 0) {
          return false
        }
        return this.fold
      },
      set: function () {
        if (!this.totalCount) {
          this.fold = true
          return false
        }
        let show = !this.fold
        return show
      }
    }
  },
  created() {
    this.$nextTick(() => {
      this._initScroll()
    })
  },
  watch: {
    selectFoods: function() {
      this.toggleList(2)
    }
  },
  methods: {
    empty() {
      console.log('empty')
      this.$emit('empty')
    },
    _initScroll() {
      this.foodsScroll = new BScroll(this.$refs.listContent, {
        click: true
      })
    },
    toggleList(num) {
      let counted = 0
      let height = 0
      if (!this.totalCount) {
        return
      }
      if (num === 1) {
        this.fold = !this.fold
      }
      if (this.fold) {
        counted = this.selectFoods.length
        if (counted <= 4) {
          height = counted * 40
          this.$refs.listContent.style.height = height + 'px'
          this.$nextTick(() => {
            this.foodsScroll.refresh()
          })
        } else {
          height = 4 * 40
          this.$refs.listContent.style.height = height + 'px'
          this.$nextTick(() => {
            this.foodsScroll.refresh()
          })
        }
      }
    },
    drop(el) {
      this.$nextTick(() => {
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i]
          if (!ball.show) {
            ball.show = true
            ball.el = el
            this.dropBalls.push(ball)
            return
          }
        }
      })
    },
    beforeDrop: function(el, done) {
      let count = this.balls.length
      while (count--) {
        let ball = this.balls[count]
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect()
          let x = rect.left - 32
          let y = -(window.innerHeight - rect.top - 22)
          el.style.display = ''
          el.style.transform = `translate3d(0, ${y}px, 0)`
          el.style.webkitTransform = `translate3d(0, ${y}px, 0)`
          let inner = el.getElementsByClassName('inner-hook')[0]
          inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`
          inner.style.transform = `translate3d(${x}px, 0, 0)`
        }
      }
    },
    dropping: function(el, done) {
      /* eslint-disable no-unused-vars */
      let rf = el.offsetHeight
      this.$nextTick(() => {
        el.style.webkitTransform = 'translate3d(0, 0, 0)'
        el.style.transform = 'translate3d(0, 0, 0)'
        let inner = el.getElementsByClassName('inner-hook')[0]
        inner.style.webkitTransform = 'translate3d(0, 0, 0)'
        inner.style.transform = 'translate3d(0, 0, 0)'
        done()
      })
    },
    afterDrop: function(el) {
      let ball = this.dropBalls.shift()
      if (ball) {
        ball.show = false
        el.style.display = 'none'
      }
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin"
.shopcart
  position: fixed
  left: 0
  bottom: 0
  z-index: 50
  width: 100%
  height: 48px
  background: 000
  .content
    display: flex
    background: #141d27
    .content-left
      flex: 1
      .logo-wrapper
        display: inline-block
        position: relative
        top: -10px
        margin: 0 12px
        padding: 6px
        width: 56px
        height: 56px
        box-sizing: border-box
        vertical-align: top
        border-radius: 50%
        background: #141d27
        .logo
          width: 100%
          height: 100%
          border-radius: 50%
          text-align: center
          background: #2d343c
          &.highlight
            background: rgb(0, 160, 220)
          .icon-shopping_cart
            padding-left: 3px
            line-height: 44px
            font-size: 24px
            color: #80858a
            &.highlight
              color: white
        .num
          position: absolute
          top: 0
          right: 0
          width: 24px
          height: 16px
          line-height: 16px
          text-align: center
          border-radius: 16px
          font-size: 9px
          font-weight: 700
          color: #fff
          background: rgb(240, 20, 20)
          box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4)
      .price
        display: inline-block
        vertical-align: top
        margin-top: 12px
        line-height: 24px
        padding-right: 12px
        box-sizing: border-box
        border-right:1px soild rgba(255, 255, 255, 1)
        font-size: 16px
        font-weight: 700
        color: rgba(255, 255, 255, 0.4)
        &.highlight
          color: #fff
      .desc
        display: inline-block
        vertical-align: top
        margin: 12px 0 0 12px
        line-height: 24px
        color: rgba(255, 255, 255, 0.4)
        font-size: 10px
    .content-right
      flex: 0 0 105px
      width: 105
      &.highlight
        background: green
      .pay
        height: 48px
        line-height: 48px
        text-align: center
        font-size: 12px
        font-weight: 700
        color: rgba(255, 255, 255, 0.4)
        background: #2b333b
        &.not-enough
          background: #2b333b
        &.enough
          background: #00b43c
          color: #fff
  .ball-container
    .ball
      position: fixed
      left: 32px
      bottom: 22px
      z-index: 200
      display: inline-block
      background: rgb(0, 160, 220)
      transition: all 2s linear
      .inner
        width: 16px
        height: 16px
        border-radius: 50%
        background: red
        transition: all 2s linear
        &.drop-enter-active
          transition all 1s cubic-bezier(0.49, -0.29, 0.75, 0.41)
  .shopcart-list
    position: absolute
    top: 0
    left: 0
    z-index: -1
    width: 100%
    transform translate3d(0, -100%, 0)
    &.fold-enter-active, &.fold-leave-active
      transition: all 0s linear
    &.fold-enter, &.fold-leave-to
      transform: translate(0, 0, 0)
    .list-header
      height: 40px
      line-height: 40px
      padding: 0 18px
      background: #f3f5f7
      border-bottom: 1px solid rgba(7, 17, 27, 0.1)
      .title
        float: left
        font-size: 14px
        color: rgb(7, 17, 27)
      .empty
        float: right
        font-size: 12px
        color: rgb(0, 160, 220)
    .list-content
      padding: 0 18px
      max-height: 217px
      overflow: hidden
      background: #fff
      .food
        position: relative
        padding: 12px 0
        box-sizing: border-box
        border-1px(rgba(7, 17, 27, 0.1))
        .name
          line-heigth: 24px
          font-size: 14px
          color: rgb(7, 17, 27)
        .price
          position: absolute
          right: 90px
          bottom: 12px
          line-height: 24px
          font-size: 14px
          font-weight: 700
          color: rgb(240, 20, 20)
        .cartcontrol-wrapper
          position: absolute
          right: 0
          bottom: 6px
</style>
