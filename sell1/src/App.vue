<template>
  <div id="app">
    <v-header :seller="seller"></v-header>
    <div class="tab border-1px">
      <div class="tab-item">
        <router-link class="itemfont" to='/goods'>商品</router-link>
      </div>
      <div class="tab-item">
        <router-link class="itemfont" to='/ratings'>详情</router-link>
      </div>
      <div class="tab-item">
        <router-link class="itemfont" to='/seller'>商家</router-link>
      </div>
    </div>
    <keep-alive>
      <router-view :seller="seller"></router-view>
    </keep-alive>
  </div>
</template>

<script>
import HelloWorld from 'components/HelloWorld'
import header from 'components/header/header.vue'
import {urlParse} from './common/js/util'

const ERR_OK = 0
export default {
  data() {
    return {
      seller: {
        id: (() => {
          // urlParse方法里面会自动获取请求的url
          let queryParam = urlParse()
          console.log(queryParam)
          return queryParam.id
        })()
      }
    }
  },
  name: 'App',
  components: {
    HelloWorld,
    'v-header': header
  },
  created() {
    // 后台会根据我们传入不同的id，会去数据库查找不同的数据
    this.$http.get('/api/seller?id=' + this.seller.id).then(
      (response) => {
        response = response.body
        if (response.errno === ERR_OK) {
          this.seller = response.data
          // 给对象扩展属性的一个方法
          this.seller = Object.assign({}, this.seller, response.data)
        }
      }
    )
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "./common/stylus/mixin.styl"
.tab
  display: flex
  width: 100%
  height: 40px
  line-height: 40px
  border-1px(rgba(7, 17, 27, 0.1))
  .tab-item
    flex: 1
    text-align: center
    .itemfont
      display: block
      font-size: 14px
      text-decoration: none
      color: rgb(77, 85, 93)
      &:hover
        color: red
</style>
