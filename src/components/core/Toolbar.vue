<template lang="pug">
  v-toolbar(
    app
    clipped-right
    color="primary"
    dark
    fixed
    height="58"
    extension-height="48"
    :flat="isHome"
    :manual-scroll="isManualScrolled"
    ref="toolbar"
  )#app-toolbar
    router-link(:to="logoLink").d-flex
      img(
        src="https://cdn.vuetifyjs.com/images/logos/v-alt.svg"
        height="38px"
        width="38px"
      )

    v-fade-transition(mode="out-in")
      v-toolbar-title.pb-1.hidden-xs-only Vuetify for Designers

</template>

<script>

  // Utilities
  import {
    mapGetters,
    mapMutations,
    mapState
  } from 'vuex'
  import languages from '@/data/i18n/languages.json'

  export default {
    data: vm => ({}),
    computed: {
      ...mapGetters('store', ['storeSale']),
      ...mapState('translation', [
        'isTranslating'
      ]),
      ...mapState('app', [
        'appToolbar',
        'isFullscreen',
        'releases',
        'stateless',
        'currentVersion'
      ]),
      ...mapState(['route']),
      logoLink () {
        return this.isHome
          ? { name: 'getting-started/QuickStart' }
          : { name: 'home/Home' }
      }
    },

    methods: {
      ...mapMutations({
        showCreateDialog: 'translation/SHOW_CREATE_DIALOG'
      })
    }
  }
</script>

<style lang="stylus">
  #app-toolbar
    .v-toolbar__title
      margin-left .5em
      font-weight 300
      font-size 21px
      position relative
      top 1px

    .v-toolbar__items
      .v-btn
        text-transform capitalize
        font-size 16px
        font-weight 300

    .v-toolbar__extension
      padding: 0
</style>
