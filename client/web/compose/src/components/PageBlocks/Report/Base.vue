<template>
  <wrap
    v-bind="$props"
    v-on="$listeners"
    @refreshBlock="refresh"
  >
    <div
      v-if="processing"
      class="d-flex align-items-center justify-content-center h-100"
    >
      <b-spinner />
    </div>

    <display-element
      v-else-if="displayElement"
      :key="key"
      :display-element="displayElement"
      :labels="{
        previous: $t('recordList.pagination.prev'),
        next: $t('recordList.pagination.next'),
      }"
      @update="getDataframes"
    />
  </wrap>
</template>
<script>
import axios from 'axios'
import base from '../base'
import { system, reporter, NoID } from '@cortezaproject/corteza-js'
import DisplayElement from './DisplayElements'

export default {
  name: 'ReportBase',

  components: {
    DisplayElement,
  },

  extends: base,

  data () {
    return {
      processing: false,
      report: undefined,
      displayElement: undefined,
      cancelTokenSource: axios.CancelToken.source(),
    }
  },

  watch: {
    options: {
      deep: true,
      immediate: true,
      handler ({ reportID = NoID }) {
        if (reportID !== NoID) {
          this.fetchReport(reportID)
        }
      },
    },
  },

  beforeDestroy () {
    this.abortRequests()
  },

  created () {
    this.refreshBlock(this.refresh)
  },

  methods: {
    fetchReport (reportID) {
      this.processing = true

      return this.$SystemAPI
        .reportRead({ reportID }, { cancelToken: this.cancelTokenSource.token })
        .then(report => {
          this.report = new system.Report(report)

          return this.getDataframes()
        }).catch((e) => {
          if (!axios.isCancel(e)) {
            this.toastErrorHandler(this.$t('notification:report.fetchFailed'))(e)
          }
        })
        .finally(() => {
          this.processing = false
        })
    },

    getDataframes (definition = {}) {
      const { elementID } = this.options

      if (elementID) {
        const block = this.report.blocks.find(({ elements }) => {
          return elements.some(e => e.elementID === elementID)
        })

        let element = (block.elements || []).find(e => e.elementID === elementID)

        if (element && element.kind !== 'Text') {
          element = reporter.DisplayElementMaker(element)

          const scenarioDefinition = this.getScenarioDefinition(element)

          Object.entries(definition).forEach(([key, value]) => {
            definition[key] = { ...value, ...scenarioDefinition[key] }
          })

          const { dataframes: frames = [] } = element.reportDefinitions({ ...definition, ...scenarioDefinition })

          if (frames.length) {
            return this.$SystemAPI.reportRun({ frames, reportID: this.options.reportID })
              .then(({ frames: dataframes = [] }) => {
                this.displayElement = {
                  ...element,
                  dataframes,
                }
              }).catch((e) => {
                this.toastErrorHandler(this.$t('notification:report.runFailed'))(e)
              })
          }
        } else {
          this.displayElement = element
        }
      }
    },

    getScenarioDefinition (element) {
      const scenarioDefinition = {}
      const { scenarioID } = this.options

      const scenario = this.report.scenarios.find(({ label }) => scenarioID === label)

      // Generate filter for each load datasource
      if (scenario && scenario.filters) {
        element.options.datasources.forEach(({ name }) => {
          scenarioDefinition[name] = {
            ref: name,
            filter: scenario.filters[name] || {},
          }
        })
      }

      return scenarioDefinition
    },

    abortRequests () {
      this.cancelTokenSource.cancel(`cancel-record-list-request-${this.block.blockID}`,)
    },

    refresh () {
      this.fetchReport(this.options.reportID).then(() => {
        this.key++
      })
    },
  },
}
</script>
