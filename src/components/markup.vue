<template lang="pug">
  div(:class="['markup', color]" :data-lang="lang").mb-3
    pre
      code(v-bind:class="lang" ref="markup")
        slot
    div(class="markup__copy")
      v-icon(v-on:click="copyMarkup") content_copy
      v-slide-x-transition
        span(class="component-example-copied" v-if="copied") Copied
</template>

<script>
import highlight from 'highlight.js/lib/highlight.js'
import highlightBash from 'highlight.js/lib/languages/bash'
import highlightYaml from 'highlight.js/lib/languages/yaml'
highlight.registerLanguage('bash', highlightBash)
highlight.registerLanguage('js', highlightYaml)

export default {
  name: 'Markup',
  props: {
    color: {
      type: String,
      default: 'grey lighten-3'
    },
    lang: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      copied: false,
      content: '',
      highlightAttempts: 0
    }
  },
  mounted () {
    const cb = deadline => {
      if (deadline.timeRemaining() < 33.3 && this.highlightAttempts < 3 && !deadline.didTimeout) {
        ++this.highlightAttempts
        requestIdleCallback(cb, {
          timeout: 250
        })
      } else {
        highlight.highlightBlock(this.$refs.markup)
      }
    }
    'requestIdleCallback' in window ? window.requestIdleCallback(cb, { timeout: 500 }) : highlight.highlightBlock(this.$refs.markup)
  },
  methods: {
    copyMarkup () {
      const markup = this.$refs.markup
      markup.setAttribute('contenteditable', 'true')
      markup.focus()
      document.execCommand('selectAll', false, null)
      this.copied = document.execCommand('copy')
      markup.removeAttribute('contenteditable')
      setTimeout(() => {
        this.copied = false
      }, 2000)
    }
  }
}
</script>

<style lang="stylus">
  @import '../../node_modules/vuetify/src/stylus/settings/_colors.styl'
  .markup
    font-size: 1.3rem
    font-family: 'Inconsolata', monospace
    transition: .3s ease-out
    box-shadow: none
    display: flex
    padding: 24px
    border-radius: 2px
    position: relative
    align-items: center
    .component-example-copied
      position: absolute
      top: 12px
      right: 50px
    &__copy
      position: absolute
      right: 0px
      top: 0px
      cursor: pointer
      width: 25px
      height: 25px
      z-index: 1
    &:after
      position: absolute
      right: 10px
      transition: opacity .2s ease-in
      content: attr(data-lang)
      color: rgba(#000, 0.3)
      font-size: 1rem
      font-weight: 700
      top: 5px
    &:hover
      &:after
        opacity: 0
    .icon
      position: absolute
      right: 0
      transition: opacity .2s ease-in
      font-size: 1.5rem
      opacity: 0
      top: 0
      cursor: pointer
      width: 50px
      height: 50px
      z-index: 4
    &:hover
      .icon
        opacity: 1
    pre, code
      background: transparent
      font-family: 'Inconsolata', monospace
      line-height: 1.5
      width: 100%
    code
      position: relative
      box-shadow: none
      overflow-x: auto
      overflow-y: hidden
      word-break: break-word
      flex-wrap: wrap
      align-items: center
      vertical-align: middle
      > div
        width: 100%
      &:before,
      &:after
        display: none
    .hljs
      color: #1867C0
      font-weight: 300
      .hljs-keyword,
      .hljs-attr
        color: #333
        font-weight: 700
      .hljs-tag, .hljs-variable
        color: $purple.lighten-1
      .hljs-string, .hljs-literal, .hljs-number
        color: $red.lighten-1
        font-weight: 400
      .hljs-comment
        color: $grey.base
        font-weight: 300
  .tabs
    .markup
      max-width: 100%
</style>
