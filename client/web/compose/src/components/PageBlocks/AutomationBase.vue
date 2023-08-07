<template>
  <wrap
    v-bind="$props"
    v-on="$listeners"
  >
    <div
      v-if="processing"
      class="d-flex align-items-center justify-content-center h-100"
    >
      <b-spinner />
    </div>

    <automation-buttons
      v-else
      class="d-flex flex-wrap my-2"
      button-class="my-1 mx-3 flex-fill"
      :buttons="options.buttons"
      :automation-scripts="automationScripts"
      v-bind="$props"
    />
  </wrap>
</template>
<script>
import base from './base'
import AutomationButtons from './Shared/AutomationButtons'
import axios from 'axios'

export default {
  components: { AutomationButtons },
  extends: base,

  props: {
    extraEventArgs: {
      type: Object,
      default: () => ({}),
    },
  },

  data () {
    return {
      processing: false,

      automationScripts: [],

      cancelTokenSource: axios.CancelToken.source(),
    }
  },

  computed: {
    hasUIHooks () {
      return this.$UIHooks.set && !!this.$UIHooks.set.length
    },
  },

  beforeDestroy () {
    this.abortRequests()
  },

  created () {
    if (this.$UIHooks.set && !!this.$UIHooks.set.length) {
      return
    }

    this.fetchAutomationLists()
  },

  methods: {
    fetchAutomationLists () {
      this.processing = true

      return this.$ComposeAPI
        .automationList(
          { eventTypes: ['onManual'], excludeInvalid: true },
          { cancelToken: this.cancelTokenSource.token },
        )
        .then(({ set = [] }) => {
          this.automationScripts = set
        })
        .finally(() => {
          this.processing = false
        })
    },

    abortRequests () {
      this.cancelTokenSource.cancel(`abort-request-${this.block.blockID}`)
    },
  },
}
</script>
