<template>
  <div style="flex:1">
    <news></news>
  </div>
</template>

<!--引入bui-weex样式文件-->
<style lang="scss" src="bui-weex/src/css/buiweex.scss"></style>

<script>
  import news from "./news.vue";
  module.exports = {
    data() {
      return {}
    },
    components: {
      news
    }
  }
</script>

<style>
</style>