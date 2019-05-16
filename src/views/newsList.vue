<template>
  <div class="news">
      <!-- 新闻列表 -->
      <div class="news-list">
        <div class="news-list-title flex">
          <text class="c3">新闻列表</text>
          <text class="f26 c6" @click="newMoreEvent">更多</text>
        </div>
        <div class="new-list-content">
          <div class="content-item flex-dr" v-for="(item,index) in newsList" :key='index'
            @click='newsListItemEvent(index)'>
            <div class='content-item-left'>
              <bui-image src="/image/test.jpg" radius='10' width="250px" height="150px"></bui-image>
            </div>
            <div class='content-item-right flex-sb'>
              <text
                class="item-right-text lines2">11111111111111111111111111111111111111111111111111111111111111111</text>
              <div class="item-right-date flex">
                <div class="date-origin flex">
                  <bui-image src="/image/test.jpg" radius='10' width="35px" height="35px"></bui-image>
                  <text class="f22">泊云客</text>
                  <text class="f20">03-04</text>
                </div>
                <div class="date-time flex">
                  <text class="f22 c6">10</text>
                  <text class="f22 c6">3</text>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        newsList: ['', '', ''],
      }
    },
    methods: {
      // 更多
      newMoreEvent() {
        this.$alert('消息更多')
      },
      // 新闻列表
      newsListItemEvent(index) {
        this.$alert(index)
      },
    }
  }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
  .news {
    flex: 1;
    background-color: #f2f2f2;
  }

  .news-list {
    background-color: #fff;
    border-radius: 10px;
  }

  .news-list-title {
    padding: 0 20px;
    height: 86px;
  }

  .content-item {
    padding: 20px;
    border-top: 1px solid #eee;
  }

  .content-item-right {
    margin-left: 20px;
    width: 440px;
    flex-direction: column;
  }

  .item-right-text {
    width: 440px;
    color: rgb(165, 164, 164);
  }

  .item-right-date {
    padding-top: 20px;
  }

  .date-origin {
    width: 200px;
  }

  .date-time {
    width: 100px;
    margin-right: 10px;
  }
</style>