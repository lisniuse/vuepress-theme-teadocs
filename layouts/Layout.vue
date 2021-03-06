<template>
  <div
    class="theme-container"
    :class="pageClasses"
    @touchstart="onTouchStart"
    @touchend="onTouchEnd"
  >
    <Navbar
      v-if="shouldShowNavbar"
      @toggle-sidebar="toggleSidebar"
    />
 
    <div
      class="sidebar-mask"
      @click="toggleSidebar(false)"
    ></div>

    <Sidebar
      v-if="$page.frontmatter.home && !linksWrapMaxWidth"
      :items="sidebarItems"
      @toggle-sidebar="toggleSidebar"
    >
      <slot
        name="sidebar-top"
        slot="top"
      />
      <slot
        name="sidebar-bottom"
        slot="bottom"
      />
    </Sidebar>

    <Home v-if="$page.frontmatter.home"/>

    <div v-else class="docs-layout"> 

      <div class="search-layout">
        <AlgoliaSearchBox
          v-if="isAlgoliaSearch && linksWrapMaxWidth"
          :options="algolia"
        />
        <SearchBox v-else-if="$site.themeConfig.search !== false && $page.frontmatter.search !== false && linksWrapMaxWidth"/>
      </div>
      
      <Sidebar
        :items="sidebarItems"
        @toggle-sidebar="toggleSidebar"
      >
        <slot
          name="sidebar-top"
          slot="top"
        />
        <slot
          name="sidebar-bottom"
          slot="bottom"
        />
      </Sidebar>

      <Page
        :sidebar-items="sidebarItems"
      >
        <slot
          name="page-top"
          slot="top"
        />
        <slot
          name="page-bottom"
          slot="bottom"
        />
      </Page>
    </div>
    <alert></alert>
    <side-panel v-if="sidePanelConf.enable === true"></side-panel>
  </div>
</template>

<script>
import sidePanel from '@theme/components/sidePanel.vue'
import Home from '@theme/components/Home.vue'
import Navbar from '@theme/components/Navbar.vue'
import Page from '@theme/components/Page.vue'
import Sidebar from '@theme/components/Sidebar.vue'

import AlgoliaSearchBox from '@AlgoliaSearchBox'
import SearchBox from '@SearchBox'

import { resolveSidebarItems } from '../util'

export default {
  components: {
    Home,
    Page,
    Sidebar,
    Navbar,
    AlgoliaSearchBox,
    SearchBox,
    sidePanel
  },

  data () {
    return {
      linksWrapMaxWidth: null,
      isSidebarOpen: false
    }
  },

  computed: {

    sidePanelConf () {
      return this.$site.themeConfig.sidePanel || {
        enable: false
      }
    },

    shouldShowNavbar () {
      const { themeConfig } = this.$site
      const { frontmatter } = this.$page
      if (
        frontmatter.navbar === false
        || themeConfig.navbar === false) {
        return false
      }
      return (
        this.$title
        || themeConfig.logo
        || themeConfig.repo
        || themeConfig.nav
        || this.$themeLocaleConfig.nav
      )
    },

    shouldShowSidebar () {
      const { frontmatter } = this.$page
      return (
        !frontmatter.home
        && frontmatter.sidebar !== false
        && this.sidebarItems.length
      )
    },

    sidebarItems () {
      return resolveSidebarItems(
        this.$page,
        this.$page.regularPath,
        this.$site,
        this.$localePath
      )
    },

    pageClasses () {
      const userPageClass = this.$page.frontmatter.pageClass
      return [
        {
          'no-navbar': !this.shouldShowNavbar,
          'sidebar-open': this.isSidebarOpen,
          'no-sidebar': !this.shouldShowSidebar
        },
        userPageClass
      ]
    },

    algolia () {
      return this.$themeLocaleConfig.algolia || this.$site.themeConfig.algolia || {}
    },

    isAlgoliaSearch () {
      return this.algolia && this.algolia.apiKey && this.algolia.indexName
    }

  },

  mounted () {
    this.$router.afterEach(() => {
      this.isSidebarOpen = false
    });
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

    toggleSidebar (to) {
      this.isSidebarOpen = typeof to === 'boolean' ? to : !this.isSidebarOpen
    },

    // side swipe
    onTouchStart (e) {
      this.touchStart = {
        x: e.changedTouches[0].clientX,
        y: e.changedTouches[0].clientY
      }
    },

    onTouchEnd (e) {
      const dx = e.changedTouches[0].clientX - this.touchStart.x
      const dy = e.changedTouches[0].clientY - this.touchStart.y
      if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > 40) {
        if (dx > 0 && this.touchStart.x <= 80) {
          this.toggleSidebar(true)
        } else {
          this.toggleSidebar(false)
        }
      }
    }
  }
}
</script>

<style src="prismjs/themes/prism-solarizedlight.css"></style>

<style lang="stylus">
$MQMobile = 1048px

.docs-layout
  max-width: 1048px
  margin: 0 auto
  .search-layout
    top 7rem
    padding-left 0rem
    box-sizing border-box
    width 17rem
    position fixed
    z-index 100
    .search-box
      width 100% !important
      box-sizing border-box
      input
        width 14.4rem !important
        border-radius 4px
        border-color #eeeeee
    .suggestions
      border none !important
      background-color #ffffff
      box-shadow 0 5px 10px rgba(0,0,0,0.12)
@media (min-width: $MQMobile)
  .docs-layout
    .search-box
      input
        border none
        background-color lighten($accentColor, 97%)
        color $textColor
        &:focus
          background-color #ffffff
          box-shadow 0 5px 10px rgba(0,0,0,0.06)
</style>
