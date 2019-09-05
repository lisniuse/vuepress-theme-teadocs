<template>
  <div class="alert-warp">
    <template v-for="(item, index) in data">
      <div
        :key="index"
        class="warning custom-block teadocs-alert"
        :ref="item.ref"
        v-if="item.elShow"
        :class="{'show': item.isShow}"
      >
        <p class="custom-block-title">{{item.title}}</p>
        <p v-html="item.content"></p>
        <div class="btn-close" @click="close(item)">✕</div>
      </div>
    </template>
  </div>
</template>

<script>
const KEY = 'vuepress/teadocs/alert/elShow/';

export default {
  data() {
    return {
      data: []
    }
  },
  mounted() {
    this.paseAlert();
  },
  methods: {

    /**
     * 解析公告数据
     */
    paseAlert() {
      let array = this.$site.themeConfig.alert || [];
      array.forEach(item => {
        let elShow = localStorage.getItem(`${KEY}${item.id}`);
        if (elShow === null) {
          elShow = true;
        } else {
          elShow = Boolean(Number(elShow));
        }
        this.data.push({
          id: item.id,
          title: item.title,
          content: item.content,
          ref: `alert_${item.id}`,
          elShow: elShow,
          isShow: false
        });
      });
      this.showAll();
    },

    close(item) {
      localStorage.setItem(`${KEY}${item.id}`, '0');
      this.hide(item);
    },

    showAll() {
      let index = -1;
      let timer = window.setInterval(() => {
        index++;
        let item = this.data[index];
        if (!item) {
          window.clearInterval(timer);
        } else {
          if (item.elShow === true) {
            item.isShow = true;
          }
        }
      }, 500);
    },

    hide(item) {
      item.isShow = false;
    }
  },
}
</script>

<style lang="stylus">
$MQMobile = 1048px

.alert-warp
  position fixed
  right 0px
  top 60px
  z-index 100

  .teadocs-alert
    margin-bottom 10px
    position relative
    visibility hidden
    right -501px
    width 500px
    box-shadow 0 5px 10px rgba(0, 0, 0, 0.2)
    background-color #fff8d2 !important
    opacity 0
    transition all 0.4s ease-in-out;

    &.show
      right 0px
      opacity 1
      visibility visible

    .btn-close
      position absolute
      right 20px
      top 10px
      color #2c3e50
      font-size 24px
      cursor pointer
      transition color 0.4s;

      &:hover
        color #000

@media (max-width: $MQMobile)
  .alert-warp
    top $navbarHeight;
    z-index 0
    width 100%
    .teadocs-alert
      width 100%
      margin 0px !important
      box-sizing border-box
      box-shadow none
</style>
