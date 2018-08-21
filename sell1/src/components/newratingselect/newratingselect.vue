<template>
  <div class="newratingselect">
    <div class="rating-type border-1px">
      <span @click="selected(2,$event)" class="block positive" :class="{'active':select_Now===2}">
        {{desc.all}}
        <span class="count">{{ratings.length}}</span>
      </span>
      <span @click="selected(0,$event)" class="block positive" :class="{'active':select_Now===0}">
        {{desc.positive}}
        <span class="count">{{positives.length}}</span>
      </span>
      <span @click="selected(1,$event)" class="block negative" :class="{'active':select_Now===1}">
        {{desc.negative}}
        <span class="count">{{negatives.length}}</span>
      </span>
    </div>
    <div @click="toggleContent" class="switch" :class="{'on':onlyContent_now}">
      <span class="icon-check_circle"></span>
      <span class="text" @click="toggleContent">只看该内容的评价</span>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
const POSITIVE = 0
const NEGATIVE = 1
const ALL = 2
export default {
  props: {
    ratings: {
      type: Array,
      default() {
        return []
      }
    },
    selectType: {
      type: Number,
      default: ALL
    },
    onlyContent: {
      type: Boolean,
      default: false
    },
    desc: {
      type: Object,
      default() {
        return {
          all: '全部',
          positive: '满意',
          negative: '不满意'
        }
      }
    }
  },
  data() {
    return {
      select_Now: 2,
      onlyContent_now: true
    }
  },
  watch: {
    selectType: function() {
      // this.select_Now = this.selectType
      // this.$emit('rating_select', this.type)
    },
    onlyContent: function() {
      // this.onlyContent_now = this.onlyContent
      // this.$emit('Content_toggle', this.onlyContent)
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
  methods: {
    selected(type, event) {
      if (!event._constructed) {
        return
      }
      this.select_Now = type
      this.$emit('rating_selected', type)
    },
    toggleContent(event) {
      if (!event._constructed) {
        return
      }
      this.onlyContent_now = !this.onlyContent_now
      this.$emit('content_toggle', this.onlyContent_now)
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .newratingselect
    .rating-type
      padding: 18px 0
      margin: 0 18px
      border-1px(rgba(7, 17, 27, 0.1))
      font-size: 0
      .block
        display: inline-block
        padding: 8px 12px
        margin-right: 8px
        line-height: 16px
        border-radius: 1px
        font-size: 12px
        color: rgb(77, 85, 93)
        &.active
          color: #fff
        .count
          margin-left: 2px
          font-size: 8px
        &.positive
          background: rgba(0, 160, 220, 0.2)
          &.active
            background: rgb(0, 160, 200)
        &.negative
          background: rgba(77, 85, 93, 0.2)
          &.active
            background: rgb(77, 85, 93)
    .switch
      padding: 12px 18px
      line-height: 24px
      border-bottom: 1px soild rgba(7, 17, 27, 0.1)
      color: rgb(147, 153, 159)
      font-size: 0
      &.on
        .icon-check_circle
          color: #00c850
      .icon-check_circle
        display: inline-block
        vertical-align: top
        margin-right: 4px
        font-size: 24px
      .text
        display: inline-block
        vertical-align: top
        font-size: 12px
</style>
