<template>
  <vxe-table-column class="mc-virtual-table-col" v-bind="attrs" v-on="$listeners">
    <template v-slot="{ row, rowIndex }">
      <!-- @slot Слот для mc-virtual-table-col -->
      <slot :row="row">
        <mc-title :text-align="textAlign" :ellipsis="ellipsis" class="mc-virtual-table-col__title">
          {{ $attrs.type === "seq" ? rowIndex + 1 : row[defaultTitle] }}
        </mc-title>
      </slot>
      <div v-if="$scopedSlots.right" class="mc-virtual-table-col__right">
        <!-- @slot Слот справа в ячейке (абсолютно спозиционированный, с бэкграундом) -->
        <slot name="right" :row="row" />
      </div>
    </template>
    <template v-slot:header="{ column }">
      <!-- @slot Слот заголовка столбца -->
      <slot name="header" :column="column">
        <mc-title :text-align="textAlign" class="mc-virtual-table-col__title">
          <mc-svg-icon v-if="isSortable" slot="icon-prepend" :name="getSortIcon(column)" />
          {{ $attrs.type === "seq" ? "#" : column.title }}
          <div slot="icon-append">
            <!-- @slot Слот для вставки в начало перед заголовком столбца -->
            <slot name="header-append" />
          </div>
        </mc-title>
        <mc-svg-icon
          v-if="draggable"
          name="drag"
          fill="gray"
          class="mc-virtual-table-col__drag-icon"
        />
      </slot>
    </template>
    <template v-slot:footer="{ columnIndex, items }">
      <mc-title v-if="items[columnIndex]" :text-align="textAlign">
        {{ items[columnIndex] }}
      </mc-title>
      <template v-else-if="getVisibilityCommonInfo(columnIndex, items)">
        <mc-title v-if="!provideData.canShowLoader" class="mc-virtual-table-col__title">
          {{ provideData.placeholders.all_loaded }}
        </mc-title>
        <span
          v-if="provideData.canShowLoader"
          class="mc-virtual-table-col__loader"
          ref="loader"
        ></span>
      </template>
    </template>
  </vxe-table-column>
</template>

<script>
import McTitle from "../../elements/McTitle"
import McSvgIcon from "../../elements/McSvgIcon"

/**
 * Смотреть mc-virtual-table
 */
export default {
  name: "McVirtualTableCol",
  status: "ready",
  release: "1.1.0",
  components: {
    McTitle,
    McSvgIcon,
  },
  inject: ["provideData"],
  props: {
    hasBorder: {
      type: Boolean,
      default: false,
    },
    draggable: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    attrs() {
      const classes = {
        "class-name": this.handleClassName,
        "header-class-name": this.handleClassName,
        "footer-class-name": this.handleFooterClassName,
      }
      return {
        ...classes,
        ...this.$attrs,
      }
    },
    defaultTitle() {
      return this.$attrs.field
    },
    textAlign() {
      return this.$attrs.align || "left"
    },
    isSortable() {
      return this.$attrs.hasOwnProperty("sortable")
    },
    ellipsis() {
      if (!this.$attrs.hasOwnProperty("show-overflow")) return true
      return this.$attrs["show-overflow"] !== false
    },
  },
  methods: {
    getVisibilityCommonInfo(columnIndex, items) {
      const index = items.indexOf(null)
      return columnIndex === index
    },
    getSortIcon(column) {
      if (!this.provideData.nativeSort) {
        if (this.provideData.sortedBy && column.property === this.provideData.sortedBy) {
          return this.provideData.sortedDescending ? "arrow_downward" : "arrow_upward"
        }
        return "unfold_more"
      }
      if (!column.order) return "unfold_more"
      return column.order === "desc" ? "arrow_downward" : "arrow_upward"
    },
    handleClassName() {
      const classes = []
      if (this.hasBorder && !this.provideData.cardIsOpen) {
        classes.push("mc-virtual-table-col--border-right")
      }
      if (this.$attrs["show-overflow"] === false) {
        classes.push("mc-virtual-table-col--overflow-visible")
      }
      if (this.draggable && !this.provideData.cardIsOpen) {
        classes.push("mc-virtual-table-col--draggable")
      }
      return classes
    },
    handleFooterClassName() {
      if (this.hasBorder && !this.provideData.cardIsOpen) {
        return "mc-virtual-table-col--border-right"
      }
    },
  },
}
</script>

<style lang="scss">
.mc-virtual-table-col {
  &--border-right {
    border-right: 1px solid $color-secondary-gray;
  }
  &--border-top {
    border-top: 1px solid $color-secondary-gray;
  }
  &--border-bottom {
    border-bottom: 1px solid $color-secondary-gray;
  }
  &--overflow-visible {
    .vxe-cell {
      overflow: visible !important;
    }
  }
  &--draggable {
    cursor: pointer;
    .mc-virtual-table-col__drag-icon {
      @include position(absolute, 50% 5px null null);
      transform: translateY(-50%);
    }
    .vxe-cell--title {
      margin-right: $space-m;
    }
  }

  &__title {
    width: auto;
    max-width: 101%;
  }
  &__right {
    display: flex;
    align-items: center;
    flex-wrap: nowrap;
    @include position(absolute, 0 $space-xs 0 null);
    background-color: $color-white;

    &::before {
      @include pseudo();
      @include position(null, 0 100% 0 null);
      width: $tappable-element-xs;
      background: linear-gradient(90deg, hsla(0, 0%, 100%, 0) 0, $color-white);
    }
  }
}

.vxe-cell {
  &.c--tooltip,
  &.c--title {
    text-overflow: clip !important;
    white-space: normal !important;
  }
}
.vxe-body--row {
  transition: background-color 0s;
  &.row--hover {
    .mc-virtual-table-col__right {
      background-color: #f5f7fa;
      &::before {
        background: linear-gradient(90deg, hsla(0, 0%, 100%, 0) 0, #f5f7fa);
      }
    }
    &.row--current {
      .mc-virtual-table-col__right {
        background-color: #d7effb;
        &::before {
          background: linear-gradient(90deg, hsla(0, 0%, 100%, 0) 0, #d7effb);
        }
      }
    }
  }
  &.row--current {
    .mc-virtual-table-col__right {
      background-color: #e6f7ff;
      &::before {
        background: linear-gradient(90deg, hsla(0, 0%, 100%, 0) 0, #e6f7ff);
      }
    }
  }
}
</style>
