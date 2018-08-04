<template>
  <div
    class="expand-block"
    :class="[{ 'hover': hovering }]"
    @mouseenter="hovering = true"
    @mouseleave="hovering = false">
    <slot></slot>
    <div class="meta" ref="meta">
      <div class="expand">
        <slot name="expand"></slot>
      </div>
    </div>
    <div
      class="expand-block-control"
      ref="control"
      :class="{ 'is-fixed': fixedControl }"
      @click="isExpanded = !isExpanded">
      <el-button v-show="!isExpanded" size="middle" type="text" class="control-button">
        {{tip}}
      </el-button>

      <transition name="arrow-slide">
        <i :class="[iconClass, { 'hovering': hovering }]"></i>
      </transition>
      <transition name="text-slide">
        <span v-show="hovering">{{ controlText }}</span>
      </transition>
    </div>
  </div>
</template>

<script type="text/babel">
  export default {
    name: 'expandBlock',
    props: {
      tip: {
        type: String,
        default: ''
      }
    },
    data() {
      return {
        hovering: false,
        isExpanded: false,
        fixedControl: false,
        scrollParent: null
      }
    },
    methods: {
      scrollHandler() {
        const { top, bottom, left } = this.$refs.meta.getBoundingClientRect()
        this.fixedControl = bottom > document.documentElement.clientHeight &&
          top + 44 <= document.documentElement.clientHeight
        this.$refs.control.style.left = this.fixedControl ? `${left}px` : '0'
      },

      removeScrollHandler() {
        this.scrollParent && this.scrollParent.removeEventListener('scroll', this.scrollHandler)
      }
    },

    computed: {
      iconClass() {
        return this.isExpanded ? 'el-icon-caret-top' : 'el-icon-caret-bottom'
      },

      controlText() {
        return this.isExpanded ? this.$t('expandBlock.collapse') : this.$t('expandBlock.expand')
      },

      codeArea() {
        return this.$el.getElementsByClassName('meta')[0]
      },

      codeAreaHeight() {
        return this.$el.getElementsByClassName('expand')[0].clientHeight
      }
    },

    watch: {
      isExpanded(val) {
        this.codeArea.style.height = val ? `${this.codeAreaHeight + 1}px` : '0'
        if (!val) {
          this.fixedControl = false
          this.$refs.control.style.left = '0'
          this.removeScrollHandler()
          return
        }
        setTimeout(() => {
          this.scrollParent = document.querySelector('.page-component__scroll > .el-scrollbar__wrap')
          this.scrollParent && this.scrollParent.addEventListener('scroll', this.scrollHandler)
          this.scrollHandler()
        }, 200)
      }
    },

    mounted() {
      this.$nextTick(() => {
        const expand = this.$el.getElementsByClassName('expand')[0]
        expand.style.width = '100%'
        expand.borderRight = 'none'
      })
    },

    beforeDestroy() {
      this.removeScrollHandler()
    }
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .expand-block {
    border: solid 1px #ebebeb;
    border-radius: 3px;
    transition: .2s;

    &.hover {
      box-shadow: 0 0 8px 0 rgba(232, 237, 250, .6), 0 2px 4px 0 rgba(232, 237, 250, .5);
    }

    code {
      font-family: Menlo, Monaco, Consolas, Courier, monospace;
    }

    .expand-button {
      float: right;
    }

    .meta {
      background-color: #fafafa;
      border-top: solid 1px #eaeefb;
      overflow: hidden;
      height: 0;
      transition: height .2s;
    }

    .expand {
      pre {
        margin: 0;
      }
    }

    .expand-block-control {
      border-top: solid 1px #eaeefb;
      height: 44px;
      box-sizing: border-box;
      background-color: #fff;
      border-bottom-left-radius: 4px;
      border-bottom-right-radius: 4px;
      text-align: center;
      margin-top: -1px;
      color: #d3dce6;
      cursor: pointer;
      position: relative;

      &.is-fixed {
        position: fixed;
        bottom: 0;
        width: 868px;
      }

      i {
        font-size: 16px;
        line-height: 44px;
        transition: .3s;

        &.hovering {
          transform: translateX(-40px);
        }

      }

      > span {
        position: absolute;
        transform: translateX(-30px);
        font-size: 14px;
        line-height: 44px;
        transition: .3s;
        display: inline-block;
      }

      &:hover {
        color: #409EFF;
        background-color: #f9fafc;
      }

      &.text-slide-enter,
      &.text-slide-leave-active {
        opacity: 0;
        transform: translateX(10px);
      }

      .control-button {
        line-height: 26px;
        position: absolute;
        top: 0;
        left: 5px;
        font-size: 14px;
        padding-left: 5px;
        padding-right: 25px;
      }
    }
  }
</style>
