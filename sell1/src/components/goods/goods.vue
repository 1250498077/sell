<template>
<div class="goods">
  <div class="menu-wrapper" ref="menuWrapper">
    <ul>
      <li class="menu-item" v-for="(item, key) in goods" :key="key">
        <span class="text">
          <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
          {{item.name}}
        </span>
      </li>
    </ul>
  </div>
  <div class="foods-wrapper" ref="foodsWrapper">
    <ul>
      <li v-for="(item, key) in goods" :key="key" class="food-list food-list-hook">
        <h1 class="title">{{item.name}}</h1>
        <ul>
          <li @click="selectFood(food,$event)" v-for="(food, key) in item.foods" :key="key" class="food-item border-1px">
            <div class="icon">
              <img :src="food.icon" width="57" height="57">
            </div>
            <div class="content">
              <h2 class="name">{{food.name}}</h2>
              <p class="desc">{{food.description}}</p>
              <div class="extra">
                <span>月售{{food.sellCount}}份</span>
                <span>好评率{{food.rating}}%</span>
              </div>
              <div class="price">
                <span class="now">￥{{food.price}}</span>
                <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
              </div>
              <div class="cartcontrol-wrapper">
                <cartcontrol :food="food" v-on:cartadd="cartadd"></cartcontrol>
              </div>
            </div>
          </li>
        </ul>
      </li>
    </ul>
  </div>
  <shopcart ref="shopcart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" v-on:empty="empty"></shopcart>
  <food :food="selectedFood" ref="food"></food>
</div>
</template>

<script type="text/ecmascript-6">
import BScroll from 'better-scroll'
import shopcart from 'components/shopcart/shopcart'
import cartcontrol from 'components/cartcontrol/cartcontrol'
import food from 'components/food/food'
import Vue from 'vue'
const ERR_OK = 0
export default {
  components: {
    shopcart,
    cartcontrol,
    food

  },
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return {
      goods: [],
      classMap: ['decrease', 'discount', 'special', 'invoice', 'guarantee'],
      selectedFood: {}
    }
  },
  created() {
    this.$http.get('/api/goods').then(response => {
      response = response.body
      if (response.errno === ERR_OK) {
        this.goods = response.data
        this.$nextTick(() => {
          this._initScroll()
        })
      }
    })
  },
  methods: {
    _initScroll() {
      console.log('_initScroll')
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {})
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3
      })
    },
    cartadd: function (target, fooded) {
      console.log(fooded)
      this._drop(target)
    },
    empty: function() {
      console.log('empty')
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          Vue.set(food, 'count', 0)
        })
      })
    },
    _drop(target) {
      // 调用子组件shopcart的方法，并且把target传进去
      this.$refs.shopcart.drop(target)
    },
    selectFood(food, event) {
      if (!event._constructed) {
        return
      }
      this.selectedFood = food
      this.$refs.food.show()
    }
  },
  computed: {
    // 在子组件cartcontrol组件改变food的值，只要父组件接受这个数据响应式变化，那么在父组件一切基于摄者数据的响应式
    // 都会发生变化，自动触发父组件的计算属性，计算属性发生变化，就响应式传入shopcart组件
    selectFoods() {
      let foods = []
      // this.$refs.shopcart.new()
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food)
          }
        })
      })
      return foods
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin"
.goods
  display: flex
  position: absolute
  top: 174px
  bottom: 46px
  width: 100%
  overflow: hidden
  .menu-wrapper
    flex: 0 0 80px
    background: #f3f5f7
    .menu-item
      display: table
      height: 54px
      width: 56px
      padding: 0 12px
      line-height: 14px
      .icon
        display: inline-block
        vertical-align: top
        width: 12px
        height: 12px
        margin-right: 4px
        background-size: 12px 12px
        background-repeat: no-repeat
        &.decrease
          bg-image('decrease_3')
        &.discount
          bg-image('discount_3')
        &.guarantee
          bg-iamge('guarantee_3')
        &.invoice
          bg-image('invoice_3')
        &.special
          bg-image('special_3')
      .text
        display: table-cell
        width: 56px
        vertical-align: middle
        font-size: 12px
  .foods-wrapper
    flex: 1
    .food-list
      list-style: none
      .title
        padding-left: 14px
        height: 26px
        font-size: 12px
        color: rgb(147, 153, 159)
        background: #f3f5f7
        border-left: 2px soild #d9dde1
        line-height: 26px
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          margin-right: 10px
        .content
          flex: 1
          .name
            margin: 2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color: rgb(7, 17, 27)
          .desc, .extra
            line-height: 10px
            font-size: 10px
            color: rgb(147, 153, 159)
          .desc
            line-height: 12px
            margin-bottom: 8px
          .extra
            .count
              margin-right: 12px
          .price
            font-weight: 700
            line-height: 24px
            .now
              margin-right: 8px
              font-size: 14px
              color: rgb(240, 20, 20)
            .old
              text-decoration: line-through
              font-size: 10px
              color: rgb(147, 153, 159)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 12px
            height: 50px
            width: 100px
</style>
