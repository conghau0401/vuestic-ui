<template>
  <va-dropdown
    :position="position"
    :disabled="disabled"
    class="va-select__dropdown"
    :max-height="maxHeight"
    keepAnchorWidth
    ref="dropdown"
    :fixed="fixed"
    :style="{width}"
    :closeOnAnchorClick="false"
    boundaryBody
    tabindex="0"
    @mousedown.native="hasMouseDown = true"
    @focus.native="onFocus"
    @blur.native="(isKeyboardFocused = false, hasMouseDown = false)"
    @keydown.native.enter.prevent.stop="openDropdown"
    @keydown.native.space.prevent.stop="openDropdown"
    @keydown.native.esc.prevent.stop="closeDropdown"
    @keydown.native.down.prevent.stop="focusFirstOption"
    @keydown.native.up.prevent.stop="focusLastOption"
    @keydown.native.tab="closeDropdown"
    @keydown.native.backspace.prevent.stop="clear"
    @mouseleave.native="updateHoverState(false)"
    @mouseover.native="updateHoverState(true)"
  >
    <va-input
      v-if="searchable"
      :placeholder="placeholder"
      v-model="search"
      class="va-select__input"
      ref="search"
      removable
      @keydown.native.tab.exact.prevent.stop="focusFirstOption"
      @keydown.native.shift.tab.exact.prevent.stop="closeDropdown"
      @keydown.native.down.exact.prevent.stop="focusFirstOption"
      @keydown.native.up.exact.prevent.stop="focusLastOption"
    />
    <ul
      class="va-select__option-list"
      :style="optionsListStyle"
    >
      <li
        v-for="option in filteredOptions"
        :key="getKey(option)"
        :class="getOptionClass(option)"
        :style="getOptionStyle(option)"
        @click.stop="selectOption(option)"
        @mouseleave="updateHoveredOption(null)"
        @mouseenter="updateHoveredOption(option)"
        tabindex="0"
        ref="options"
        @focus.prevent.stop="updateHoveredOption(option)"
        @keydown.down.exact.prevent.stop="focusNextOption(option)"
        @keydown.up.exact.prevent.stop="focusPrevOption(option)"
        @keydown.enter.exact.prevent.stop="selectOption(option)"
        @keydown.space.exact.prevent.stop="selectOption(option)"
        @keydown.esc.exact.prevent.stop="closeDropdown"
        @keydown.tab.exact.stop="closeDropdown"
        @keydown.shift.tab.exact.stop="searchable ? focusInput() : closeDropdown()"
      >
        <va-icon v-show="option.icon" :name="option.icon" class="va-select__option__icon"/>
        <span>{{getText(option)}}</span>
        <va-icon
          v-show="isSelected(option)"
          class="va-select__option__selected-icon"
          name="material-icons">
          done
        </va-icon>
      </li>
    </ul>
    <div
      class="va-select__option-list no-options"
      :style="optionsListStyle"
      v-if="!filteredOptions.length"
    >
      {{noOptionsText}}
    </div>

    <div
      slot="anchor"
      :class="selectClass"
      :style="selectStyle"
    >
      <label
        :style="labelStyle"
        aria-hidden="true"
        class="va-select__label"
      >
        {{label}}
      </label>
      <div
        class="va-select__input-wrapper"
        :style="inputWrapperStyles"
      >
        <span v-if="multiple && valueProxy.length <= tagMax">
          <span class="va-select__tag">
            {{ computedSelectTag }}
          </span>
        </span>
        <span v-else-if="displayedText" class="va-select__displayed-text">{{displayedText}}</span>
        <span v-else class="va-select__placeholder">{{placeholder}}</span>
      </div>
      <va-icon
        v-if="showClearIcon"
        class="va-select__clear-icon"
        name="fa fa-times-circle"
        @click.native.stop="clear()"
      />
      <spring-spinner
        :color="$themes.success"
        v-if="loading"
        :size="24"
        class="va-select__loading"
      />
      <va-icon
        class="va-select__open-icon"
        :class="{'va-select__open-icon--keyboard-focused': !disabled && isKeyboardFocused}"
        :name="visible ? 'fa fa-angle-up' : 'fa fa-angle-down'"
      />
    </div>
  </va-dropdown>
</template>

<script>
import VaDropdown from '../va-dropdown/VaDropdown'
import { SpringSpinner } from 'epic-spinners'
import VaIcon from '../va-icon/VaIcon'
import VaInput from '../va-input/VaInput'
import { getHoverColor } from '../../../services/color-functions'
import { KeyboardOnlyFocusMixin } from '../va-checkbox/KeyboardOnlyFocusMixin'

const positions = {
  'top': 'T',
  'bottom': 'B',
}
export default {
  name: 'va-select',
  components: { VaIcon, SpringSpinner, VaDropdown, VaInput },
  data () {
    return {
      search: '',
      hoverState: false,
      mounted: false,
      hoveredOption: null,
    }
  },
  mixins: [KeyboardOnlyFocusMixin],
  props: {
    value: {},
    label: String,
    placeholder: String,
    options: {
      type: Array,
      default: () => [],
    },
    position: {
      type: String,
      default: 'bottom',
      validator: position => Object.keys(positions).includes(position),
    },
    tagMax: {
      type: Number,
      default: 5,
    },
    searchable: Boolean,
    multiple: Boolean,
    disabled: Boolean,
    readonly: Boolean,
    loading: Boolean,
    width: {
      type: String,
      default: '100%',
    },
    maxHeight: {
      type: String,
      default: '128px',
    },
    keyBy: {
      type: String,
      default: 'id',
    },
    textBy: {
      type: String,
      default: 'text',
    },
    clearValue: {
      default: '',
    },
    noOptionsText: {
      type: String,
      default: 'Items not found',
    },
    fixed: {
      type: Boolean,
      default: true,
    },
    noClear: Boolean,
    error: Boolean,
    success: Boolean,
  },
  watch: {
    search (val) {
      this.$emit('update-search', val)
    },
    visible (val) {
      if (val && this.searchable) {
        this.$nextTick(() => {
          this.$refs.search.$refs.input.focus()
        })
      }
    },
  },
  computed: {
    visible () {
      return this.mounted ? this.$refs.dropdown.isClicked : false
    },
    selectClass () {
      return {
        'va-select': true,
        'va-select--multiple': this.multiple,
        'va-select--visible': this.visible,
        'va-select--searchable': this.searchable,
        'va-select--disabled': this.disabled,
        'va-select--loading': this.loading,
      }
    },
    selectStyle () {
      if (this.error) {
        return {
          backgroundColor: getHoverColor(this.$themes.danger),
          borderColor: this.$themes.danger,
        }
      }

      if (this.success) {
        return {
          backgroundColor: getHoverColor(this.$themes.success),
          borderColor: this.$themes.success,
        }
      }

      return {
        backgroundColor: '#f5f8f9',
        borderColor: (this.isKeyboardFocused || this.hoverState) ? null : this.$themes.gray,
      }
    },
    optionsListStyle () {
      return { maxHeight: this.maxHeight }
    },
    displayedText () {
      if (!this.valueProxy) {
        return ''
      }
      if (this.multiple) {
        return this.valueProxy.length ? `${this.valueProxy.length} items selected` : ''
      }
      // We try to find a match from options, if we don't find any - we take value.
      // This way select can display value even when options are not loaded yet.
      const selectedOption = this.valueProxy || this.selectedOption
      const isString = typeof selectedOption === 'string'
      return isString ? selectedOption : selectedOption[this.textBy]
    },
    selectedOption () {
      return (!this.valueProxy || this.multiple) ? null : this.options.find(option => this.compareOptions(option, this.valueProxy)) || null
    },
    filteredOptions () {
      if (!this.search) {
        return this.options
      }

      return this.options.filter(option => {
        const optionText = this.getText(option).toUpperCase()
        const search = this.search.toUpperCase()
        return optionText.includes(search)
      })
    },
    showClearIcon () {
      if (this.noClear) {
        return false
      }
      if (this.disabled) {
        return false
      }
      return this.multiple ? this.valueProxy.length : this.valueProxy !== this.clearValue
    },
    inputWrapperStyles () {
      let paddingRight = 2
      if (this.showClearIcon) {
        paddingRight += 2
      }
      return {
        paddingRight: `${paddingRight}rem`,
        paddingTop: this.label ? this.multiple ? '.59rem' : '.84rem' : 'inherit',
        paddingBottom: this.label ? 0 : this.multiple ? '.3125rem' : '.4375rem',
      }
    },
    labelStyle () {
      if (this.error) return { color: this.$themes.danger }
      if (this.success) return { color: this.$themes.success }
      return { color: this.$themes.primary }
    },
    valueProxy: {
      get () {
        return this.value
      },
      set (value) {
        this.$emit('input', value)
      },
    },
    computedSelectTag () {
      return this.valueProxy.map(val => this.getText(val)).join(', ')
    },
  },
  methods: {
    getOptionClass (option) {
      return {
        'va-select__option': true,
        'va-select__option--selected': this.isSelected(option),
      }
    },
    getOptionStyle (option) {
      return {
        color: this.isSelected(option) ? this.$themes['primary'] : 'inherit',
        backgroundColor: this.isHovered(option) ? getHoverColor(this.$themes['primary']) : 'transparent',
      }
    },
    getText (option) {
      return typeof option === 'string' ? option : option[this.textBy]
    },
    getKey (option) {
      return typeof option === 'string' ? option : option[this.keyBy]
    },
    updateSearch (val) {
      this.search = val
    },
    compareOptions (one, two) {
      // identity check works nice for strings and exact matches.
      if (one === two) {
        return true
      }
      if (typeof this.value === 'string') {
        return false
      }
      return one[this.keyBy] === two[this.keyBy]
    },
    isSelected (option) {
      if (typeof option === 'string') {
        return this.multiple
          ? this.valueProxy.includes(option)
          : this.valueProxy === option
      } else {
        return this.multiple
          ? this.valueProxy.filter(item => item[this.keyBy] === option[this.keyBy]).length
          : this.valueProxy[this.keyBy] === option[this.keyBy]
      }
    },
    isHovered (option) {
      return this.hoveredOption
        ? typeof option === 'string' ? option === this.hoveredOption : this.hoveredOption[this.keyBy] === option[this.keyBy]
        : false
    },
    selectOption (option) {
      this.search = ''
      const isSelected = this.isSelected(option)
      const value = this.value || []

      if (this.multiple) {
        this.valueProxy = isSelected
          ? value.filter(optionSelected => !this.compareOptions(option, optionSelected))
          : [...value, option]
        this.$refs.dropdown.updatePopper()
      } else {
        this.valueProxy = typeof option === 'string' ? option : { ...option }
        this.search = ''
        this.$refs.dropdown.hide()
      }
      if (this.searchable) {
        this.$refs.search.$refs.input.focus()
      }
    },
    clear () {
      this.valueProxy = this.multiple
        ? (Array.isArray(this.clearValue) ? this.clearValue : [])
        : this.clearValue
      this.search = ''
    },
    updateHoverState (hoverState) {
      if (!this.disabled) {
        this.hoverState = hoverState
      }
    },
    updateHoveredOption (option) {
      if (option) {
        this.hoveredOption = typeof option === 'string' ? option : { ...option }
        let indexOfCurrent = this.options.indexOf(option)
        let currentOption = this.$refs.options[indexOfCurrent]
        currentOption && currentOption.focus()
      } else {
        this.hoveredOption = null
      }
    },
    openDropdown (e) {
      if (!this.disabled) {
        this.$refs.dropdown.isClicked = true
      }
    },
    closeDropdown (e) {
      this.$refs.dropdown.hide()
      this.updateHoveredOption(null)
      this.$refs.dropdown.$el.focus()
    },
    focusFirstOption (e) {
      let firstOption = this.$refs.options[0]
      if (!this.$refs.dropdown.isClicked || !firstOption) return
      firstOption.focus()
    },
    focusLastOption (e) {
      let lastOption = this.$refs.options[this.$refs.options.length - 1]
      if (!this.$refs.dropdown.isClicked || !lastOption) return
      lastOption.focus()
    },
    focusNextOption (currentOption) {
      let indexOfCurrent = this.options.indexOf(currentOption)
      let nextOption = this.$refs.options[indexOfCurrent + 1]
      nextOption && nextOption.focus()
    },
    focusPrevOption (currentOption) {
      let indexOfCurrent = this.options.indexOf(currentOption)
      let prevOption = this.$refs.options[indexOfCurrent - 1]
      prevOption && prevOption.focus()
    },
    focusInput () {
      this.updateHoveredOption(null)
      this.$refs.search.$el.focus()
    },
    focusClear (e) {
      e.preventDefault()
      this.$refs.clear.$el.focus()
    },
  },
  mounted () {
    this.mounted = true
  },
}
</script>

<style lang="scss">
@import "../../vuestic-sass/resources/resources";

.va-select {
  cursor: pointer;
  display: flex;
  align-items: flex-end;
  position: relative;
  width: 100%;
  min-height: 2.375rem;
  border-style: solid;
  border-width: 0 0 thin 0;
  border-top-left-radius: 0.5rem;
  border-top-right-radius: 0.5rem;
  margin-bottom: 1rem;
  transition: $transition-primary;

  &--disabled {
    @include va-disabled()
  }

  &--loading {
    .va-select__clear-icon,
    .va-select__open-icon {
      visibility: hidden;
    }
  }

  &__label {
    @include va-title();
    position: absolute;
    top: .125rem;
    left: .5rem;
    margin-bottom: .5rem;
    max-width: calc(100% - .25rem);
    @include va-ellipsis();
    transform-origin: top left;
  }

  &__input-wrapper {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    height: 100%;
    width: 100%;
    justify-content: stretch;
    padding-left: .5rem;
  }

  &__input {
    border: none;
    background: transparent;
    padding: 0.25rem 0;
    font-size: 1rem;
    font-family: $font-family-sans-serif;
    font-weight: normal;
    font-style: normal;
    font-stretch: normal;
    line-height: 1.5;
    letter-spacing: normal;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
    margin: 0 .5rem;

    &:focus {
      outline: none;
    }
  }

  &__displayed-text {
    white-space: nowrap;
    overflow-x: hidden;
    text-overflow: ellipsis;
    width: 100%;
  }
  &__placeholder {
    opacity: .5;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
    width: 100%;
  }

  &__clear-icon {
    color: $va-link-color-secondary;
    display: flex;
    justify-content: center;
    border-radius: 50%;
    width: 1.5rem;
    height: 1.5rem;
    padding: .25rem;
    position: absolute;
    top: 0;
    bottom: 0;
    right: 2rem;
    margin: auto;
  }

  &__open-icon {
    @extend .va-select__clear-icon;
    right: .5rem;

    &--keyboard-focused {
      background-color: rgba(100, 100, 100, 0.1);
    }
  }

  &__tag {
    word-break: break-word;
  }

  &__loading {
    position: absolute;
    right: .5rem;
    top: 0;
    bottom: 0;
    margin: auto;
  }

  &__dropdown {
    outline: none;
    margin: 0;
    padding: 0;
    background: $light-gray3;
    border-radius: .5rem;

    &.va-select__dropdown-position-top {
      box-shadow: 0 -2px 3px 0 rgba(98, 106, 119, 0.25);
    }

    .va-dropdown__anchor {
      display: block;
    }

    .va-dropdown__content {
      background-color: $light-gray3;
      margin: 0;
      padding: 0;
      overflow-y: auto;
      box-shadow: $datepicker-box-shadow;
      border-radius: .5rem;
    }
  }

  &__option-list {
    width: 100%;
    list-style: none;
    &.no-options {
      padding: .5rem;
    }
  }

  &__option {
    cursor: pointer;
    display: flex;
    align-items: center;
    padding: .375rem .5rem .375rem .5rem;
    min-height: 2.25rem;
    word-break: break-word;

    &__selected-icon {
      margin-left: auto;
      font-size: 1.2rem;
    }

    &__icon {
      margin-right: .5rem;
    }
  }
}
</style>
