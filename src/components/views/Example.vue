<template lang="pug">
  views-doc(:id="folder")
      section(v-if="usage")#usage
        helpers-example(
          :new-in="usage.new"
          :file="`${folder}/${usage.file}`"
          :inverted="usage.inverted"
          :has-inverted="!usage.uninverted"
          :id="`usage-${-1}`"
          :key="usage.file"
          :desc="usage.desc"
        )

      section(v-if="examples.length > 1")#examples
        template(v-for="(example, i) in examples.slice(1)")
          helpers-example(
            :header="example.header"
            :new-in="example.newIn"
            :file="`${folder}/${example.file}`"
            :inverted="example.inverted"
            :has-inverted="!example.uninverted"
            :id="`example-${camelCaseToDash(example.file)}`"
            :key="example.file"
            :desc="example.desc"
          )

      slot
</template>

<script>
  import api from 'api-generator'
  // Utilities
  import { camel } from '@/util/helpers'

  export default {
    inheritAttrs: false,

    props: {
      data: {
        type: Object,
        default: () => ({})
      }
    },

    data () {
      return {
        api,
        current: null,
        id: '',
        headers: {
          props: [
            { value: 'name', align: 'left', size: 3 },
            { value: 'default', align: 'left', size: 6 },
            { value: 'type', align: 'right', size: 3 }
          ],
          slots: [
            { value: 'name', align: 'left' }
          ],
          scopedSlots: [
            { value: 'name', align: 'left', size: 3 },
            { value: 'props', align: 'right', size: 9 }
          ],
          events: [
            { value: 'name', align: 'left' },
            { value: 'value', align: 'right' }
          ],
          functions: [
            { value: 'name', align: 'left' },
            { value: 'signature', align: 'right' }
          ],
          functional: [
            { value: 'name', align: 'left' },
            { value: 'description', align: 'left' }
          ],
          options: [
            { value: 'name', align: 'left', size: 3 },
            { value: 'default', align: 'left', size: 3 },
            { value: 'type', align: 'right' }
          ]
        },
        search: null,
        tab: null,
        tabs: ['props', 'slots', 'scopedSlots', 'params', 'events', 'functions', 'functional', 'options']
      }
    },

    computed: {
      components () {
        let components = (this.data.components || []).slice()

        if (this.data.component) {
          components.unshift(this.data.component)
        }

        return components
      },
      computedTabs () {
        return this.tabs.filter(tab => (this.currentApi[tab] || []).length > 0)
      },
      currentApi () {
        return this.api[this.current] || {
          props: [],
          slots: [],
          scopedSlots: [],
          params: [],
          events: [],
          functions: [],
          functional: [],
          options: []
        }
      },
      examples () {
        return this.data.examples || []
      },
      folder () {
        return this.data.folder || this.$route.params.component
      },
      namespace () {
        const component = camel(this.$route.params.component)
        const section = camel(this.$route.params.section)

        return `${section}.${component}`
      },
      supplemental () {
        const namespace = `${this.namespace}.supplemental`
        return this.$te(namespace)
          ? this.$t(namespace)
          : this.$te(namespace, 'en')
            ? this.$t(namespace, 'en')
            : []
      },
      usage () {
        return this.examples.slice(0, 1)[0]
      }
    },

    watch: {
      currentApi () {
        const api = this.currentApi[this.tab]

        if (api && api.length) return

        for (const tab of ['props', 'slots', 'options']) {
          if (this.currentApi[tab] && this.currentApi[tab].length > 0) {
            this.tab = tab
            return
          }
        }

        this.tab = ''
      }
    },

    created () {
      if (this.components.length) {
        this.current = this.components[0]
      }
    },

    methods: {
      hasTab (tab) {
        return (this.currentApi[tab] || []).length > 0
      },
      camelCaseToDash (str) {
        return str.replace(/([a-z])([A-Z])/g, '$1-$2').toLowerCase()
      }
    }
  }
</script>
