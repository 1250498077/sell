<template>
  <transition name="move">
    <div v-show="showFlag" class="food" ref="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span>
            <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food"></cartcontrol>
          </div>
          <transition name="fade">
            <div @click.stop.prevent="addFirst" class="buy" v-show="!food.count || food.count===0">加入购物车</div>
          </transition>
        </div>
        <split v-show="food.info"></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品信息</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <ratingselect :selectType="selectType" :only-content="onlyContent" :desc="desc" :ratings="food.ratings" v-on:ratingtype_select="ratingtype_select" v-on:toggleContent="content_toggle"></ratingselect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li class="rating-item border-1px" v-show="ratingItem_show" v-for="(rating, key) in list" :key="key">
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img :src="rating.avatar" class="avatar" width="12" height="12">
                </div>
                <div class="time">{{rating.rateTime | formatDate}}</div>
                <p class="text">
                  <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
                </p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
import BScroll from 'better-scroll'
import cartcontrol from 'components/cartcontrol/cartcontrol'
import Vue from 'vue'
import {formatDate} from 'common/js/date'
import split from 'components/split/split'
import ratingselect from 'components/ratingselect/ratingselect'
const POSITIVE = 0
const NEGATIVE = 1
const ALL = 2
export default {
  components: {
    cartcontrol,
    split,
    ratingselect
  },
  props: {
    food: {
      type: Object,
      default: function () {
        return {}
      }
    }
  },
  computed: {
    positives() {
      return this.ratings.filter((rating) => {
        return rating.rateType === POSITIVE
      })
    },
    negatives() {
      return this.ratings.filter((rating) => {
        return rating.rateType === NEGATIVE
      })
    }
  },
  data() {
    return {
      showFlag: false,
      selectType: 1,
      onlyContent: false,
      ratingItem_show: true,
      desc: {
        all: '全部',
        positive: '推荐',
        negative: '吐槽'
      },
      list: [],
      typeSelect: 2
    }
  },
  filters: {
    formatDate(time) {
      let date = new Date(time)
      return formatDate(date, 'yyyy-mm-dd hh:mm')
    }
  },
  methods: {
    rating_list(type) {
      this.list.length = 0
      for (let i = 0; i < this.food.ratings.length; i++) {
        if (this.food.ratings[i].rateType === type) {
          this.list.push(this.copy(this.food.ratings[i]))
        }
      }

      this.toggleContentList()
    },
    copy(obj) {
      let newobj = {}
      for (let attr in obj) {
        newobj[attr] = obj[attr]
      }
      return newobj
    },
    toggleContentList: function () {
      if (this.onlyContent) {
        for (let i = 0; i < this.list.length; i++) {
          if (!this.list[i].text) {
            this.list.splice(i, 1)
            i--
          }
        }
      }
    },
    ratingtype_select: function (type) {
      // 保存一下type的值
      this.typeSelect = type
      this.list.length = 0
      this.selectType = type
      if (type !== 2) {
        this.list = this.food.ratings.concat()
        this.rating_list(type)
      }
      if (type === 2) {
        this.list = this.food.ratings.concat()
        this.toggleContentList()
      }
      this.$nextTick(() => {
        this.scroll.refresh()
      })
    },
    content_toggle: function (onlyContent) {
      console.log('content_toggle')
      this.onlyContent = onlyContent
      console.log(this.onlyContent)
      this.ratingtype_select(this.typeSelect)
      this.$nextTick(() => {
        this.scroll.refresh()
      })
    },
    show() {
      this.showFlag = true
      this.selectType = ALL
      this.onlyContent = true
      if (!this.list) {
        this.list = []
      }
      // setInterval(function() {
      //   if (!this.food.ratings) {
      //     console.log('没有数据')
      //   }
      // }, 2000)
      // this.ratingtype_select(2)
      // 为什么在$nextTick方法里面已经调用了ratingtype_select方法，为什么这里还要设置this.list = this.food.ratings.concat()
      // 因为$nextTick只在组件数据变化和组件第一次加载的时候会被调用，而如果我频繁按切换按钮，数据会在第二次就显示不出来，具体原因
      // 我也不清楚，可以注释下面的if里面的代码观察变化
      // if (this.food.ratings) {
      //   this.list = this.food.ratings.concat()
      // }
      this.$nextTick(() => {
        if (!this.scroll) {
          this.selectType = 2
          this.ratingtype_select(2)
          this.scroll = new BScroll(this.$refs.food, {
            click: true,
            probeType: 3
          })
        } else {
          this.scroll.refresh()
        }
      })
    },
    hide() {
      if (!this.list) {
        this.list = []
      }
      this.showFlag = false
    },
    addFirst(event) {
      console.log('click')
      if (!event._constructed) {
        return
      }
      this.$emit('cartadd', event.target)
      Vue.set(this.food, 'count', 1)
    }
  },
  mounted() {
    console.log('asdasdasdasdasddddddddddddddddddddddddddddd')
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin"
.food
  position: fixed
  left: 0
  top: 0
  bottom: 0
  z-index: 60
  width: 100%
  background: #fff
  &.move-enter-active, &.move-leave-active
    transition: all 0.2s linear
    transform: translate3d(0, 0, 0)
  &.move-enter, &.move-leave-to
    transform: translate3d(100%, 0, 0)
  .image-header
    position: relative
    width: 100%
    height: 0
    padding-top: 100%
    img
      position: absolute
      top: 0
      left: 0
      whidth: 100%
      height: 100%
    .back
      position: absolute
      top: 10px
      left: 0
      .icon-arrow_lift
        display: block
        padding: 10px
        font-size: 20px
        color: #fff
  .content
    position: relative
    padding: 18px
    .title
      line-height: 14px
      margin_bottom: 8px
      font-size: 14px
      font-weight: 700
      color: rgb(7, 17, 27)
    .detail
      margin-top: 8px
      margin-bottom: 18px
      line-height: 10px
      font-size: 0px
      .sell-count, .rating
        font-size: 10px
        color: rgb(147, 153, 159)
      .rating
        margin-left: 12px
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
      right: 12px
      bottom: 12px
    .buy
      position: absolute
      right: 18px
      bottom: 18px
      z-index: 10
      height: 24px
      line-height: 24px
      padding: 0 12px
      box-sizing: border-box
      border-radius: 12px
      font-size: 10px
      color: #fff
      background: rgb(0, 160, 220)
      &.fade-enter-active, &.fade-leave-active
        transition: all 0.4s linear
      &.fade-enter, &.fade-leave-to
        opacity: 0
        right: 0px
  .info
    padding: 18px
    .title
      line-height: 14px
      margin-bottom: 6px
      font-size: 14px
      color: rgb(7, 17, 27)
    .text
      line-height: 24px
      padding: 0 8px
      font-size: 12px
      color: rgb(77, 85, 93)
  .rating
    padding-top: 18px
    .title
      line-height: 14px
      margin-left: 18px
      font-size: 14px
      color: rgb(7, 17, 27)
    .rating-wrapper
      padding: 0 18px
      .rating-item
        position: relative
        padding: 16px 0
        border-1px(rgba(7, 17, 27, 0.1))
        .user
          position: absolute
          right: 0
          top: 16px
          line-height: 12px
          font-size: 0
          .name
            display: inline-block
            margin-right: 6px
            vertical-align: top
            font-size: 10px
            color: rgb(147, 153, 159)
          .avatar
            border-radius: 50%
        .time
          margin-bottom: 6px
          line-height: 12px
          font-size: 10px
          color: rgb(147, 153, 159)
        .text
          line-height: 16px
          font-size: 12px
          color: rgb(7, 17, 27)
          .icon-thumb_up, .icon-thumb_down
            margin-right: 4px
            line-height: 4px
            font-size: 12px
          .icon-thumb_up
            color: rgb(0, 160, 220)
          .icon-thumb_down
            color: rgb(147, 153, 159)
      .no-rating
        padding: 16px 0
        font-size: 12px
        color: rgb(147, 153, 159)
</style>
