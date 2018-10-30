<template lang="pug">
  section.component-example(:id="id")

    v-card(
      :dark="invertedProxy"
      :class="{ 'elevation-0': readonly }"
    ).mt-2
      //- Example options
      v-divider(v-if="!readonly")

      //- Example mount
      div(data-app :class="exampleClasses").application.application--example.pa-3
        component(:is="component")

</template>

<script>
  import { goTo } from '@/util/helpers'

  const release = process.env.RELEASE

  export default {
    props: {
      active: {
        type: Boolean,
        default: false
      },
      hasInverted: {
        type: Boolean,
        default: false
      },
      file: {
        type: String,
        default: ''
      },
      header: {
        type: String,
        default: ''
      },
      desc: {
        type: String,
        default: ''
      },
      inverted: {
        type: Boolean,
        default: false
      },
      newIn: {
        type: [Boolean, String],
        default: false
      },
      id: {
        type: String,
        default: ''
      },
      readonly: {
        type: Boolean,
        default: false
      }
    },

    data () {
      return {
        tab: 'template',
        tabs: ['template', 'script', 'style'],
        component: null,
        invertedProxy: this.inverted,
        panel: null,
        parsed: {
          script: null,
          style: null,
          template: null,
          codepenDeps: null
        },
        url: release ? `releases/${release}/` : ''
      }
    },

    computed: {
      currentColor () {
        return this.$store.state.currentColor
      },
      iconColor () {
        return 'grey ' + (this.invertedProxy ? 'lighten-1' : 'darken-1')
      },
      exampleClasses () {
        return {
          'theme--dark': this.invertedProxy,
          'theme--light': !this.invertedProxy,
          'grey lighten-3': !this.invertedProxy
        }
      }
    },

    watch: {
      panel () {
        requestAnimationFrame(this.$refs.tabs.callSlider)
      }
    },

    created () {
      if (this.active || this.readonly) {
        this.panel = []
      }
    },

    mounted () {
      import(
        /* webpackChunkName: "examples" */
        /* webpackMode: "lazy-once" */
        `../../examples/${this.file}.vue`
      ).then(comp => {
        this.component = comp.default
      })

      import(
        /* webpackChunkName: "examples-source" */
        /* webpackMode: "lazy-once" */
        `!raw-loader!../../examples/${this.file}.vue`
        ).then(comp => this.boot(comp.default))
    },

    methods: {
      getLang (tab) {
        if (tab === 'script') return 'js'
        if (tab === 'style') return 'css'
        return 'vue'
      },
      parseTemplate (target, template) {
        const string = `(<${target}(.*)?>[\\w\\W]*<\\/${target}>)`
        const regex = new RegExp(string, 'g')
        const parsed = regex.exec(template) || []

        return parsed[1] || ''
      },
      boot (res) {
        const template = this.parseTemplate('template', res)
        const script = this.parseTemplate('script', res)
        const style = this.parseTemplate('style', res)
        const codepenResources = this.parseTemplate('codepen-resources', res)
        const codepenAdditional = this.parseTemplate('codepen-additional', res)

        this.parsed = {
          template,
          script,
          style,
          codepenResources,
          codepenAdditional
        }
      },
      goTo () {
        goTo.call(this, `#${this.id}`)
      },
      toggle () {
        this.active = !this.active
      },
      sendToCodepen () {
        this.$refs.codepen.submit()
      },
      togglePanel () {
        this.panel = this.panel === 0 ? null : 0
        this.$refs.tabs && requestAnimationFrame(() => {
          this.$refs.tabs.callSlider()
        })
      }
    }
  }
</script>

<style lang="stylus">
  @import '~vuetify/src/stylus/settings/_variables.styl'

  #snackbars, #data-tables
    .component-example .application--example
      z-index: auto

  .component-example
    // margin-bottom: 32px

    .application--example
      position: relative
      transition: .3s $transition.swing
      overflow: hidden
      z-index: 0

      > div,
      > form,
      > footer
        width: 100%

    .component-example__panel
      .v-expansion-panel__body
        border: none

      .v-tabs__item, .markup
        height: 100%

      .v-tabs__items
        border: none
        max-height: 500px
        overflow-y: auto

      > li
        border: none

    .justify
      text-align: justify

    aside.v-navigation-drawer,
    .v-overlay
      z-index: 1

    nav.v-toolbar
      z-index: 0
</style>
