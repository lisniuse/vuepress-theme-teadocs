<template>
  <aside class="sidebar" :class="{'cursor': cursor === true || !linksWrapMaxWidth}" @mouseenter="cursor=true" @mouseleave="cursor=false">
    <NavLinks/>
    <slot name="top"/>
    <SidebarLinks :depth="0" :items="items"/>
    <slot name="bottom"/>
  </aside>
</template>

<script>
import SidebarLinks from '@theme/components/SidebarLinks.vue'
import NavLinks from '@theme/components/NavLinks.vue'

export default {
  name: 'Sidebar',

  props: ['items'],

  components: {
    SidebarLinks,
    NavLinks
  },

  data() {
    return {
      linksWrapMaxWidth: null,
      cursor: false
    }
  },

  mounted() {
    this.bindSizeChange();
  },

  methods: {
    bindSizeChange() {
      const MOBILE_DESKTOP_BREAKPOINT = 1048 // refer to config.styl
      const handleLinksWrapWidth = () => {
        if (document.documentElement.clientWidth < MOBILE_DESKTOP_BREAKPOINT) {
          this.linksWrapMaxWidth = null;
        } else {
          this.linksWrapMaxWidth = document.documentElement.clientWidth;
        }
      }
      handleLinksWrapWidth()
      window.addEventListener('resize', handleLinksWrapWidth, false)
    },
  },

}
</script>

<style lang="stylus">
$MQMobile = 1048px

.sidebar
  ul
    padding 0
    margin 0
    list-style-type none
  a
    display inline-block
  .nav-links
    display none
    border-bottom 1px solid $borderColor
    padding 0.5rem 0 0.75rem 0
    a
      font-weight 600
    .nav-item, .repo-link
      display block
      line-height 1.25rem
      font-size 1.1em
      padding 0.5rem 0 0.5rem 1.5rem
  & > .sidebar-links
    padding 0 0 2rem 0
    & > li > a.sidebar-link
      font-size 1.1em
      line-height 1.7
      font-weight bold
    & > li:not(:first-child)
      margin-top .75rem

@media (max-width: $MQMobile)
  .sidebar
    .nav-links
      display block
      border-bottom 1px solid #eaecef
      padding .5rem 0 .75rem
      height auto
      .nav-item
        display block
        float none
        line-height 1.25rem
        font-size 1.1em
        padding .5rem 0 .5rem 1.5rem
      .dropdown-wrapper .nav-dropdown .dropdown-item a.router-link-active::after
        top calc(1rem - 2px)
    & > .sidebar-links
      padding 1rem 1rem
</style>
