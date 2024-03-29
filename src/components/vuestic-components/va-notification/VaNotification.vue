<template>
  <transition v-if="value" name="fade">
    <div class="va-notification"
      :style="notificationStyle"
    >
      <div class="va-notification__content">
        <slot/>
      </div>

      <div
        v-if="closeable"
        class="va-notification__close-icon"
        @click="hideNotification()"
      >
        <va-icon
          :color="color"
          name="ion-md-close ion"
        />
      </div>
    </div>
  </transition>
</template>

<script>
import VaIcon from '../va-icon/VaIcon'
import {
  getHoverColor,
  getBoxShadowColor,
} from '../../../services/color-functions'

export default {
  name: 'va-notification',
  components: {
    VaIcon,
  },
  computed: {
    notificationStyle () {
      return {
        background: getHoverColor(this.$themes[this.color]),
        boxShadow: '0 0.125rem 0.125rem 0 ' + getBoxShadowColor(this.$themes[this.color]),
      }
    },
  },
  props: {
    color: {
      color: String,
      default: 'success',
    },
    value: {
      type: Boolean,
      default: true,
    },
    closeable: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    hideNotification () {
      this.$emit('input', false)
    },
  },
}
</script>

<style lang='scss'>
@import "../../vuestic-sass/resources/resources";

// Notifications
$va-notification-margin-y: 0.25rem;
$va-notification-padding-x: 0.5rem;
$va-notification-padding-y: 0.75rem;
$va-notification-border: 0;
$va-notification-border-radius: 0.5rem;
$va-notification-box-shadow: 0.125rem;

// Badge
$va-badge-margin-right: 0.5rem;
$va-badge-padding-x: 0.5rem;
$va-badge-padding-y: 0.125rem;
$va-badge-border-radius: 0.5rem;
$va-badge-font-size: 0.625rem;
$va-badge-letter-spacing: 0.0625rem;

// Close Icon
$va-close-icon-padding-x: 0.5rem;
$va-close-icon-padding-y: 0.0625rem;
$va-close-icon-font-size: 1.5rem;

.va-notification {
  padding: $va-notification-padding-y $va-notification-padding-x;
  margin: $va-notification-margin-y auto;
  display: flex;
  align-items: center;
  border: $va-notification-border solid transparent;
  border-radius: $va-notification-border-radius;

  &__content {
    display: flex;
    align-items: center;
    flex-grow: 1;
  }

  &__close-icon {
    padding: $va-close-icon-padding-y $va-close-icon-padding-x;
    font-size: $va-close-icon-font-size;
    cursor: pointer;
  }

  @include media-breakpoint-down(xs) {
    @at-root {
      .va-notification {
        &__content {
          flex-direction: column;
          align-items: flex-start;
        }

        &__close-icon {
          align-self: flex-start;
          display: flex;
          align-items: flex-start;
          padding: 0;
          padding-right: $va-close-icon-padding-x;
          margin: 0;
        }
      }
    }
  }
}
</style>
