<template>
  <header class="navbar" ref="navbar" :class="{'shadow': isShowShadow}">
    <div class="inner">
      <SidebarButton @toggle-sidebar="$emit('toggle-sidebar')"/>
      <router-link
        :to="$localePath"
        class="home-link"
      >
        <img
          v-if="$site.themeConfig && $site.themeConfig.logo"
          class="logo"
          :src="$withBase($site.themeConfig.logo.image || '')"
          :alt="$site.themeConfig.logo.text"
        >
        <span
          ref="siteName"
          class="site-name"
          v-if="$site.themeConfig && $site.themeConfig.logo && $site.themeConfig.logo.text"
        >{{ $site.themeConfig.logo.text }}</span>
        <span class="sub-text" v-if="$site.themeConfig && $site.themeConfig.logo && $site.themeConfig.logo.subText">
          {{ $site.themeConfig.logo.subText }}
        </span>
      </router-link>

      <div class="links">
        <AlgoliaSearchBox
          v-if="isAlgoliaSearch && !linksWrapMaxWidth"
          :options="algolia"
        />
        <SearchBox v-else-if="$site.themeConfig.search !== false && $page.frontmatter.search !== false && !linksWrapMaxWidth"/>
        <NavLinks class="can-hide"/>
      </div>
    </div>
  </header>
</template>

<script>
import AlgoliaSearchBox from '@theme/components/AlgoliaSearchBox'
import SearchBox from '@SearchBox'
import SidebarButton from '@theme/components/SidebarButton.vue'
import NavLinks from '@theme/components/NavLinks.vue'

const MOBILE_DESKTOP_BREAKPOINT = 1048 // refer to config.styl

function css (el, property) {
  // NOTE: Known bug, will return 'auto' if style value is 'auto'
  const win = el.ownerDocument.defaultView
  // null means not to return pseudo styles
  return win.getComputedStyle(el, null)[property]
}

export default {
  components: { SidebarButton, NavLinks, SearchBox, AlgoliaSearchBox },

  data () {
    return {
      isShowNavbar: true,
      isShowShadow: false,
      linksWrapMaxWidth: 1
    }
  },

  mounted () {
    // 监听窗口大小改变
    this.listenResize();

    // 监听滚动条改变
    this.listenScroll();
  },

  computed: {
    algolia () {
      return this.$themeLocaleConfig.algolia || this.$site.themeConfig.algolia || {}
    },

    isAlgoliaSearch () {
      return this.algolia && this.algolia.apiKey && this.algolia.indexName
    }
  },
  
  methods: {
    /**
     * 监听窗口大小改变
     */
    listenResize() {
      const NAVBAR_VERTICAL_PADDING = parseInt(css(this.$el, 'paddingLeft')) + parseInt(css(this.$el, 'paddingRight'))
      const handleLinksWrapWidth = () => {
        if (document.documentElement.clientWidth < MOBILE_DESKTOP_BREAKPOINT) {
          this.linksWrapMaxWidth = null
        } else {
          this.linksWrapMaxWidth = this.$el.offsetWidth - NAVBAR_VERTICAL_PADDING
            - (this.$refs.siteName && this.$refs.siteName.offsetWidth || 0)
        }
      }
      handleLinksWrapWidth();
      // 监听窗口大小改变
      window.addEventListener('resize', handleLinksWrapWidth, false);
    },

    /**
     * 监听滚动条
     */
    listenScroll() {
      const handle = (e) => {
        let top = document.documentElement.scrollTop || document.body.scrollTop;
        if (top >= 2) {
          this.isShowShadow = true;
        } else {
          this.isShowShadow = false;
        }
      }
      if (!this.linksWrapMaxWidth) {
        this.isShowShadow = true;
      } else {
        window.addEventListener("scroll", handle);
        handle();
      }
    }
  }
}

</script>

<style lang="stylus">
$navbar-vertical-padding = 0.7rem
$navbar-horizontal-padding = 1.5rem
$MQMobile = 1048px

.navbar
  &.shadow
    box-shadow 0px 3px 10px rgba(0,0,0,0.05)
  .inner
    height 100%
    max-width 1048px
    margin 0 auto
    &:after
     content ""
     display block
     height 0
     clear both
     visibility hidden
    .home-link
      position relative
      height 100%
      line-height 78px
      margin-right 0rem
      padding-right 2.2rem
      float left
      // border-right 1px solid rgba(0, 0, 0, 0.04)
      transition transform .2s
      &:hover
        border none
        transform scale(1.2)

  a, span, img
    display inline-block
  .logo
    position relative
    height $navbarHeight - 2.2rem
    width $navbarHeight - 2.2rem
    vertical-align text-bottom
  .site-name
    height 100%
    font-size 1rem;
    font-weight bold;
    color $textColor
    position relative
  .sub-text
    position absolute
    top 21px
    right 0px
    font-size 12px
    color #ffffff
    line-height 1.2
    background-color darken($accentColor, 10%)
    padding-left 2px
    padding-right 2px
    border-radius 2px
    transform: scale(0.8);
  .links
    height 100%
    box-sizing border-box
    white-space nowrap
    font-size 0.9rem
    float right
    right $navbar-horizontal-padding
    top $navbar-vertical-padding
    display block
    .search-box
      flex: 0 0 auto
      vertical-align top

@media (max-width: $MQMobile)
  .navbar
    padding-left 4rem
    .inner
      .home-link
        border none
        line-height $navbarHeight
        .logo
          display none 
        .sub-text 
          font-size: 1rem;
          display initial;
          position: initial;
    .can-hide
      display none
    .links
      position relative
      float right
      top 0
      .search-box
        padding-left 1rem
        background-color transparent
        position absolute
        right: 0
        top 0.7rem
</style>
