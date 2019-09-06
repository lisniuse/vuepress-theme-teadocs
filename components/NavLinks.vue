<template>
  <nav
    class="nav-links"
    v-if="userLinks.length || repoLink"
  >
    <!-- user links -->
    <div
      class="nav-item"
      v-for="item in userLinks"
      :key="item.link"
    >
      <DropdownLink
        v-if="item.type === 'links'"
        :item="item"
      />
      <NavLink
        v-else
        :item="item"
      />
    </div>

    <!-- repo link -->
    <div class="nav-item">
      <a
        v-if="repoLink"
        :href="repoLink"
        class="repo-link"
        target="_blank"
        rel="noopener noreferrer"
      >
        <Github/>
        <span class="text">{{ repoLabel }}</span>
      </a>
    </div>
  </nav>
</template>

<script>
import DropdownLink from '@theme/components/DropdownLink.vue'
import NavLink from '@theme/components/NavLink.vue'
import Github from "@theme/global-components/Github.vue";

import { resolveNavLinkItem } from '../util'

export default {
  components: { 
    NavLink,
    DropdownLink,
    Github
  },

  computed: {
    userNav () {
      return this.$themeLocaleConfig.nav || this.$site.themeConfig.nav || []
    },

    nav () {
      const { locales } = this.$site
      if (locales && Object.keys(locales).length > 1) {
        const currentLink = this.$page.path
        const routes = this.$router.options.routes
        const themeLocales = this.$site.themeConfig.locales || {}
        const languageDropdown = {
          text: this.$themeLocaleConfig.selectText || 'Languages',
          items: Object.keys(locales).map(path => {
            const locale = locales[path]
            const text = themeLocales[path] && themeLocales[path].label || locale.lang
            let link
            // Stay on the current page
            if (locale.lang === this.$lang) {
              link = currentLink
            } else {
              // Try to stay on the same page
              link = currentLink.replace(this.$localeConfig.path, path)
              // fallback to homepage
              if (!routes.some(route => route.path === link)) {
                link = path
              }
            }
            return { text, link }
          })
        }
        return [...this.userNav, languageDropdown]
      }
      return this.userNav
    },

    userLinks () {
      return (this.nav || []).map(link => {
        return Object.assign(resolveNavLinkItem(link), {
          items: (link.items || []).map(resolveNavLinkItem)
        })
      })
    },

    repoLink () {
      const { repo } = this.$site.themeConfig
      if (repo) {
        return /^https?:/.test(repo)
          ? repo
          : `https://github.com/${repo}`
      }
    },

    repoLabel () {
      if (!this.repoLink) return
      if (this.$site.themeConfig.repoLabel) {
        return this.$site.themeConfig.repoLabel
      }

      const repoHost = this.repoLink.match(/^https?:\/\/[^/]+/)[0]
      const platforms = ['GitHub', 'GitLab', 'Bitbucket']
      for (let i = 0; i < platforms.length; i++) {
        const platform = platforms[i]
        if (new RegExp(platform, 'i').test(repoHost)) {
          return platform
        }
      }

      return 'Source'
    }
  }
}
</script>

<style lang="stylus">
$MQMobile = 1048px

.nav-links
  display block
  height 100%
  .nav-item
    > a:not(.repo-link), .dropdown-title
      outline none
      height 100%
      display block !important
      box-sizing border-box
      color inherit
      font-weight normal
      @media (min-width: $MQMobile)
        &
          line-height 80px
      &:hover
        color $accentColor
        font-weight normal
        &:after
          width 40% !important
      &.router-link-active
        color $accentColor
        font-weight 500
        &:after
          width 40% !important
      &:after
        content ""
        display block
        position absolute
        width 0
        left 50%
        -webkit-transform translate(-50%)
        transform translate(-50%)
        height 3px
        border-radius 0 0 2px 2px
        background $accentColor
        top -1px
        transition all .2s ease
  .nav-item
    height 100%
    position relative
    display block
    line-height 2rem
    float left
    .dropdown-wrapper
      &.open
        a
          &.dropdown-title
            margin-bottom 0.5rem
    a
      margin-left 1rem
      margin-right 1rem

  .repo-link
    position relative
    line-height 80px
    display inline !important
    border-radius 20px
    padding 5px
    padding-left 10px
    padding-right 10px
    color #ffffff !important
    background-color $accentColor
    transition all .5s ease
    border none
    &:hover
      border none
      background-color #ffffff !important
      box-shadow 0 2px 4px rgba(0,0,0,0.12)
      color $textColor !important
      svg
        color $textColor
        fill $textColor
    .text
      display inline
      position relative
      top: -1px
    .svg-icon-github
      svg
        position relative
        top: 1px
    svg
      transition all .5s ease
      position relative
      top: -3px
      color #ffffff

@media (max-width: $MQMobile)
  .nav-links
    .repo-link
      background-color $accentColor
      color #ffffff !important
      padding-top 0.2rem !important
      padding-bottom 0.2rem !important
      padding-left 0.5rem !important
      padding-right 0.5rem !important
      font-size 1em !important
      svg
        color #aaa
    a
      line-height 1.4rem !important
      margin-left 0px !important
      &:after
        display none !important
    .nav-item, .repo-link
      margin-left 0

@media (min-width: $MQMobile)
  .nav-links a
    &:hover, &.router-link-active
      color $accentColor
  .nav-item > a:not(.external)
    &:hover, &.router-link-active
      margin-bottom 0px
      border-color lighten($accentColor, 8%)
</style>
