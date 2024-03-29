<template>
  <div
    class="va-slider"
    :class="sliderClass"
  >
    <div class="va-slider__input-wrapper" v-if="$slots.beforeInput">
      <slot name="beforeInput"/>
    </div>
    <span
      v-if="label && !inverseLabel"
      :style="labelStyle"
      class="va-slider__label">
      {{ label }}
    </span>
    <span
      v-if="icon"
      class="va-slider__label">
      <va-icon :name="icon" :color="colorComputed" :size="16"/>
    </span>
    <div
      class="va-slider__container"
      @click="wrapClick"
      ref="sliderContainer"
    >
      <div
        class="va-slider__container__track"
        :style="trackStyles"/>
      <template v-if="pins">
        <div
          v-for="(pin, key) in pinsCol"
          :key="key"
          class="va-slider__container__mark"
          :class="{ 'va-slider__container__mark--active': checkActivePin(pin) }"
          :style="getPinStyles(pin)"
        />
      </template>
      <template v-if="isRange">
        <div
          ref="process"
          class="va-slider__container__track va-slider__container__track--active"
          :style="processedStyles"
          @mousedown="moveStart($event, 0)"/>
        <div
          ref="dot0"
          class="va-slider__container__handler"
          :class="{'va-slider__container__handler--on-keyboard-focus': isKeyboardFocused === 1}"
          :style="dottedStyles[0]"
          @mousedown="(moveStart($event, 0), setMouseDown($event, 1))"
          @touchstart="moveStart($event, 0)"
          @focus="onFocus($event, 1)"
          @blur="isKeyboardFocused = false"
          :tabindex="!this.disabled && 0"
        >
          <div
            v-if="valueVisible"
            :style="labelStyle"
            class="va-slider__container__handler-value"
          >
            {{ val[0] }}
          </div>
        </div>
        <div
          ref="dot1"
          class="va-slider__container__handler"
          :class="{'va-slider__container__handler--on-keyboard-focus': isKeyboardFocused === 2}"
          :style="dottedStyles[1]"
          @mousedown="(moveStart($event, 1), setMouseDown($event, 2))"
          @touchstart="moveStart($event, 1)"
          @focus="onFocus($event, 2)"
          @blur="isKeyboardFocused = false"
          :tabindex="!this.disabled && 0"
        >
          <div
            v-if="valueVisible"
            :style="labelStyle"
            class="va-slider__container__handler-value">
            {{ val[1] }}
          </div>
        </div>
      </template>
      <template v-else>
        <div
          ref="process"
          class="va-slider__container__track va-slider__container__track--active"
          :style="processedStyles"
          @mousedown="moveStart($event, 0)"/>
        <div
          ref="dot"
          class="va-slider__container__handler"
          :class="{'va-slider__container__handler--on-keyboard-focus': isKeyboardFocused}"
          :style="dottedStyles"
          @mousedown="(moveStart(), setMouseDown())"
          @touchstart="moveStart()"
          @focus="onFocus"
          @blur="isKeyboardFocused = false"
          :tabindex="!this.disabled && 0"
        >
          <div
            v-if="valueVisible"
            :style="labelStyle"
            class="va-slider__container__handler-value"
          >
            {{ labelValue || val }}
          </div>
        </div>
      </template>
    </div>
    <span
      v-if="iconRight"
      class="va-slider__inverse-label">
      <va-icon :name="iconRight" :color="colorComputed" :size="16"/>
    </span>
    <span
      v-if="inverseLabel"
      :style="labelStyle"
      class="va-slider__label va-slider__inverse-label">
      {{ label }}
    </span>
    <div class="va-slider__input-wrapper" v-if="$slots.afterInput">
      <slot name="afterInput"/>
    </div>
  </div>
</template>

<script>
import { validateSlider } from './validateSlider'
import { getHoverColor } from '../../../services/color-functions'
import VaIcon from '../va-icon/VaIcon'
import { ColorThemeMixin } from '../../../services/ColorThemePlugin'
import { KeyboardOnlyFocusMixin } from '../va-checkbox/KeyboardOnlyFocusMixin'

export default {
  name: 'va-slider',
  components: {
    VaIcon,
  },
  mixins: [
    ColorThemeMixin,
    KeyboardOnlyFocusMixin,
  ],
  props: {
    range: {
      type: Boolean,
    },
    value: {
      type: [Number, Array],
    },
    labelValue: {
      type: String,
    },
    valueVisible: {
      type: Boolean,
    },
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 100,
    },
    step: {
      type: Number,
      default: 1,
    },
    label: {
      type: String,
    },
    inverseLabel: {
      type: Boolean,
    },
    disabled: {
      type: [Boolean, Array],
    },
    pins: {
      type: Boolean,
    },
    icon: {
      type: String,
    },
    iconRight: {
      type: String,
    },
  },
  data () {
    return {
      flag: false,
      size: 0,
      currentValue: this.value,
      currentSlider: 0,
      isComponentExists: false,
    }
  },
  computed: {
    sliderClass () {
      return {
        'va-slider--disabled': this.disabled,
      }
    },
    labelStyle () {
      return {
        color: this.colorComputed,
      }
    },
    trackStyles () {
      return {
        backgroundColor: getHoverColor(this.colorComputed),
      }
    },
    processedStyles () {
      const validatedValue = this.limitValue(this.value)

      if (this.range) {
        const val0 = ((validatedValue[0] - this.min) / (this.max - this.min)) * 100
        const val1 = ((validatedValue[1] - this.min) / (this.max - this.min)) * 100

        return {
          left: `${val0}%`,
          width: `${val1 - val0}%`,
          backgroundColor: this.colorComputed,
        }
      } else {
        const val = ((validatedValue - this.min) / (this.max - this.min)) * 100

        return {
          width: `${val}%`,
          backgroundColor: this.colorComputed,
        }
      }
    },
    dottedStyles () {
      const validatedValue = this.limitValue(this.value)

      if (this.range) {
        const val0 = ((validatedValue[0] - this.min) / (this.max - this.min)) * 100
        const val1 = ((validatedValue[1] - this.min) / (this.max - this.min)) * 100

        return [
          {
            left: `calc(${val0}% - 8px)`,
            backgroundColor: '#ffffff',
            borderColor: this.colorComputed,
          },
          {
            left: `calc(${val1}% - 8px)`,
            backgroundColor: '#ffffff',
            borderColor: this.colorComputed,
          },
        ]
      } else {
        const val = ((validatedValue - this.min) / (this.max - this.min)) * 100

        return {
          left: `calc(${val}% - 8px)`,
          backgroundColor: '#ffffff',
          borderColor: this.colorComputed,
        }
      }
    },
    val: {
      get () {
        return this.value
      },
      set (val) {
        if (!this.range) {
          val = this.limitValue(val)
        }
        this.$emit('input', val)
      },
    },
    total () {
      return (this.max - this.min) / this.step
    },
    gap () {
      return this.size / this.total
    },
    multiple () {
      let decimals = `${this.step}`.split('.')[1]
      return decimals ? Math.pow(10, decimals.length) : 1
    },
    interval () {
      return this.value[1] - this.value[0]
    },
    pinsCol () {
      return (this.max / this.step) - 1
    },
    position () {
      return this.isRange ? [(this.value[0] - this.min) / this.step * this.gap, (this.value[1] - this.min) / this.step * this.gap] : ((this.value - this.min) / this.step * this.gap)
    },
    limit () {
      return [0, this.size]
    },
    valueLimit () {
      return [this.min, this.max]
    },
    isRange () {
      return Array.isArray(this.value)
    },
  },
  watch: {
    val (val) {
      validateSlider(val, this.step, this.min, this.max)
    },
    max (val) {
      if (val < this.min) {
        validateSlider(this.value, this.step, val, this.max)
      }
    },
    min (val) {
      if (val > this.max) {
        validateSlider(this.value, this.step, this.min, val)
      }
    },
  },
  methods: {
    bindEvents () {
      document.addEventListener('mousemove', this.moving)
      document.addEventListener('touchmove', this.moving)
      document.addEventListener('mouseup', this.moveEnd)
      document.addEventListener('mouseleave', this.moveEnd)
      document.addEventListener('touchcancel', this.moveEnd)
      document.addEventListener('touchend', this.moveEnd)
      document.addEventListener('keydown', this.moveWithKeys)
    },
    unbindEvents () {
      document.removeEventListener('mousemove', this.moving)
      document.removeEventListener('touchmove', this.moving)
      document.removeEventListener('mouseup', this.moveEnd)
      document.removeEventListener('mouseleave', this.moveEnd)
      document.removeEventListener('touchcancel', this.moveEnd)
      document.removeEventListener('touchend', this.moveEnd)
      document.removeEventListener('keydown', this.moveWithKeys)
    },
    setMouseDown (e, index) {
      this.hasMouseDown = index || true
    },
    moveStart (e, index) {
      if (this.isRange) {
        this.currentSlider = index
      }

      this.flag = true
      this.$emit('drag-start', this)
    },
    moving (e) {
      if (!this.disabled) {
        if (!this.flag) {
          return false
        }

        if (e.type === 'touchmove') {
          this.setValueOnPos(this.getPos(e.touches[0]))
        } else {
          e.preventDefault()
          this.setValueOnPos(this.getPos(e))
        }
      }
    },
    moveEnd (e) {
      if (!this.disabled) {
        if (this.flag) {
          this.$emit('drag-end', this)
        } else {
          return false
        }
        this.flag = false
        this.hasMouseDown = false
      }
    },
    moveWithKeys (event) {
      // don't do anything if a dot isn't focused or if the slider's disabled
      if (![this.$refs.dot0, this.$refs.dot1, this.$refs.dot].includes(document.activeElement)) return
      if (this.disabled) return

      /*
        where: where to move
          0 - to left
          1 - to right

        which: which dot to move (only makes sence when isRange is true)
          0 - left dot
          1 - right dot
       */
      const moveDot = (isRange, where, which) => {
        if (isRange) {
          if (!this.pins) return this.val.splice(which, 1, this.val[which] + (where ? this.step : -this.step))

          // how many value units one pin occupies
          let onePinInterval = (this.max - this.min) / (this.pinsCol + 1)
          // how many full pins are to the left of the dot now
          let fullPinsNow = this.val[which] / onePinInterval | 0
          // the value of the nearest pin
          let nearestPinVal = fullPinsNow * onePinInterval

          if (this.val[which] !== nearestPinVal) { // if the dot's not pinned already
            nearestPinVal += where ? onePinInterval : 0 // take one more pin if moving right
            this.val.splice(which, 1, nearestPinVal)
          } else {
            this.val.splice(which, 1, this.val[which] + (where ? this.step : -this.step))
          }
        } else {
          if (!this.pins) {
            this.val += where ? this.step : -this.step
            return
          }

          // how many value units one pin occupies
          let onePinInterval = (this.max - this.min) / (this.pinsCol + 1)
          // how many full pins are to the left of the dot now
          let fullPinsNow = this.val / onePinInterval | 0
          // the value of the nearest pin
          let nearestPinVal = fullPinsNow * onePinInterval

          if (this.val !== nearestPinVal) { // if the dot's not pinned already
            nearestPinVal += where ? onePinInterval : 0 // take one more pin if moving right
            this.val = nearestPinVal
          } else {
            this.val += where ? this.step : -this.step
          }
        }
      }

      if (this.range) {
        if (this.$refs.dot0 === document.activeElement) { // left dot
          if (
            event.keyCode === 37 && // left arrow pressed
            !((this.val[0] - this.step) < this.min) // and won't become less than `min`
          ) moveDot(true, 0, 0)

          if (
            event.keyCode === 39 && // right arrow pressed
            !((this.val[0] + this.step) > this.val[1]) // and won't become more than the second dot is
          ) moveDot(true, 1, 0)
        } else if (this.$refs.dot1 === document.activeElement) { // right dot
          if (
            event.keyCode === 37 && // left arrow pressed
            !((this.val[1] - this.step) < this.val[0]) // and won't become less then the first dot is
          ) moveDot(true, 0, 1)

          if (
            event.keyCode === 39 && // right arrow pressed
            !((this.val[1] + this.step) > this.max) // and won't become more than `max`
          ) moveDot(true, 1, 1)
        }
      } else {
        if (event.keyCode === 37) moveDot(false, 0)
        if (event.keyCode === 39) moveDot(false, 1)
      }
    },
    wrapClick (e) {
      if (!this.disabled && !this.flag) {
        let pos = this.getPos(e)
        if (this.isRange) {
          this.currentSlider = pos > ((this.position[1] - this.position[0]) / 2 + this.position[0]) ? 1 : 0
        }
        this.setValueOnPos(pos)
        if (this.pins) {
          if (this.isRange) {
            if (this.currentValue[0] % this.step !== 0) {
              this.currentValue[0] = this.normalizeValue(this.currentValue[0])
              this.val = [this.currentValue[0], this.val[1]]
            }
            if (this.currentValue[1] % this.step !== 0) {
              this.currentValue[1] = this.normalizeValue(this.currentValue[1])
              this.val = [this.val[0], this.currentValue[1]]
            }
          } else {
            this.currentValue = this.normalizeValue(this.currentValue)
            this.val = this.currentValue
          }
        }
      }
    },
    checkActivePin (pin) {
      if (this.isRange) {
        return pin * this.step > this.val[0] && pin * this.step < this.val[1]
      } else {
        return pin * this.step < this.val
      }
    },
    getPinStyles (pin) {
      return {
        backgroundColor: this.checkActivePin(pin) ? this.colorComputed : getHoverColor(this.colorComputed),
        left: `${pin * this.step}%`,
      }
    },
    getPos (e) {
      this.getStaticData()
      return e.clientX - this.offset
    },
    getStaticData () {
      if (this.$refs.sliderContainer) {
        this.size = this.$refs.sliderContainer.offsetWidth
        this.offset = this.$refs.sliderContainer.getBoundingClientRect().left
      }
    },
    getValueByIndex (index) {
      return ((this.step * this.multiple) * index + (this.min * this.multiple)) / this.multiple
    },
    setCurrentValue (val) {
      let slider = this.currentSlider
      if (this.isRange) {
        if (this.isDiff(this.currentValue[slider], val)) {
          this.currentValue.splice(slider, 1, val)
          if (slider === 0) {
            this.val = [this.currentValue.splice(slider, 1, val)[0], this.value[1]]
            this.currentValue = [this.currentValue.splice(slider, 1, val)[0], this.value[1]]
          } else {
            this.val = [this.value[0], this.currentValue.splice(slider, 1, val)[0]]
            this.currentValue = [this.value[0], this.currentValue.splice(slider, 1, val)[0]]
          }
        }
      } else {
        if (val < this.min || val > this.max) {
          return false
        }
        if (this.isDiff(this.currentValue, val)) {
          this.currentValue = val
          this.val = val
        }
      }
    },
    setValueOnPos (pixelPosition, isDrag) {
      const range = this.limit
      const valueRange = this.valueLimit

      this.setTransform()

      if (pixelPosition >= range[0] && pixelPosition <= range[1]) {
        if (this.currentSlider) {
          if (pixelPosition <= this.position[0]) {
            this.currentSlider = 0
          }
          let v = this.getValueByIndex(Math.round(pixelPosition / this.gap))
          this.setCurrentValue(v, isDrag)
        } else {
          if (pixelPosition >= this.position[1]) {
            this.currentSlider = 1
          }
          let v = this.getValueByIndex(Math.round(pixelPosition / this.gap))
          this.setCurrentValue(v, isDrag)
        }
      } else if (pixelPosition < range[0]) {
        this.setCurrentValue(valueRange[0])
      } else {
        this.setCurrentValue(valueRange[1])
      }
    },
    setTransform () {
      if (this.isRange) {
        const slider = this.currentSlider
        const difference = 100 / (this.max - this.min)
        const val0 = (this.value[0] - this.min) * difference
        const val1 = (this.value[1] - this.min) * difference
        const processSize = `${val1 - val0}%`
        const processPosition = `${val0}%`

        this.$refs.process.style.width = processSize
        this.$refs.process.style['left'] = processPosition
        if (slider === 0) {
          this.$refs.dot0.style['left'] = `calc('${processPosition} - 8px)`
        } else {
          this.$refs.dot1.style['left'] = `calc('${processPosition} - 8px)`
        }
      } else {
        const val = ((this.value - this.min) / (this.max - this.min)) * 100

        this.$refs.process.style.width = `${val}%`
        this.$refs.dot.style['left'] = `calc('${val} - 8px)`
      }
    },
    normalizeValue (value) {
      const currentRest = value % this.step
      if ((currentRest / this.step) >= 0.5) {
        value = value + (this.step - currentRest)
      } else {
        value = value - currentRest
      }
      return value
    },
    limitValue (val) {
      const inRange = (v) => {
        if (v < this.min) {
          return this.min
        } else if (v > this.max) {
          return this.max
        }
        return v
      }

      if (this.isRange) {
        if (val[0] >= val[1] && this.currentSlider === 0) {
          return [val[1], val[1]]
        }
        if (val[0] >= val[1] && this.currentSlider === 1) {
          return [val[0], val[0]]
        }
        return val.map((v) => inRange(v))
      } else {
        return inRange(val)
      }
    },
    isDiff (a, b) {
      if (Object.prototype.toString.call(a) !== Object.prototype.toString.call(b)) {
        return true
      } else if (Array.isArray(a) && a.length === b.length) {
        return a.some((v, i) => v !== b[i])
      }
      return a !== b
    },
  },
  mounted () {
    this.$nextTick(() => {
      if (validateSlider(this.value, this.step, this.min, this.max)) {
        this.getStaticData()
        this.bindEvents()
      }
    })
  },
  beforeDestroy () {
    this.unbindEvents()
  },
}
</script>

<style lang='scss'>
@import "../../vuestic-sass/resources/resources";

.va-slider {
  display: flex;
  align-items: center;

  &--disabled {
    @include va-disabled;

    .va-slider__container__handler {

      &:hover {
        cursor: default;
      }
    }
  }

  &__input-wrapper {
    flex-basis: 8.33333%;
    flex-grow: 0;
    max-width: 8.33333%;
    margin-right: 1rem;
    min-width: 2.5rem;

    position: relative;
    display: flex;

    &:last-of-type {
      margin-left: 1rem;
    }
  }

  &__label {
    margin-right: 1rem;
    user-select: none;
    font-size: .625rem;
    letter-spacing: 0.6px;
    line-height: 1.2;
    font-weight: bold;
    text-transform: uppercase;
  }

  &__inverse-label {
    user-select: none;
    font-size: .625rem;
    letter-spacing: 0.6px;
    line-height: 1.2;
    font-weight: bold;
    text-transform: uppercase;
    margin-left: 1rem;
  }

  &__container {
    position: relative;
    width: 100%;
    height: 1.5rem;
    display: flex;
    align-items: center;

    &__track, &__track--active {
      position: absolute;
      height: 0.5rem;
      border-radius: 0.25rem;
    }

    &__track {
      width: 100%;
    }

    &__mark {
      position: absolute;
      width: 0.125rem;
      height: 0.75rem;
    }

    &__handler {
      position: absolute;
      width: 1.25rem;
      height: 1.25rem;
      background: $white;
      border: 0.375rem solid;
      border-radius: 50%;
      outline: none !important;

      &:hover {
        cursor: pointer;
      }

      &--on-keyboard-focus {
        @at-root .va-slider__container__handler#{&}:before {
          content: '';
          transform: translate(-0.625rem, -0.625rem);
          background-color: black !important;
          display: block;
          width: 1.75rem;
          height: 1.75rem;
          position: absolute;
          border-radius: 50%;
          opacity: 0.1;
          pointer-events: none;
        }
      }

      &-value {
        position: absolute;
        top: -8px;
        left: 50%;
        transform: translate(-50%, -100%);
        user-select: none;
        font-size: .625rem;
        letter-spacing: 0.6px;
        line-height: 1.2;
        font-weight: bold;
        text-transform: uppercase;
      }
    }
  }
}
</style>
