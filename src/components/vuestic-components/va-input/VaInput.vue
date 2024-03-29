<template>
  <va-input-wrapper
    class="va-input"
    :disabled="disabled"
    :success="success"
    :messages="messages"
    :error="error"
    :error-messages="errorMessages"
    :errorCount="errorCount"
  >
    <div
      class="va-input__container"
      :class="{'va-input__container--textarea': isTextarea}"
      :style="containerStyles"
    >
      <div
        v-show="success || error || $slots.prepend || (removable && hasContent)"
        class="va-input__container__icon-wrapper va-input__container__icon-wrapper--start"
      >
        <va-icon
          v-if="success"
          class="va-input__container__icon"
          :style="iconStyles"
          name="check"
        />
        <va-icon
          v-if="error"
          class="va-input__container__icon"
          :style="iconStyles"
          name="warning"
        />
        <div :style="iconStyles">
          <slot name="prepend" />
        </div>
      </div>
      <div
        class="va-input__container__content-wrapper"
        :style="{ paddingTop: label ? '' : '0'}"
      >
        <label
          :style="labelStyle"
          aria-hidden="true"
          class="va-input__container__label"
        >
          {{ label }}
        </label>
        <textarea
          v-if="isTextarea"
          class="va-input__container__input"
          :style="textareaStyles"
          :aria-label="label"
          :placeholder="placeholder"
          :disabled="disabled"
          :readonly="readonly"
          :value="value"
          v-on="inputListeners"
          v-bind="$attrs"
          ref="input"
        />
        <input
          v-else
          class="va-input__container__input"
          :style="{ paddingBottom: label ? '0.125rem' : '0.875rem' }"
          :aria-label="label"
          :type="type"
          :placeholder="placeholder"
          :disabled="disabled"
          :readonly="readonly"
          :value="value"
          v-on="inputListeners"
          v-bind="$attrs"
          ref="input"
        />
      </div>
      <div
        v-if="success || error || $slots.append || (removable && hasContent)"
        class="va-input__container__icon-wrapper va-input__container__icon-wrapper--end"
      >
        <va-icon
          v-if="success"
          class="va-input__container__icon"
          :style="iconStyles"
          name="check"
        />
        <va-icon
          v-if="error"
          class="va-input__container__icon"
          :style="iconStyles"
          name="warning"
        />
        <div :style="iconStyles">
          <slot name="append" />
        </div>
        <va-icon
          v-if="removable && hasContent"
          @click.native="clearContent()"
          class="va-input__container__close-icon"
          name="fa fa-times-circle"
          :style="iconStyles"
        />
      </div>
    </div>
  </va-input-wrapper>
</template>

<script>
import VaInputWrapper from '../va-input/VaInputWrapper'
import VaIcon from '../va-icon/VaIcon'
import {
  getHoverColor,
} from './../../../services/color-functions'
import calculateNodeHeight from './calculateNodeHeight'

export default {
  name: 'va-input',
  extends: VaInputWrapper,
  components: { VaInputWrapper, VaIcon },
  props: {
    value: {
      type: [String, Number],
    },
    label: {
      type: String,
    },
    placeholder: {
      type: String,
    },
    type: {
      type: String,
      default: 'text',
    },
    disabled: {
      type: Boolean,
    },
    readonly: {
      type: Boolean,
    },
    removable: {
      type: Boolean,
    },

    // textarea-specific
    autosize: {
      type: Boolean,
      default: false,
    },
    minRows: {
      type: Number,
      validator: (val) => {
        if (!(val > 0 && (val | 0) === val)) {
          throw new Error(`\`minRows\` must be a positive integer grater than 0, but ${val} is provided`)
        } return true
      },
    },
    maxRows: {
      type: Number,
      validator: (val) => {
        if (!(val > 0 && (val | 0) === val)) {
          throw new Error(`\`maxRows\` must be a positive integer grater than 0, but ${val} is provided`)
        } return true
      },
    },
  },
  mounted () {
    this.adjustHeight()
  },
  watch: {
    value () {
      this.adjustHeight()
    },
  },
  data () {
    return {
      isFocused: false,
    }
  },
  computed: {
    iconStyles () {
      if (this.error) {
        return { color: this.$themes.danger }
      }

      if (this.success) {
        return { color: this.$themes.success }
      }

      return { color: this.$themes.gray }
    },
    labelStyle () {
      if (this.error) return { color: this.$themes.danger }
      if (this.success) return { color: this.$themes.success }
      return { color: this.$themes.primary }
    },
    containerStyles () {
      return {
        backgroundColor:
          this.error ? getHoverColor(this.$themes['danger'])
            : this.success ? getHoverColor(this.$themes['success']) : '#f5f8f9',
        borderColor:
          this.error ? this.$themes.danger
            : this.success ? this.$themes.success
              : this.isFocused ? this.$themes.dark : this.$themes.gray,
      }
    },
    textareaStyles () {
      return {
        paddingBottom: this.label ? '0.125rem' : '',
        marginTop: this.label ? '0.875rem' : '',
        paddingTop: this.label ? 0 : '',
        minHeight: this.label ? '1.5rem' : '2.25rem',
        marginBottom: 0,
      }
    },
    inputListeners () {
      // TODO Probably not the best idea to stick this in computed.
      return Object.assign(
        {},
        this.$listeners,
        {
          input: event => {
            this.$emit('input', event.target.value)
          },
          click: event => {
            this.$emit('click', event)
          },
          focus: event => {
            // eslint-disable-next-line vue/no-side-effects-in-computed-properties
            this.isFocused = true
            this.$emit('focus', event)
          },
          blur: event => {
            // eslint-disable-next-line vue/no-side-effects-in-computed-properties
            this.isFocused = false
            this.$emit('blur', event)
          },
          keyup: event => {
            this.$emit('keyup', event)
          },
          keydown: event => {
            this.$emit('keydown', event)
          },
        },
      )
    },
    hasContent () {
      return ![null, undefined, ''].includes(this.value)
    },
    isTextarea () {
      return this.type === 'textarea'
    },
  },
  methods: {
    adjustHeight () {
      if (!this.autosize || !this.isTextarea) return

      const minRows = this.minRows || 1
      const maxRows = this.maxRows || Number.MAX_SAFE_INTEGER
      const textareaStyles = calculateNodeHeight(this.$refs.input, false, minRows, maxRows)

      // We modify DOM directly instead of using reactivity because the whole adjustHeight method takes place
      // each time the value of textarea is modified, so there's no real need in an additional layer of reactivity.
      // The operation is basically reactive though implicitly.
      Object.assign(this.$refs.input.style, textareaStyles)
    },

    clearContent () {
      this.$emit('input', '')
    },
  },
}
</script>

<style lang='scss'>
@import '../../vuestic-sass/resources/resources';

.va-input {
  &__container {
    display: flex;
    position: relative;
    width: 100%;
    min-height: 2.375rem;
    border-style: solid;
    border-width: 0 0 thin 0;
    border-top-left-radius: 0.5rem;
    border-top-right-radius: 0.5rem;

    &__content-wrapper {
      display: flex;
      position: relative;
      align-items: flex-end;
      width: 100%;
      /*min-width: 100%;*/
    }

    &__icon-wrapper {
      display: flex;
      align-items: center;

      &--start {
        margin-left: 0.5rem;
      }

      &--end {
        margin-right: 0.5rem;
      }
    }

    &__close-icon {
      cursor: pointer;
      margin-left: 0.25rem;
    }

    &__label {
      position: absolute;
      bottom: 0.875rem;
      left: 0.5rem;
      margin-bottom: 0.5rem;
      max-width: calc(100% - 0.25rem);
      color: $vue-green;
      font-size: 0.625rem;
      letter-spacing: 0.0375rem;
      line-height: 1.2;
      font-weight: bold;
      text-transform: uppercase;
      @include va-ellipsis();
      transform-origin: top left;
    }

    &.va-input__container--textarea &__label {
      bottom: auto;
      top: 0.125rem;
    }

    &__input {
      width: 100%;
      height: 1.5rem;
      margin-bottom: 0.125rem;
      padding: 0.25rem 0.5rem;
      color: #34495e;
      background-color: transparent;
      border-style: none;
      outline: none;
      font-size: 1rem;
      font-family: $font-family-sans-serif;
      font-weight: normal;
      font-style: normal;
      font-stretch: normal;
      line-height: 1.5;
      letter-spacing: normal;

      &::placeholder {
        color: $brand-secondary;
      }

      &:placeholder-shown {
        padding-bottom: 0.875rem;
      }
    }

    &.va-input__container--textarea &__input {
      resize: vertical;
      height: inherit;
    }
  }
}
</style>
