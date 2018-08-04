<template>
  <div class="sql-editor">
    <textarea ref="textarea"></textarea>
  </div>
</template>

<script>
  import _ from 'lodash'
  import CodeMirror from 'codemirror'
  // codemirror addon lint
  import 'codemirror/addon/lint/lint.css'
  import 'codemirror/addon/lint/lint'
  // codemirror addon hint
  import 'codemirror/addon/hint/show-hint.css'
  import 'codemirror/addon/hint/show-hint'
  import 'codemirror/addon/hint/sql-hint'
  import 'codemirror/addon/hint/anyword-hint'
  // codemirror addon scroll
  import 'codemirror/addon/scroll/simplescrollbars.css'
  import 'codemirror/addon/scroll/simplescrollbars'

  import 'codemirror/mode/sql/sql'
  import 'codemirror/lib/codemirror.css'
  import 'codemirror/theme/rubyblue.css'

  export default {
    name: 'sqlEditor',
    data() {
      return {
        sqlEditor: false
      }
    },
    props: ['value'],
    watch: {
      value(newValue, oldValue) {
        const editor_value = this.sqlEditor.getValue()
        if (newValue !== editor_value) {
          this.sqlEditor.setValue(this.value)
        }
      }
    },
    mounted() {
      this.sqlEditor = CodeMirror.fromTextArea(this.$refs.textarea, {
        lineNumbers: true,
        scrollbarStyle: 'simple',
        smartIndent: true,
        mode: 'text/x-mysql',
        gutters: ['CodeMirror-lint-markers'],
        extraKeys: {
          'Ctrl': 'autocomplete'
        },
        theme: 'rubyblue',
        lint: true
      })

      if (this.value) {
        this.sqlEditor.setValue(this.value)
      }
      this.sqlEditor.on('change', (editor, change) => { // 任意键触发autocomplete
        // 向父组件发射编辑器内容发生变化的事件
        this.$emit('changed', editor.getValue())
        this.$emit('input', editor.getValue())
        // 如果是增量输入则激活自动完成
        if (change.origin === '+input') {
          const text = change.text
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
  .sql-editor {
    height: 100%;
    position: relative;
  }

  .sql-editor >>> .CodeMirror {
    height: auto;
    min-height: 300px;
  }

  .sql-editor >>> .CodeMirror-scroll {
    min-height: 300px;
  }

  .sql-editor >>> .cm-s-rubyblue span.cm-string {
    color: #F08047;
  }
</style>
