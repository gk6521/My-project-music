<template>
  <scroll class="listview"
          :data="data"
          ref="listview"
          :listenScroll="listenScroll"
          :probeType="probeType"
          @scroll="scroll"
  >
    <ul>
      <li class="list-group" v-for="(group, index) in data" :key="index" ref="listGroup">
        <h2 class="list-group-title">{{group.title}}</h2>
        <ul>
          <li class="list-group-item" v-for="(item, index) in group.items" :key="index">
            <img v-lazy="item.avatar" class="avatar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <!--右侧字母-->
    <div class="list-shortcut" @touchstart="onShortcutTouchStart" @touchmove.stop.prevent="onShortcutTouchMove">
      <ul>
        <li class="item"
            v-for="(item, index) in shortcutList"
            :key="index"
            :data-index="index"
            :class="{current: currentIndex===index}"
        >
          {{item}}
        </li>
      </ul>
    </div>
    <div class="list-fixed" v-show="fixedTitle" ref="listFixed">
      <h1 class="fixed-title">{{fixedTitle}}</h1>
    </div>
    <div v-show="!data.length" class="loading-container">
      <loading></loading>
    </div>
  </scroll>
</template>

<script>
  import Scroll from 'base/scroll/scroll'
  import {getData} from 'common/js/dom'
  import Loading from 'base/loading/loading'

  const TOUCH_HEIGHT = 18
  const FIXED_HEIGHT = 30

  export default {
    created() {
//    touch为了保存 获取的y1 和 y2 的位置用来计算距离
      this.touch = {}
      this.listenScroll = true
      this.probeType = 3
      this.listHeight = []
    },
    data() {
      return {
        currentIndex: 0,
        scrollY: -1,
        diff: -1
      }
    },
    props: {
      data: {
        type: Array,
        default: []
      }
    },
//  在计算属性里获取右侧快速入口的列表
    computed: {
      shortcutList() {
        return this.data.map((group) => {
          return group.title.substr(0, 1)
        })
      },
      fixedTitle() {
        if (this.scrollY > 0) {
          return ''
        }
        return this.data[this.currentIndex] ? this.data[this.currentIndex].title : ''
      }
    },
    methods: {
//    手指按下时监听touchstart事件 触发onShortcutTouchStart
      onShortcutTouchStart(e) {
        let anchorIndex = getData(e.target, 'index')
//      获得触碰元素的手指 取第一个
        let firstTouch = e.touches[0]
//      获得触摸点相对于页面的Y轴的位置
        this.touch.y1 = firstTouch.pageY
//        初始化刚开始的下标值
        this.touch.anchorIndex = anchorIndex
        this._scrollTo(anchorIndex)
      },
      // 手机滑动时监听touchmove事件  触发onShortcutTouchMove
      onShortcutTouchMove(e) {
        let firstTouch = e.touches[0]
        this.touch.y2 = firstTouch.pageY
//          获取移动了几个元素                                 | 0 表示向下取整的意思
        let dalta = (this.touch.y2 - this.touch.y1) / TOUCH_HEIGHT | 0
//                       从dom中取出的值是字符串格式,需要转为number类型
        let anchorIndex = parseInt(this.touch.anchorIndex) + dalta
        this._scrollTo(anchorIndex)
      },
//    scroll方法属于子于父间的通讯,带有参数pos   pos中含有x , y 的实时坐标
      scroll(pos) {
        this.scrollY = pos.y
      },
      _scrollTo(index) {
        // 判断点击右侧列表时index的值是否为null
        if (!index && index !== 0) {
          return
        }
        // 判断在滑动右侧列表时  index是否值是否超出范围
        if (index < 0) {
          index = 0
        } else if (index > this.listHeight.length - 2) {
          index = this.listHeight.length - 2
        }
        this.scrollY = -this.listHeight[index]
//      调用BScroll的scrollToElement方法      参数去第几个元素  和动画时长
        this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0)
      },
//      计算list-group的高度
      _calculateHeight() {
        this.listHeight = []
        let list = this.$refs.listGroup
        let height = 0
        this.listHeight.push(height)
//        获得每个listGroup的高度
        for (let i = 0; i < list.length; i++) {
          let item = list[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      }
    },
    watch: {
//    监听props中data数据的变化,在确保dom全部挂载之后调用_calculateHeight
      data() {
        setTimeout(() => {
          this._calculateHeight()
        }, 20)
      },
//     监听scrollY数据的变化,用listHeight的数据与sceollY做对比,来确定listGroup落在哪个区间
      scrollY(newY) {
        let listHeight = this.listHeight

        // 滚动时分为三种情况
        // 第一种情况是 在顶端往下拉的时候 newY是大于0的
        if (newY > 0) {
          this.currentIndex = 0
          return
        }
        // 第二种情况  是在中间滚动的时候
        //          为什么length-1,因为listHeight中的个数要多一个
        for (let i = 0; i < listHeight.length - 1; i++) {
//          分别获取一个高度的上线 和下线
          let height1 = listHeight[i]
          let height2 = listHeight[i + 1]
          if (-newY >= height1 && -newY < height2) {
            this.currentIndex = i
            this.diff = height2 + newY
            return
          }
        }
        // 第三种情况 是在最底部向上拉的时候 且 newY 要大于最有一个元素的上线
        this.currentIndex = listHeight.length - 2
      },
      diff(newVal) {
        let fixedTop = (newVal > 0 && newVal < 30) ? newVal - FIXED_HEIGHT : 0
        if (this.fixedTop === fixedTop) {
          return
        }
        this.fixedTop = fixedTop
        this.$refs.listFixed.style.transform = `translate3d(0, ${this.fixedTop}px,  0)`
      }
    },
    components: {
      Scroll,
      Loading
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .listview
    position: relative
    width: 100%
    height: 100%
    overflow: hidden
    background: $color-background
    .list-group
      padding-bottom: 30px
      .list-group-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
      .list-group-item
        display: flex
        align-items: center
        padding: 20px 0 0 30px
        .avatar
          width: 50px
          height: 50px
          border-radius: 50%
        .name
          margin-left: 20px
          color: $color-text-l
          font-size: $font-size-medium
    .list-shortcut
      position: absolute
      z-index: 30
      right: 0
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding: 20px 0
      border-radius: 10px
      text-align: center
      background: $color-background-d
      font-family: Helvetica
      .item
        padding: 3px
        line-height: 1
        color: $color-text-l
        font-size: $font-size-small
        &.current
          color: $color-theme
    .list-fixed
      position: absolute
      top: 0
      left: 0
      width: 100%
      .fixed-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
    .loading-container
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>
