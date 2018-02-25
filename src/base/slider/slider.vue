<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot>
      </slot>
    </div>
    <div class="dots">
      <span class="dot" v-for="(item, index) in dots" :key="index" :class="{active:currentPageIndex == index}"></span>
    </div>
  </div>
</template>

<script>
  import {addClass} from 'common/js/dom'
  import BScroll from 'better-scroll'

  export default {
    data() {
      return {
        dots: [],
        currentPageIndex: 0
      }
    },
    props: {
//      是否在播放
      loop: {
        type: Boolean,
        default: true
      },
//      是否循环播放
      autoPlay: {
        type: Boolean,
        default: true
      },
//      轮播的时间间隔
      interval: {
        type: Number,
        default: 4000
      }
    },

    mounted() {
      setTimeout(() => {
        this._setSliderWidth()
        this._initDots()
        this._initSlider()
//      如果是循环播放的话就调用播放的方法
        if (this.autoPlay) {
          this._play()
        }
      }, 20)
//   监听窗口改变的,来改变轮图的宽度
      window.addEventListener('resize', () => {
//        当slider没有发生改变的时候什么都不做
        if (!this.slider) {
          return
        }
//        当从新resize的时候 不需要在给wdth添加两倍的宽度
        this._setSliderWidth(true)
//        BScroll里的一个接口,当宽度发生变化从新刷新slider
        this.slider.refresh()
      })
    },
    methods: {
      _setSliderWidth(isResize) {
        this.children = this.$refs.sliderGroup.children
        let width = 0
        let sliderWidth = this.$refs.slider.clientWidth
        for (var i = 0; i < this.children.length; i++) {
          let child = this.children[i]
          addClass(child, 'slider-item')
          child.style.width = sliderWidth + 'px'
          width += sliderWidth
        }
//        当从新resize的时候 不需要在给wdth添加两倍的宽度
        if (this.loop && !isResize) {
          width += 2 * sliderWidth
        }
        this.$refs.sliderGroup.style.width = width + 'px'
      },
//    定义个一方法 数组长度为加载出图片个数
      _initDots() {
        this.dots = new Array(this.children.length)
      },
      _initSlider() {
        this.slider = new BScroll(this.$refs.slider, {
          scrollX: true,
          scrollY: false,
          momentum: false,
          snap: true,
          snapLoop: this.loop,
          snapThreshold: 0.3,
          snapSpeed: 400,
          click: true
        })

//        为slider绑定一个滚动时间,事件是BScroll中自带的
        this.slider.on('scrollEnd', () => {
//          用BScroll 里面的方法获取在当前页面的位置
          let pageIndex = this.slider.getCurrentPage().pageX
//          如果在循环情况下让pageIndex-1,
          if (this.loop) {
            pageIndex -= 1
          }
          this.currentPageIndex = pageIndex
          if (this.autoPlay) {
            clearTimeout(this.timer)
            this._play()
          }
        })
      },
//    定义循环播放的方法
      _play() {
//      声明一个变量pageIndex来保存当前位置,因为我们复制多添加了两个元素,所以要+1
        let pageIndex = this.currentPageIndex + 1
        if (this.loop) {
          pageIndex += 1
        }
        this.timer = setTimeout(() => {
          this.slider.goToPage(pageIndex, 0, 400)
        }, this.interval)
      }
    },
    destroyed() {
      clearTimeout(this.timer)
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .slider
    min-height: 1px
    .slider-group
      position: relative
      overflow: hidden
      white-space: nowrap
      .slider-item
        float: left
        box-sizing: border-box
        overflow: hidden
        text-align: center
        a
          display: block
          width: 100%
          overflow: hidden
          text-decoration: none
        img
          display: block
          width: 100%
    .dots
      position: absolute
      right: 0
      left: 0
      bottom: 12px
      text-align: center
      font-size: 0
      .dot
        display: inline-block
        margin: 0 4px
        width: 8px
        height: 8px
        border-radius: 50%
        background: $color-text-l
        &.active
          width: 20px
          border-radius: 5px
          background: $color-text-ll
</style>
