<template>
  <div style="flex:1">
    <newsList></newsList>
  </div>
</template>

<!--引入bui-weex样式文件-->
<style lang="scss" src="bui-weex/src/css/buiweex.scss"></style>

<script>
  import newsList from "./newsList.vue";
  module.exports = {
    data() {
      return {}
    },
    components: {
      newsList
    }
  }
</script>

<style>
</style>