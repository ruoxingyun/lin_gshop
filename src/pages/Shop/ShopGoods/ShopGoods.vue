<template>
  <div>
    <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li class="menu-item" v-for="(shopGood, index) in shopGoods" :key="index"
              :class="{current: index===currentIndex}" @click="clickMenuItem(index)">
            <span class="text bottom-border-1px">
              <img class="icon" :src="shopGood.icon" v-if="shopGood.icon" alt="商品图">
              {{ shopGood.name }}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul ref="footUL">
          <li class="food-list-hook" v-for="(shopGood, index) in shopGoods" :key="index"
              :class="{current: index===currentIndex}">
            <h1 class="title">{{ shopGood.name }}</h1>
            <ul>
              <li class="food-item bottom-border-1px" v-for="(food, index) in shopGood.foods"
                  :key="index" @click="showFood(food)">
                <div class="icon">
                  <img width="57" height="57" :src="food.icon" alt="商品图">
                </div>
                <div class="content">
                  <h2 class="name">{{ food.name }}</h2>
                  <p class="desc">{{ food.description }}</p>
                  <div class="extra">
                    <span class="count">月售 {{ food.sellCount }} 份</span>
                    <span>好评率 {{ food.rating }}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{ food.price }}</span>
                    <span class="old" v-if="food.oldPrice">￥{{ food.oldPrice }}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <CartControl :food="food"/>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <ShopCart />
    </div>
    <Food :food="food" ref="food"/>
  </div>
</template>
<script>
import {mapActions, mapState} from 'vuex'
import BScroll from 'better-scroll'
import CartControl from '../../../components/CartControl/CartControl'
import Food from '../../../components/Food/Food'
import ShopCart from '../../../components/ShopCart/ShopCart'

export default {
  data () {
    return {
      scrollY: 0, // 右侧滑动的Y轴坐标 (滑动过程时实时变化)
      tops: [], // 所有右侧分类li的top组成的数组  (列表第一次显示后就不再变化)
      food: {} // 需要显示的food
    }
  },
  mounted () {
    this.getShopGoods(() => {
      this.$nextTick(() => {
        this._initScroll()
        this._initTops()
      })
    })
  },
  computed: {
    ...mapState(['shopGoods']),
    // 计算得到当前分类的下表
    currentIndex () {
      // 得到条件数据
      const {scrollY, tops} = this
      // 根条件计算产生一个结果
      return tops.findIndex((top, index) => {
        return scrollY >= top && scrollY < tops[index + 1]
      })
    }
  },
  methods: {
    ...mapActions(['getShopGoods']),
    // 滚动
    _initScroll () {
      // eslint-disable-next-line no-unused-vars
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      })
      this.foodScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 2
      })

      // 给右侧列表绑定scroll监听
      this.foodScroll.on('scroll', ({y}) => {
        // console.log(x, y)
        this.scrollY = Math.abs(y)
      })
      // 给右侧列表绑定scroll结束的监听
      this.foodScroll.on('scrollEnd', ({x, y}) => {
        // console.log('scrollEnd', x, y)
        this.scrollY = Math.abs(x, y)
      })
    },
    // tops
    _initTops () {
      const tops = []
      let top = 0
      tops.push(top)
      // 找到所有分类的li
      const lis = this.$refs.footUL.getElementsByClassName('food-list-hook')
      Array.prototype.slice.call(lis).forEach(li => {
        top += li.clientHeight
        tops.push(top)
      })
      this.tops = tops
    },

    clickMenuItem (index) {
      // 使用右侧列表滑到对应的位置
      const scrollY = this.tops[index]
      // 立即更新scrollY(让点击的分类项成为当前分类)
      this.scrollY = scrollY
      this.foodScroll.scrollTo(0, -scrollY, 300)
    },

    // 显示点击的food
    showFood (food) {
      this.food = food
      this.$refs.food.toggleShow()
    }
  },
  components: {
    CartControl,
    Food,
    ShopCart
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../../common/stylus/mixins.styl"
.goods
  display: flex
  position: absolute
  top: 195px
  bottom: 46px
  width: 100%
  background: #fff;
  overflow: hidden

  .menu-wrapper
    flex: 0 0 80px
    width: 80px
    background: #f3f5f7

    .menu-item
      display: table
      height: 54px
      width: 56px
      padding: 0 12px
      line-height: 14px

      &.current
        position: relative
        z-index: 10
        margin-top: -1px
        background: #fff
        color: $green
        font-weight: 700

        .text
          border-none()

      .icon
        display: inline-block
        vertical-align: top
        width: 12px
        height: 12px
        margin-right: 2px
        background-size: 12px 12px
        background-repeat: no-repeat

      .text
        display: table-cell
        width: 56px
        vertical-align: middle
        bottom-border-1px(rgba(7, 17, 27, 0.1))
        font-size: 12px

  .foods-wrapper
    flex: 1

    .title
      padding-left: 14px
      height: 26px
      line-height: 26px
      border-left: 2px solid #d9dde1
      font-size: 12px
      color: rgb(147, 153, 159)
      background: #f3f5f7

    .food-item
      display: flex
      margin: 18px
      padding-bottom: 18px
      bottom-border-1px(rgba(7, 17, 27, 0.1))

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
</style>
