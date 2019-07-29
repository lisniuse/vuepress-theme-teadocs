<template>
  <div class="warning custom-block teadocs-alert" v-if="$site.themeConfig.alert && isShow" :class="{'show': elShow}">
    <p class="custom-block-title">文档公告</p>
    <p v-html="$site.themeConfig.alert"></p>
    <div class="btn-close" @click="close">✕</div>
  </div>
</template>

<script>
import { setTimeout } from 'timers';
const KEY = 'vuepress/teadocs/alert/isShow';

export default {
  data() {
    return {
      isShow: false,
      elShow: false
    }
  },
  mounted() {
    let isShow = localStorage.getItem(KEY);
    if (isShow === null) {
      this.isShow = true;
    } else if (isShow === true) {
      this.isShow = isShow;
    }
    if (this.isShow === true) {
      this.show();
    }
  },
  methods: {
    close() {
      localStorage.setItem(KEY, '0');
      this.hide();
    },

    show() {
      setTimeout(() => {
        this.elShow = true;
      }, 500);
    },

    hide() {
      this.elShow = false;
    }
  },
}
</script>

<style lang="stylus">
$MQMobile = 918px

.teadocs-alert
  position fixed
  visibility hidden
  z-index 100
  right -501px
  top 60px
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
  .teadocs-alert
    top $navbarHeight;
    z-index 0
    width 100%
    margin 0px !important
    box-sizing border-box
    box-shadow none
</style>
