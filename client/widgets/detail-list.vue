<script>
import moment from 'moment'

const jsonKeys = ['result', 'input', 'details', 'data', 'Error'],
      preKeys = jsonKeys.concat(['stackTrace', 'details.stackTrace'])

export default {
  name: 'details-list',
  props: ['item', 'highlight', 'compact', 'title'],
  data() {
    return {}
  },
  computed: {
    kvps() {
      var kvps = []

      function flatten(prefix, obj, root) {
        Object.entries(obj).forEach(([k, value]) => {
          if (value == null) return
         var key = prefix ? `${prefix}.${k}` : k

          if (value.routeLink) {
            kvps.push({ key, value: value.text, routeLink: value.routeLink })
          } else if (typeof value === 'object' && !jsonKeys.includes(key)) {
            flatten(key, value, root)
          } else if (key === 'newExecutionRunId') {
            kvps.push({ key, value, routeLink: {
              name: 'execution/summary', params: { runId: value } }
            })
          } else if (key === 'parentWorkflowExecution.runId') {
            kvps.push({ key, value, routeLink: {
              name: 'execution/summary',
              params: {
                domain: root.parentWorkflowDomain,
                workflowId: root.parentWorkflowExecution.workflowId,
                runId: value,
              }
            }})
          } else if (key === 'workflowExecution.runId') {
            kvps.push({ key, value, routeLink: {
              name: 'execution/summary',
              params: {
                domain: root.domain,
                workflowId: root.workflowExecution.workflowId,
                runId: value,
              }
            }})
          } else if (key === 'taskList.name' || key === 'Tasklist') {
            kvps.push({ key, value, routeLink: {
              name: 'task-list', params: { taskList: value } }
            })
          } else if (/.+TimeoutSeconds/.test(key)) {
            kvps.push({ key: key.replace(/Seconds$/, ''), value: moment.duration(value, 'seconds').format() })
          } else {
            kvps.push({ key, value })
          }
        })
      }

      flatten('', this.item || {}, this.item)
      return kvps
    }
  },
  methods: {
    format(val) {
      return val == null ? '' : (String(val) || '""')
    }
  },
  render(h) {
    var { highlight, compact, title } = this
    function dd(kvp) {
      if (kvp.routeLink) {
        return [h('router-link', { props: { to: kvp.routeLink } }, kvp.value)]
      }
      return preKeys.includes(kvp.key) ? [h('data-viewer', {
        props: { item: kvp.value, compact, highlight, title: `${title} - ${kvp.key}` }
      })] : kvp.value
    }

    return h('dl', { class: 'details' }, this.kvps.map(kvp => h('div', { attrs: { 'data-prop': kvp.key } }, [
      h('dt', null, kvp.key),
      h('dd', null, dd(kvp))
    ])))
  }
}
</script>

<style lang="stylus">
@require "../styles/definitions.styl"

dl.details
  > div
    display flex
    padding 4px 0
    justify-content space-between
    min-width 0
    &:nth-child(2n)
      background-color rgba(0,0,0,0.03)
  dt
    flex 0 1 300px
    font-family monospace-font-family
    font-weight 200
    margin-right 1em
  dd
    flex 1 1 auto
    max-width calc(100vw - 700px)
    @media (max-width: 1000px)
      max-width 500px
    one-liner-ellipsis()
</style>