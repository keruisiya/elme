<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuwrap">
      <ul>
        <li v-for="(item,index) in goods" :key="index" class="menu-item" :class="{'current':currentIndex===index}" @click="selectMenu(index, $event)">
          <span class="text border-1px" >
            <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodwrap">
      <ul>
        <li v-for="(item,index) in goods" :key="index" class="food-list foodlisthook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li v-for="(food,_index) in item.foods" :key="_index" class="food-item border-1px">
              <div class="icon">
                <img :src="food.icon" width="57" height="57">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span>
                  <span>好评率{{food.rating}}%</span>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span>
                    <span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food"></cartcontrol>
                  </div>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart :selectFoods="selectFoods" :deliveryprice="seller.deliveryPrice" :minprice="seller.minPrice"></shopcart>
  </div>
</template>

<script>
import BScroll from 'better-scroll'
import shopcart from '../shopcart/shopcart'
import cartcontrol from '../cartcontrol/cartcontrol'
const ERR_OK = 0

export default {
  name: 'Goods',
  props: {
    seller: {
      type: Object
    }
  },
  components: {
    shopcart,
    cartcontrol
  },
  data () {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0
    }
  },
  computed: {
    // 在类的定义的时候，使用currentIndex===index 来给对应的li加上样式
    currentIndex () {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i]
        let height2 = this.listHeight[i + 1]
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i
        }
      }
      return 0
    },
    selectFoods () {
      let foods = []
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food)
          }
        })
      })
      return foods
    }
  },
  methods: {
    // 跳转的时候传入index就可以了，另外需要打开foodList的click事件，默认better-scorll屏蔽该所有事件
    selectMenu (index, event) {
      if (!event._constructed) {
        return
      }
      let foodList = this.$refs.foodwrap.getElementsByClassName('foodlisthook')
      let el = foodList[index]
      // 300 是动画时间
      this.foodScroll.scrollToElement(el, 300)
    },
    _initScroll () {
      this.menuScroll = new BScroll(this.$refs.menuwrap, {
        click: true
      })
      this.foodScroll = new BScroll(this.$refs.foodwrap, {
        click: true,
        probeType: 3
      })
      // **这里面的pos.y 对应着我们的卷曲高度值。 pos表示卷曲的位置**
      this.foodScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y))
        console.log(this.scrollY, 'foodscroll')
      })
    },
    _calculateHeight () {
      let foodList = this.$refs.foodwrap.getElementsByClassName('foodlisthook')
      let height = 0
      this.listHeight.push(height)
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i]
        height += item.clientHeight
        this.listHeight.push(height)
      }
    }
  },
  created () {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
    this.$http.get('/api/goods').then((response) => {
      response = response.body
      if (response.errno === ERR_OK) {
        this.goods = response.data
        this.$nextTick(() => {
          this._initScroll()
          this._calculateHeight()
        })
        console.log(this.goods)
      }
    })
  }
}

</script>

<style lang="stylus">
@import "../../common/stylus/mixin.styl";
.goods
  display : flex
  position: absolute
  top: 174px
  bottom: 46px
  width: 100%
  overflow: hidden
  .menu-wrapper
    touch-action: none
    flex:0 0 80px
    width: 80px
    background: #f3f5f7
    .menu-item
      display: table
      height: 54px
      width: 56px
      padding: 0 12px
      line-height: 14px
      &.current
        position:relative
        margin-top: -1px
        z-index: 10
        background: #ffffff
        font-weight: 700
        .text
          border-none()
      .icon
        display :inline-block
        vertical-align :top
        width:12px
        height:12px
        margin-right: 2px
        background-size: 12px 12px
        background-repeat :no-repeat
        &.decrease
          bg-image('decrease_3')
        &.discount
          bg-image('discount_3')
        &.guarantee
          bg-image('guarantee_3')
        &.invoice
          bg-image('invoice_3')
        &.special
          bg-image('special_3')
      .text
        display :table-cell
        width:56px
        vertical-align:middle
        font-size:12px
        border-1px(rbga(7,17,27,0.2))
  .foods-wrapper
    touch-action: none
    flex:1
    .title
      padding-left: 14px
      height: 26px
      line-height: 26px
      border-left: 2px solid #d9dde1
      font-size: 12px
      color: rgb(147,153,159)
      background: #f3f5f7
    .food-item
      display: flex
      margin:18px
      padding-bottom: 18px
      border-1px(rbga(7,17,27,0.2))
      &:last-child
        border-none()
        margin-bottom:0
      .icon
        flex: 0 0 57px
        margin-right: 10px
      .content
        flex:1
        .name
          margin:2px 0 8px 0
          height: 14px
          line-height:14px
          font-size 14px
          color: rgb(7,17,27)
        .desc, .extra
          line-height: 10px
          font-size: 10px
          color: rgb(147,153,159)
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
            color: rgb(240,20,20)
          .old
            text-decoration: line-through
            font-size: 10px
            color: rgb(147,153,159)
        .cartcontrol-wrapper
          position: absolute
          right: 0
          bottom: 12px

</style>
