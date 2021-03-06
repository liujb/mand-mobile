<template>
  <div class="md-codebox-wrapper">
    <div class="md-codebox" @click="focus">
      <template v-if="maxlength > 0">
        <span
          v-for="i in num"
          :key="i"
          :class="['md-codebox-box', (i === code.length + 1) && focused && 'is-active']"
        >
          <template v-if="code.charAt(i-1)">
            <template v-if="mask">
              <i class="md-codebox-dot"></i>
            </template>
            <template v-else>
              {{code.charAt(i-1)}}
            </template>
          </template>
          <template v-if="i === code.length + 1 && focused">
            <i class="md-codebox-blink"></i>
          </template>
        </span>
      </template>
      <template v-else>
        <input
          v-if="mask"
          type="password"
          :maxlength="maxlength"
          :value="code"
          readonly
          disabled
          :class="['md-codebox-holder', focused && 'is-active']"
        />
        <input
          v-else
          type="text"
          :maxlength="maxlength"
          :value="code"
          readonly
          disabled
          :class="['md-codebox-holder', focused && 'is-active']"
        />
      </template>
    </div>

    <form action="" v-show="system" @submit="$_onSubmit">
      <input
        :value="code"
        :type="mask ? 'password' : 'text'"
        :maxlength="maxlength"
        @input="$_onInputChange"
        ref="input"
        class="md-codebox-input"
      />
    </form>
    <md-number-keyboard
      v-show="!system"
      ref="keyboard"
      class="md-codebox-number-keyboard"
      :type="maxlength > 0 ? 'simple' : 'professional'"
      :okText="okText"
      :disorder="disorder"
      :is-view="isView"
      v-model="focused"
      @delete="$_onDelete"
      @enter="$_onEnter"
      @confirm="$_onConfirm"
    ></md-number-keyboard>
  </div>
</template>

<script>import NumberKeyboard from '../number-keyboard'

export default {
  name: 'md-codebox',
  components: {
    [NumberKeyboard.name]: NumberKeyboard,
  },
  props: {
    value: {
      type: String,
      default: '',
    },
    maxlength: {
      type: [Number, String],
      default: 4,
    },
    autofocus: {
      type: Boolean,
      default: false,
    },
    mask: {
      type: Boolean,
      default: false,
    },
    closable: {
      type: Boolean,
      default: true,
    },
    system: {
      type: Boolean,
      default: false,
    },
    okText: {
      type: String,
    },
    disorder: {
      type: Boolean,
      default: false,
    },
    isView: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      code: '',
      focused: this.autofocus,
    }
  },
  watch: {
    value: {
      imediate: true,
      handler(val) {
        if (val !== this.code) {
          this.code = val
        }
      },
    },
  },
  computed: {
    num() {
      return Math.abs(parseInt(this.maxlength, 10)) || 1
    },
  },
  mounted() {
    if (this.closable) {
      document.addEventListener('click', this.$_handleOutClick)
    }
    if (!this.system && !this.isView) {
      document.body.appendChild(this.$refs.keyboard.$el)
    }
  },
  beforeDestroy() {
    if (this.closable) {
      document.addEventListener('click', this.$_handleOutClick)
    }
  },
  methods: {
    // MARK: private methods

    // MARK: events handler
    $_handleOutClick(e) {
      if (!this.$el.contains(e.target)) {
        this.focused = false
      }
    },
    $_onInputChange(e) {
      if (this.maxlength < 0 || e.target.value.length <= this.maxlength) {
        this.code = e.target.value
      }

      if (this.code.length === this.maxlength) {
        this.blur()
        this.$emit('submit', this.code)
      }

      this.$emit('input', this.code)
    },
    $_onSubmit(e) {
      e.preventDefault()
      this.blur()
      this.$emit('submit', this.code)
    },
    $_onEnter(val) {
      if ((this.maxlength < 0 || this.code.length < this.maxlength) && val !== '.') {
        this.code += val
      }

      if (this.code.length === this.maxlength) {
        this.blur()
        this.$nextTick(function() {
          this.$emit('submit', this.code)
        })
      }

      this.$emit('input', this.code)
    },
    $_onDelete() {
      this.code = this.code.slice(0, this.code.length - 1)
      this.$emit('input', this.code)
    },
    $_onConfirm() {
      this.blur()
      this.$emit('submit', this.code)
    },
    // MARK: public methods
    blur() {
      this.focused = false
      if (this.system) {
        this.$refs.input.blur()
      }
    },
    focus() {
      this.focused = true
      if (this.system) {
        this.$refs.input.focus()
      }
    },
  },
}
</script>

<style lang="stylus">
  .md-codebox-wrapper
    .md-codebox-input
      position absolute
      left -9999px
      opacity 0
  .md-codebox
    position relative
    display flex
    flex-wrap nowrap
    justify-content center
    .md-codebox-box
      display flex
      align-items center
      justify-content center
      width codebox-width
      color color-text-base
      font-size codebox-font-size
      margin-left codebox-gutter
      margin-right codebox-gutter
      hairline(all, codebox-border-color)
      &::before
        content ''
        display block
        padding-bottom 100%
      &::after
        border-radius radius-normal
      &.is-active
        &::after
          border-color codebox-border-active-color
    .md-codebox-blink
      display block
      height 40px
      width 2px
      background-color codebox-blink-color
      animation md-codebox-flash steps(2) 1s infinite
    .md-codebox-dot
      display block
      height 10px
      width: 10px
      line-height 10px
      border-radius 5px
      background-color codebox-dot-color
    .md-codebox-holder
      height codebox-input-height
      padding codebox-input-padding
      width 100%
      text-align center
      font-size codebox-input-font-size
      outline none
      color color-text-base
      letter-spacing 0.1em
      border-radius 0
      border-style solid
      border-width 0 0 2px 0
      border-color codebox-input-border-color
      background none
      box-shadow none
      box-sizing border-box
      -webkit-appearance none
      &[disabled],
      &[readonly]
        color color-text-base
        opacity 1
      &.is-active
        border-color codebox-border-active-color
  @keyframes md-codebox-flash
    from
      opacity 0
    to
      opacity 1
</style>

