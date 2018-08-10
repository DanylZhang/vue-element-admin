<template>
  <div class="sql-editor">
    <textarea ref="textarea"></textarea>
  </div>
</template>

<script>
  import CodeMirror from 'codemirror'
  import 'codemirror/lib/codemirror.css'
  import 'codemirror/theme/rubyblue.css'
  // mode
  import 'codemirror/mode/sql/sql'
  // lint
  import 'codemirror/addon/lint/lint.css'
  import 'codemirror/addon/lint/lint'
  // hint
  import 'codemirror/addon/hint/show-hint.css'
  import 'codemirror/addon/hint/show-hint'
  import 'codemirror/addon/hint/anyword-hint'
  import 'codemirror/addon/hint/sql-hint'
  // scrollbar
  import 'codemirror/addon/scroll/simplescrollbars.css'
  import 'codemirror/addon/scroll/simplescrollbars'
  // match-highlighter
  import 'codemirror/addon/scroll/annotatescrollbar'
  import 'codemirror/addon/search/matchesonscrollbar'
  import 'codemirror/addon/search/searchcursor'
  import 'codemirror/addon/search/match-highlighter'
  // fullscreen
  import 'codemirror/addon/display/fullscreen.css'
  import 'codemirror/addon/display/fullscreen'

  export default {
    name: 'sqlEditor',
    data() {
      return {
        sqlEditor: false
      }
    },
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
        autofocus: true,
        tabSize: 4,
        styleActiveLine: true,
        lineNumbers: true,
        line: true,
        smartIndent: true,
        mode: 'text/x-mysql',
        scrollbarStyle: 'simple',
        gutters: ['CodeMirror-lint-markers'],
        extraKeys: {
          'Ctrl': 'autocomplete',
          'F11': cm => {
            cm.setOption('fullScreen', !cm.getOption('fullScreen'))
          },
          'Esc': cm => {
            if (cm.getOption('fullScreen')) cm.setOption('fullScreen', false)
          }
        },
        theme: 'rubyblue',
        lint: true,
        highlightSelectionMatches: { showToken: /\w/, annotateScrollbar: true },
        hint: CodeMirror.hint.sql
      })

      if (this.value) {
        this.sqlEditor.setValue(this.value)
      }
      this.sqlEditor.on('change', (editor, change) => { // 任意键触发autocomplete
        // 向父组件发射编辑器内容发生变化的事件
        this.$emit('changed', editor.getValue())
        this.$emit('input', editor.getValue())

        // 自动提示逻辑未完善，暂不启用
        // 如果是增量输入则激活自动完成
        // if (change.origin === '+input') {
        //   const text = change.text
        //   if (_.indexOf([' ', '-', '*', '%', '\'', '"', '  '], text[0]) === -1) {
        //     setTimeout(() => {
        //       editor.execCommand('autocomplete')
        //     }, 20)
        //   }
        // }
      })
    },
    methods: {
      getValue() {
        return this.sqlEditor.getValue()
      },
      getInstance() {
        return this.sqlEditor
      },
      getOption(option) {
        return this.sqlEditor.getOption(option)
      },
      setOption(option, value) {
        this.sqlEditor.setOption(option, value)
      }
    }
  }
</script>

<style ref="stylesheet/scss" lang="scss">
  .sql-editor {
    height: 100%;
    position: relative;

    .CodeMirror {
      min-height: 300px;
      border-top: 1px solid black;
      border-bottom: 1px solid black;
    }

    .CodeMirror-scroll {
      min-height: 300px;
    }

    .cm-s-rubyblue {
      span.cm-string {
        color: #F08047;
      }
    }

    .cm-matchhighlight {
      color: #000;
      background-color: lightgreen
    }

    .CodeMirror-focused {
      .cm-matchhighlight {
        background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAIAAAACCAYAAABytg0kAAAAFklEQVQI12NgYGBgkKzc8x9CMDAwAAAmhwSbidEoSQAAAABJRU5ErkJggg==);
        background-position: bottom;
        background-repeat: repeat-x;
      }
    }

    .CodeMirror-selection-highlight-scrollbar {
      background-color: green
    }
  }

  .CodeMirror-hint-table:before {
    content: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvLTg4NTktMSI/Pg0KPCEtLSBHZW5lcmF0b3I6IEFkb2JlIElsbHVzdHJhdG9yIDE5LjAuMCwgU1ZHIEV4cG9ydCBQbHVnLUluIC4gU1ZHIFZlcnNpb246IDYuMDAgQnVpbGQgMCkgIC0tPg0KPHN2ZyB2ZXJzaW9uPSIxLjEiIGlkPSJDYXBhXzEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHg9IjBweCIgeT0iMHB4Ig0KCSB2aWV3Qm94PSIwIDAgNDgwIDQ4MCIgc3R5bGU9ImVuYWJsZS1iYWNrZ3JvdW5kOm5ldyAwIDAgNDgwIDQ4MDsiIHhtbDpzcGFjZT0icHJlc2VydmUiPg0KPGc+DQoJPGc+DQoJCTxnPg0KCQkJPHBhdGggZD0iTTQ3MiwwSDhDMy41ODIsMCwwLDMuNTgyLDAsOHY0NjRjMCw0LjQxOCwzLjU4Miw4LDgsOGg0NjRjNC40MTgsMCw4LTMuNTgyLDgtOFY4QzQ4MCwzLjU4Miw0NzYuNDE4LDAsNDcyLDB6IE00NjQsNDY0DQoJCQkJSDE2VjgwaDQ0OFY0NjR6IE00NjQsNjRIMTZWMTZoNDQ4VjY0eiIvPg0KCQkJPHJlY3QgeD0iMzIiIHk9IjMyIiB3aWR0aD0iMTYiIGhlaWdodD0iMTYiLz4NCgkJCTxyZWN0IHg9IjY0IiB5PSIzMiIgd2lkdGg9IjE2IiBoZWlnaHQ9IjE2Ii8+DQoJCQk8cmVjdCB4PSI5NiIgeT0iMzIiIHdpZHRoPSIxNiIgaGVpZ2h0PSIxNiIvPg0KCQkJPHBhdGggZD0iTTIwMCwzOTJoODBjNC40MTgsMCw4LTMuNTgyLDgtOHYtNjRjMC00LjQxOC0zLjU4Mi04LTgtOGgtODBjLTQuNDE4LDAtOCwzLjU4Mi04LDh2NjRDMTkyLDM4OC40MTgsMTk1LjU4MiwzOTIsMjAwLDM5MnoNCgkJCQkgTTIwOCwzMjhoNjR2NDhoLTY0VjMyOHoiLz4NCgkJCTxwYXRoIGQ9Ik0yMDAsMjMyaDgwYzQuNDE4LDAsOC0zLjU4Miw4LTh2LTY0YzAtNC40MTgtMy41ODItOC04LThoLTgwYy00LjQxOCwwLTgsMy41ODItOCw4djY0QzE5MiwyMjguNDE4LDE5NS41ODIsMjMyLDIwMCwyMzJ6DQoJCQkJIE0yMDgsMTY4aDY0djQ4aC02NFYxNjh6Ii8+DQoJCQk8cGF0aCBkPSJNMzI4LDM5Mmg4MGM0LjQxOCwwLDgtMy41ODIsOC04di02NGMwLTQuNDE4LTMuNTgyLTgtOC04aC04MGMtNC40MTgsMC04LDMuNTgyLTgsOHY2NEMzMjAsMzg4LjQxOCwzMjMuNTgyLDM5MiwzMjgsMzkyeg0KCQkJCSBNMzM2LDMyOGg2NHY0OGgtNjRWMzI4eiIvPg0KCQkJPHBhdGggZD0iTTMyOCwyMzJoODBjNC40MTgsMCw4LTMuNTgyLDgtOHYtNjRjMC00LjQxOC0zLjU4Mi04LTgtOGgtODBjLTQuNDE4LDAtOCwzLjU4Mi04LDh2NjRDMzIwLDIyOC40MTgsMzIzLjU4MiwyMzIsMzI4LDIzMnoNCgkJCQkgTTMzNiwxNjhoNjR2NDhoLTY0VjE2OHoiLz4NCgkJCTxwYXRoIGQ9Ik03MiwzOTJoODBjNC40MTgsMCw4LTMuNTgyLDgtOHYtNjRjMC00LjQxOC0zLjU4Mi04LTgtOEg3MmMtNC40MTgsMC04LDMuNTgyLTgsOHY2NEM2NCwzODguNDE4LDY3LjU4MiwzOTIsNzIsMzkyeg0KCQkJCSBNODAsMzI4aDY0djQ4SDgwVjMyOHoiLz4NCgkJCTxwYXRoIGQ9Ik03MiwyMzJoODBjNC40MTgsMCw4LTMuNTgyLDgtOHYtNjRjMC00LjQxOC0zLjU4Mi04LTgtOEg3MmMtNC40MTgsMC04LDMuNTgyLTgsOHY2NEM2NCwyMjguNDE4LDY3LjU4MiwyMzIsNzIsMjMyeg0KCQkJCSBNODAsMTY4aDY0djQ4SDgwVjE2OHoiLz4NCgkJPC9nPg0KCTwvZz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjwvc3ZnPg0K);
    display: inline-block;
    width: 10px;
    height: 10px;
  }
</style>
