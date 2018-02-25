<template>
  <div class="singer">
    <list-view :data="singers"></list-view>
  </div>
</template>

<script>
  import {getSingerList} from 'api/singer'
  import {ERR_OK} from 'api/config'
  import Singer from 'common/js/singer'
  import ListView from 'base/listview/listview'

  // 声明一个常量 设置为  '热播'
  const HOT_NAME = '热播'
  // 声明常量 保存前十个歌曲
  const HOT_SINGER_LEN = 10

  export default {
    data() {
      return {
        singers: []
      }
    },
    created() {
      this._getSingerList()
    },
    methods: {
      _getSingerList() {
        getSingerList().then((res) => {
          if (res.code === ERR_OK) {
            this.singers = this._normalizeSinger(res.data.list)
          }
        })
      },
//   创建一个方法,对获取的数据进行排序处理
      _normalizeSinger(list) {
//      声明一个对象map 保存title item
        let map = {
          hot: {
            title: HOT_NAME,
            items: []
          }
        }
        list.forEach((el, index) => {
//        将前十首歌压入到map.hot.item中
          if (index < HOT_SINGER_LEN) {
            map.hot.items.push(
//            在common/js/singer中创建一个Singer的类 用于代码重用
              new Singer({
                id: el.Fsinger_mid,
                name: el.Fsinger_name
              })
            )
          }
//        声明一个变量key 保存首字母
          let key = el.Findex
//        如果map里没有key 就压入key属性,在key中保存key和一个保存数据的数组
          if (!map[key]) {
            map[key] = {
              title: key,
              items: []
            }
          }
//        将歌曲数据压入到map[key].item中
          map[key].items.push(new Singer({
            id: el.Fsinger_mid,
            name: el.Fsinger_name
          }))
        })
//      为了得到有序列表,我们将map进行升序的处理
        let ret = []
        let hot = []
        for (let key in map) {
          let val = map[key]
//      验证val.title中是否包含  字母A,B....有就压入到ret数组中
          if (val.title.match(/[a-zA-Z]/)) {
            ret.push(val)
//          验证val.title中是否包含 热播
          } else if (val.title === HOT_NAME) {
            hot.push(val)
          }
        }
//        将ret中的 字母转换成unicode码 按升序排列
        ret.sort((a, b) => {
          return a.title.charCodeAt(0) - b.title.charCodeAt(0)
        })
//      压入到hot数组中  处理成了一个一维的数组 返回出去
        return hot.concat(ret)
      }
    },
    components: {
      ListView
    }
  }

</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  .singer
    position: fixed
    top: 88px
    bottom: 0
    width: 100%
</style>
