<template>
<div>
   <div class="shopcart">
    <div class="content" @click="toggleList">
      <div class="content-left">
          <div class="logo-wrapper">
              <div class="logo" :class="{'highlight': totalCount > 0}">
                  <i class="icon-shopping_cart"></i>
              </div>
              <div class="num" v-if="totalCount > 0">
                  {{totalCount}}
              </div>
          </div>
          <div class="price" :class="{'highlight': totalPrice > 0}">{{totalPrice}}元</div>
          <div class="desc">另需配送费{{deliveryPrice}}元</div>
      </div>
      <div class="content-right">
          <div class="pay" :class="payClass" @click.stop="pay">
              {{payDesc}}
          </div>
      </div>
    </div>
    <div class="ball-container">
        <div class="ball-container">
        <div v-for="(ball,index) in balls" :key="index">
          <transition
            @before-enter="beforeDrop"
            @enter="dropping"
            @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
    </div>
    <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
            <div class="list-header">
                <h1 class="title">购物车</h1>
                <span class="empty" @click="empty">清空</span>
            </div>
            <div class="list-content" ref="listContent">
                <ul>
                    <li class="food" v-for="(food, index) in selectFoods" :key="index">
                    <span class="name">{{food.name}}</span>
                    <div class="price">
                        <span>${{food.price * food.count}}</span>
                    </div>
                    <div class="cartcontrol-wrapper">
                        <CartControl :food="food" />
                    </div>
                    </li> 
                </ul>
            </div>
        </div>
    </transition>
  </div>
  <transition name="fade">
      <div class="list-mask" v-show="listShow" @click="toggleList"></div> 
  </transition>
</div>
  
</template>

<script>
const innerClsHook = 'inner-hook'
const BALL_LEN = 10
import CartControl from 'components/cartControl/CartControl.vue'
import BScroll from 'better-scroll'
export default {
    components: {
        CartControl
    },
    data() {
        return {
            balls: [{
                show: false
            },
            {
                show: false
            },
            {
                show: false
            },
            {
                show: false
            },
            {
                show: false
            }],
            dropBalls: [],
            fold: true
        }
    },
    computed: {
        listShow() {
            if(!this.totalCount) {
                this.fold = true
                return false
            } 
            let show = !this.fold
            if(show) {
                this.$nextTick(() => {
                    if(!this.scroll) {
                        this.scroll = new BScroll(this.$refs.listContent, {
                            click: true
                        })
                    }
                    else {
                        this.scroll.refresh()
                    }
                })
                return show
            }
            
        },
        payClass() {
            if(this.totalPrice < this.minPrice) {
                return 'not-enough'
            } else {
                return 'enough'
            }
        },
        payDesc() {
            if(this.totalPrice === 0) {
                return `$${this.minPrice}元起送` 
            }
            else if(this.totalPrice < this.minPrice){
                let diff = this.minPrice - this.totalPrice
                return `还差$${diff}元起送`
            } else {
                return '去结算'
            }
        },
        totalCount() {
            let count = 0
            this.selectFoods.forEach(food => {
                count += food.count
            })
            return count
        },
        totalPrice() {
            let total = 0
            console.log(this)
            this.selectFoods.forEach(food => {
                total += food.price * food.count
            })
            return total
        }
    },
    props: {
        deliveryPrice: {
            type: Number,
            default: 0
        },
        minPrice: {
            type: Number,
            default: 0
        },
        selectFoods: {
            type: Array,
            default(){
                return [{
                    price: 110,
                    count: 1
                }]
            }
        }
    },
    methods: {
        pay($event) {
            if(this.totalPrice < this.minPrice) {
                return
            }
            else {
                window.alert('支付成功！')
                $event.stopPropagation()
            }
        },
        empty() {
            this.selectFoods.forEach(food => {
                food.count = 0
            })
        },
        toggleList() {
            if(!this.totalCount) {
                return
            }
            else {
                this.fold = !this.fold
            }
        },
        drop(el) {
            console.log('el')
            for(let i = 0; i < this.balls.length; i++) {
                let ball = this.balls[i]
                if(!ball.show) {
                    ball.show = true
                    ball.el = el
                    this.dropBalls.push(ball)
                    return
                }
            }
        },
        beforeDrop(el) {
            const ball = this.dropBalls[this.dropBalls.length - 1]
            const rect = ball.el.getBoundingClientRect()
            const x = rect.left - 32
            const y = -(window.innerHeight - rect.top - 22)
            el.style.display = ''
            el.style.transform = el.style.webkitTransform = `translate3d(0,${y}px,0)`
            const inner = el.getElementsByClassName(innerClsHook)[0]
            inner.style.transform = inner.style.webkitTransform = `translate3d(${x}px,0,0)`
        },
        dropping(el, done) {
            this._reflow = document.body.offsetHeight
            el.style.transform = el.style.webkitTransform = `translate3d(0,0,0)`
            const inner = el.getElementsByClassName(innerClsHook)[0]
            inner.style.transform = inner.style.webkitTransform = `translate3d(0,0,0)`
            el.addEventListener('transitionend', done)
        },
        afterDrop(el) {
            const ball = this.dropBalls.shift()
            if (ball) {
            ball.show = false
            el.style.display = 'none'
            }
        },
    }
};
</script>

<style lang="stylus" scoped>
.fold-enter-active, .fold-leave-active
  transition: all .4s cubic-bezier(1.0, 0.5, 0.8, 1.0);


.fold-enter, .fold-leave-to
    opacity 0
    transform: translateY(0) !important;
.shopcart
    .shopcart-list
        position absolute 
        top 0
        z-index -1
        left 0
        width 100%
        transform  translateY(-100%)
        .list-content 
            padding 0 18px
            max-height 217px
            overflow hiden
            background-color #fff
            .food
                position relative 
                padding 12px 0
                box-sizing border-box
                border-bottom 1px solid rgba(7, 17, 27, .1)
                .name
                    line-height 24px
                    font-size 14px
                    color rgb(7,17,27)
                .price 
                    position absolute
                    right 100px
                    bottom 12px
                    line-height 24px
                    font-weight 700
                    font-size 14px
                    color rgb(240, 20, 20)
                .cartcontrol-wrapper
                    position absolute
                    right 0
                    bottom 6px
                    
        .list-header 
            position relative
            z-index 22
            display flex
            justify-content space-between
            height 40px
            line-height 40px
            padding 0 18px
            background-color #f3f7f5
            border-bottom 2px solid rgba(7, 17, 27, .1)
            .title
                font-size 17px
                color rgb(7,17,27)
                font-weight 700
            .empty
                font-size 12px
                color rgb(0,160,220)
                font-weight 700
    .ball-container
        .ball
            position: fixed
            left: 32px
            bottom: 22px
            z-index: 200
            transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
            .inner
                width: 16px
                height: 16px
                border-radius: 50%
                background: rgb(0, 160, 220)
                transition: all 0.4s linear
    position fixed
    left 0
    bottom 0
    z-index 50
    width 100%
    height 48px
    .content 
        display flex
        background-color #141d27
        font-size 0
        height 100%
        .content-left
            flex 1
            .logo-wrapper,
            .price,
            .desc
                display inline-block
                vertical-align top
            .desc
                line-height 24px
                margin 12px 0 0 12px
                font-size 12px
                color rgba(255,255,255,.4)
            .price
                line-height 24px
                margin-top 12px
                box-sizing border-box
                padding-right 12px
                border-right 1px solid rgba(255,255,255,.1)
                font-size 16px
                font-weight 700
                color rgba(255,255,255,0.4)
                &.highlight
                    color #ffffff
            .logo-wrapper
                .num 
                    position absolute
                    top 0
                    right 0
                    width 24px
                    height 16px
                    line-height 16px
                    text-align center
                    border-radius 16px
                    font-size 9px
                    font-weight 700
                    color #ffffff
                    background-color rgb(240,20,20)
                    box-shadow 0 4px 8px 0 rgba(0,0,0,0.4)
                position relative
                top -10px
                margin 0 12px
                padding 6px
                width 56px
                height 56px
                box-sizing border-box
                border-radius 50%
                background-color #141d27
                .logo
                    &.highlight
                        background-color rgb(0, 160, 220)
                    width 100%
                    height 100%
                    border-radius 50%
                    background-color #2b343c
                    text-align center
                    .icon-shopping_cart
                        font-size 24px
                        color #80858a
                        line-height 44px
                    &.highlight .icon-shopping_cart
                        color #ffffff
        .content-right
            flex 0 0 105px
            width 105px
            .pay
                text-align center
                font-size 12px
                font-weight 700
                height 48px
                color rgba(255,255,255,.4)
                line-height 48px
                background-color #2b333b
                &.not-enough
                    background-color #2b333b
                &.enough
                    background-color #00b43c
                    color #ffffff
.list-mask
    position fixed
    top 0
    left 0
    height 100%
    z-index 40
    width 100%
    background-color rgba(7,17,27,.6)
    backdrop-filter blur(10px)
    &.fade-enter-active, &.fade-leave-active
        transition all .4s ease-in-out
    &.fade-enter, &.fade-leave-to
        opacity 0
        background-color rgba(7,17,27,.0)
</style>
