<template>
  <div class="mc-grid-col" :class="classes" :style="styles">
    <slot></slot>
  </div>
</template>

<script>
import _upperFirst from "lodash/upperFirst"
const values = ["span", "order", "offset", "push", "pull"]
const sizes = ["s", "m", "l", "xl", "xxl"]
const props = {}

values.forEach(value => {
  sizes.forEach(size => {
    props[`${value}-${size}`] = { type: Number }
  })
})

export default {
  name: "McGridCol",
  status: "ready",
  release: "0.0.1",

  props: {
    span: { type: Number },
    order: { type: Number },
    offset: { type: Number },
    push: { type: Number },
    pull: { type: Number },
    ...props,
  },

  data() {
    return {
      gutter: 0,
    }
  },

  computed: {
    classes() {
      const list = {}
      values.forEach(value => {
        list[`mc-grid-col--${value}-${this[value]}`] = this[value]
        sizes.forEach(size => {
          const sizeValue = this[`${value}${_upperFirst(size)}`]
          list[`mc-grid-col--${value}-${size}-${sizeValue}`] = sizeValue
        })
      })
      return list
    },

    styles() {
      let colStyle =
        this.gutter !== 0
          ? {
              "padding-left": `${this.gutter / 2}px`,
              "padding-right": `${this.gutter / 2}px`,
            }
          : {}
      return this.order ? { ...colStyle, order: this.order } : colStyle
    },
  },

  mounted() {
    this.$parent.updateGutter()
  },
}
</script>

<style lang="scss">
.mc-grid-col {
  $block-name: &;

  $cols: 12;

  display: block;
  position: relative;
  flex: 1;

  @mixin generate-col($viewport: "") {
    @for $i from 1 through $cols {
      &--span-#{$viewport}-#{$i} {
        width: percentage($i / $cols);
      }

      &--push-#{$viewport}-#{$i} {
        left: percentage($i / $cols);
      }

      &--pull-#{$viewport}-#{$i} {
        right: percentage($i / $cols);
      }

      &--offset-#{$viewport}-#{$i} {
        margin-left: percentage($i / $cols);
      }

      &--order-#{$viewport}-#{$i} {
        order: #{$i};
      }
    }
  }

  @include generate-col("");

  @media #{$media-query-s} {
    @include generate-col("-s");
  }

  @media #{$media-query-m} {
    @include generate-col("-m");
  }

  @media #{$media-query-l} {
    @include generate-col("-l");
  }

  @media #{$media-query-xl} {
    @include generate-col("-xl");
  }

  @media #{$media-query-xxl} {
    @include generate-col("-xxl");
  }
}
</style>