<template>
  <div id="app">
    <m-header></m-header>
    <tab></tab>
    <!-- keep-alive 的作用是把DOM 保存在内存中当再次切换进来时不会再次请求-->
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  </div>
</template>

<script type="text/ecmascript-6">
  import MHeader from 'components/m-header/m-header'
  import Tab from 'components/tab/tab'
  export default {
    components: {
      MHeader,
      Tab
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
</style>
