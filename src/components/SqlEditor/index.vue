<template>
  <div class="json-editor">
    <textarea ref="textarea"></textarea>
  </div>
</template>

<script>
  import _ from 'lodash'
  import CodeMirror from 'codemirror'
  import 'codemirror/addon/lint/lint.css'
  import 'codemirror/lib/codemirror.css'
  import 'codemirror/theme/rubyblue.css'
  import 'codemirror/addon/hint/show-hint.css'
  import 'codemirror/addon/lint/lint'
  import 'codemirror/mode/sql/sql'
  import 'codemirror/addon/hint/show-hint'
  import 'codemirror/addon/hint/sql-hint'
  import 'codemirror/addon/hint/anyword-hint'

  export default {
    name: 'sqlEditor',
    data() {
      return {
        sqlEditor: false
      }
    },
    props: ['value'],
    watch: {
      value(value) {
        const editor_value = this.sqlEditor.getValue()
        if (value !== editor_value) {
          this.sqlEditor.setValue(JSON.stringify(this.value, null, 2))
        }
      }
    },
    mounted() {
      this.sqlEditor = CodeMirror.fromTextArea(this.$refs.textarea, {
        lineNumbers: true,
        mode: 'text/x-mysql',
        gutters: ['CodeMirror-lint-markers'],
        extraKeys: {
          'Ctrl': 'autocomplete'
        },
        theme: 'rubyblue',
        lint: true
      })

      this.sqlEditor.setValue(JSON.stringify(this.value, null, 2))
      this.sqlEditor.on('change', cm => {
        this.$emit('changed', cm.getValue())
        this.$emit('input', cm.getValue())
      })

      this.sqlEditor.on('change', (editor, change) => { // 任意键触发autocomplete
        if (change.origin === '+input') {
          const text = change.text
          console.log(text)
          if (_.indexOf([' ', '-', '*', '  '], text[0]) === -1) {
            setTimeout(() => {
              editor.execCommand('autocomplete')
            }, 20)
          }
        }
      })
    },
    methods: {
      getValue() {
        return this.sqlEditor.getValue()
      }
    }
  }
</script>

<style scoped>
  .json-editor {
    height: 100%;
    position: relative;
  }

  .json-editor >>> .CodeMirror {
    height: auto;
    min-height: 300px;
  }

  .json-editor >>> .CodeMirror-scroll {
    min-height: 300px;
  }

  .json-editor >>> .cm-s-rubyblue span.cm-string {
    color: #F08047;
  }
</style>
