<script>
import StateType from './StateType.vue'

import Vue from 'vue'
import Defer from '@front/mixins/defer'

const keyOrder = {
  props: 1,
  'register module': 1,
  'unregister module': 1,
  mutation: 1,
  setup: 2,
  data: 2,
  state: 2,
  undefined: 3,
  getters: 3,
  computed: 4,
  injected: 5,
  provided: 5,
  'vuex bindings': 5,
  $refs: 6,
  refs: 6,
  $attrs: 7,
  attrs: 7,
  'setup (other)': 8,
}

export default {
  components: {
    StateType,
  },

  mixins: [
    Defer(),
  ],

  props: {
    state: {
      type: Object,
      required: true,
    },

    dimAfter: {
      type: Number,
      default: -1,
    },
  },

  data () {
    return {
      expandedState: {},
      forceCollapse: null,
    }
  },

  computed: {
    dataTypes () {
      return Object.keys(this.state).sort((a, b) => {
        return (
          (keyOrder[a] || (a.toLowerCase().charCodeAt(0) + 999)) -
          (keyOrder[b] || (b.toLowerCase().charCodeAt(0) + 999))
        )
      })
    },

    totalCount () {
      return Object.keys(this.state).reduce((total, state) => total + state.length, 0)
    },

    highDensity () {
      const pref = this.$shared.displayDensity
      return (pref === 'auto' && this.totalCount > 12) || pref === 'high'
    },
  },

  watch: {
    state () {
      this.forceCollapse = null
    },
  },

  methods: {
    toggle (dataType, currentExpanded, event = null) {
      if (event) {
        if (event.ctrlKey || event.metaKey) {
          this.setExpandToAll(false)
          this.$emit('collapse-all')
          return
        } else if (event.shiftKey) {
          this.setExpandToAll(true)
          this.$emit('expand-all')
          return
        }
      }
      Vue.set(this.expandedState, dataType, !currentExpanded)
    },

    setExpandToAll (value) {
      this.dataTypes.forEach(key => {
        this.forceCollapse = value ? 'expand' : 'collapse'
        Vue.set(this.expandedState, key, value)
      })
    },
  },
}
</script>

<template>
  <div class="data-wrapper">
    <template v-for="(dataType, index) in dataTypes">
      <StateType
        v-if="defer(index + 1)"
        :key="dataType"
        :data-type="dataType"
        :index="index"
        :state="state"
        :expanded-state="expandedState"
        :force-collapse="forceCollapse"
        :high-density="highDensity"
        :dim-after="dimAfter"
        @toggle="toggle"
        @edit-state="(path, payload) => $emit('edit-state', path, payload, dataType)"
      />
    </template>
  </div>
</template>
