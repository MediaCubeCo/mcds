<template>
  <div class="mc-date-picker" :class="classes">
    <label :for="name" class="mc-date-picker__header">
      <!-- @slot Слот для заголовка над инпутом -->
      <slot name="title">
        <mc-title :ellipsis="false" v-if="title" :level="4">{{ title }}</mc-title>
      </slot>
    </label>
    <div class="mc-date-picker__inner">
      <div class="mc-date-picker__input-wrapper">
        <date-picker
          v-model="prettyValue"
          v-on="listeners"
          ref="input"
          class="mc-date-picker__date-picker"
          range-separator=" — "
          :input-props="inputProps"
          :placeholder="placeholder"
          :name="name"
          :type="type"
          :range="range"
          :id="name"
          :lang="lang"
          :format="format"
          :editable="editable"
          :clearable="clearable"
          :disabled="disabled"
          :disabled-date="disabledDate"
          :inline="inline"
          :append-to-body="appendToBody"
          :popup-class="popupClass"
          :popup-style="popupStyle"
          @input="value => handleEmitDate(value)"
        >
          <div v-if="$slots.header" slot="header">
            <!-- @slot Слот для вставки в хедер попапа календаря -->
            <slot name="header" />
          </div>
          <div v-if="$slots.sidebar" slot="sidebar">
            <!-- @slot Слот для вставки в сайдбар попапа календаря  -->
            <slot name="sidebar" />
          </div>
          <div v-if="$slots.input" slot="input">
            <!-- @slot Слот для вставки в инпут попапа календаря -->
            <slot name="input" />
          </div>
          <div v-if="$slots.footer" slot="footer">
            <!-- @slot Слот для вставки в футер попапа календаря -->
            <slot name="footer" />
          </div>

          <!-- @slot Слот для вставки пользовательской иконки календаря -->
          <template slot="icon-calendar">
            <mc-svg-icon class="mc-date-picker__icon" name="date_range" fill="#4285F4" />
          </template>
        </date-picker>
      </div>
    </div>
    <div class="mc-date-picker__footer" v-if="errorText || helpText || $slots.bottom">
      <mc-title tag-name="div" :ellipsis="false" color="red" size="s" v-if="errorText">
        {{ errorText }}
      </mc-title>
      <br v-if="errorText" />
      <!-- @slot Слот для доп. текста под инпутом -->
      <slot name="bottom">
        <mc-title tag-name="div" :ellipsis="false" size="s" v-if="helpText">
          {{ helpText }}
        </mc-title>
      </slot>
    </div>
  </div>
</template>

<script>
import _omit from "lodash/omit"

import DatePicker from "vue2-datepicker"
import "vue2-datepicker/index.css"
import "vue2-datepicker/locale/ru"
import "vue2-datepicker/locale/en"
import "vue2-datepicker/locale/es"

import McTitle from "./McTitle"
import McSvgIcon from "./McSvgIcon"
import McButton from "./McButton"
import McTooltip from "./McTooltip"

export default {
  name: "McDatePicker",
  status: "ready",
  release: "1.0.0",
  components: {
    McTooltip,
    McButton,
    McSvgIcon,
    McTitle,
    DatePicker,
  },
  props: {
    /**
     * Рендер компонента в тег
     * */
    appendToBody: {
      type: Boolean,
      default: true,
    },
    /**
     *  Тип дейтпикера:	date|datetime|year|month|time|week
     *
     */
    type: {
      type: String,
      default: "date",
    },

    /**
     *  Заголовок поля:
     *
     */
    title: {
      type: String,
      default: null,
    },

    /**
     *  Вспомогательный текст под инпутом:
     *
     */
    helpText: {
      type: String,
      default: null,
    },

    /**
     *  Отключенное состояние
     *
     */
    disabled: {
      type: Boolean,
      default: false,
    },
    /**
     * Запретить для выбора диапазон дат
     *
     * */
    disabledDate: {
      type: Function,
      default: () => {},
    },

    /**
     *  Возможность очистить инпут
     *
     */
    clearable: {
      type: Boolean,
      default: false,
    },

    /**
     *  редактируемый инпут
     *
     */
    editable: {
      type: Boolean,
      default: false,
    },

    /**
     *  Значение
     *
     */
    value: {
      default: null,
    },

    /**
     *  Ошибки
     *
     */
    errors: {
      type: Array,
      default: null,
    },

    /**
     *  Placeholder
     *
     */
    placeholder: {
      type: String,
      default: null,
    },

    /**
     *  Name
     *
     */
    name: {
      type: String,
      required: true,
    },

    /**
     *  Формат даты (как в moment.js)
     *
     */
    lang: {
      type: String,
      default: "en",
    },

    /**
     *  Формат даты (как в moment.js)
     *
     */
    format: {
      type: String,
      default: "DD.MM.YYYY",
    },

    /**
     *  Формат отдаваемой даты
     *
     */
    toFormat: {
      type: String,
      default: "YYYY-MM-DD",
    },

    /**
     *  Дейтпикер типа range
     *
     */
    range: {
      type: Boolean,
      default: false,
    },

    /**
     *  Дейтпикер без инпута
     *
     */
    inline: {
      type: Boolean,
      default: false,
    },

    /**
     * Стили для календаря датапикера
     * */
    popupStyle: {
      type: Object,
    },

    /**
     * Кастомный datepicker
     */
    custom: {
      type: Boolean,
      default: false,
    },
  },

  data() {
    return {
      // prettyValue: this.value ? new Date(this.value) : new Date(),
      prettyValue: null,
    }
  },
  watch: {
    value() {
      if (!this.range) {
        this.prettyValue = new Date(this.value)
      }
    },
  },
  mounted() {
    if (this.range) {
      const arrValues = this.value
        ? this.value.map(item => new Date(item))
        : [new Date(), new Date()]
      this.prettyValue = arrValues
    } else {
      this.prettyValue = this.value ? new Date(this.value) : new Date()
    }
  },
  computed: {
    classes() {
      return {
        "mc-date-picker--error": this.errorText,
        "mc-date-picker--disabled": this.disabled,
        "mc-date-picker--custom": this.custom,
      }
    },
    popupClass() {
      return {
        "datepicker-popup": true,
        "datepicker-popup--en": this.lang === "en",
        "datepicker-popup--es": this.lang === "es",
        "datepicker-popup--ru": this.lang === "ru",
      }
    },

    inputProps() {
      return {
        placeholder: this.placeholder,
        disabled: this.disabled,
      }
    },

    errorText() {
      if (this.errors == null || this.errors.length === 0) return null
      return this.errors.join(", ")
    },

    listeners() {
      return _omit(this.$listeners, "input")
    },
  },

  methods: {
    handleInput(value) {
      /**
       * Событие инпута
       * @property {string}
       */
      this.$emit("input", value)
    },

    handleEmitDate(value) {
      let newValue = value
      if (!value || !value.length) {
        newValue = this.$moment(value).format(this.toFormat)
      }
      this.handleInput(newValue)
    },
  },
}
</script>

<style lang="scss">
.mc-date-picker {
  $block-name: &;

  display: block;

  &__header {
    @include reset-text-indents();
    display: block;
    margin-bottom: $space-xs;

    &:empty {
      display: none;
    }
  }

  &__inner {
    display: flex;
    align-items: center;
  }

  &__input-wrapper {
    width: 100%;
  }

  &__footer {
    margin-top: $space-xxxs;

    &:empty {
      display: none;
    }
  }

  &__icon {
    font-size: inherit;
  }
  .mx-input-wrapper {
    width: 100%;
  }
  .mx-input {
    font-size: $size-m;
    height: $tappable-element-m;
    padding: 6px 10px 6px 36px;
    cursor: pointer;
    &:hover {
      border-color: #ccc;
    }
    &:focus {
      border-color: #409aff;
    }
  }
  .mx-datepicker {
    width: 100%;
    svg {
      @include size($tappable-element-xs);
    }
    &-inline {
      width: auto;
    }
  }
  .mx-icon-calendar {
    right: unset;
    left: $space-xs;
  }

  &--error {
    .mx-input {
      border-color: $color-red !important;
    }
  }
  &--disabled {
    .mx-input {
      color: $color-gray-dark;
      cursor: not-allowed;
      background-color: $color-hover-gray;
      border-color: $color-hover-gray;
      box-shadow: none;
      &:hover {
        border-color: $color-hover-gray;
      }
    }
  }

  &.mc-date-picker--custom {
    .mx-calendar {
      &-header {
        margin-bottom: 8px;
      }
    }
    .mx-icon {
      &-double-right {
        &:before {
          display: none;
        }
      }
      &-double-left {
        &:before {
          display: none;
        }
      }
    }
    .mx-table-month {
      td {
        height: auto;
        padding: 26px 19px;
      }
    }
    .mx-datepicker {
      &-range {
        .mx-calendar {
          border-left: 0;
        }
      }
    }
  }
}
.mx-calendar {
  width: 340px;
}
.mx-calendar-content {
  height: auto;
}
.mx-input-wrapper:hover .mx-icon-clear + .mx-icon-calendar {
  display: block;
}
.datepicker-popup {
  border-radius: $radius-l;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.175);
  z-index: 12222;
  .cell {
    color: $color-black;
    width: $tappable-element-m;
    height: 33px;
    font-size: 16px;
    &:hover {
      background-color: $color-hover-gray;
      border-radius: $radius-m;
    }
  }
  .today,
  .active {
    border-radius: $radius-m;
  }
  .today,
  .in-range {
    background-color: $color-lightest-blue !important;
    color: $color-blue !important;
  }
  th {
    font-size: 16px;
    color: $color-black;
  }
  .mx-btn-text {
    font-size: $size-l;
    color: $color-black;
  }
  .mx-btn-current-year {
    color: $color-outline-gray;
  }
  .mx-datepicker-footer {
    border: none;
    text-align: left;
  }
  .mx-datepicker-header {
    border: none;
  }
  .mx-table {
    border-spacing: 8px;
  }
  .mx-calendar-header-label {
    color: $color-black;
    font-size: $size-l;
  }
  .mx-calendar-decade-separator:after {
    content: "-";
  }
  .mx-btn-icon-double-left {
    margin-left: 15px;
  }
  .mx-btn-icon-double-right {
    margin-right: 15px;
  }
  .mx-icon-left:before,
  .mx-icon-right:before,
  .mx-icon-double-left:before,
  .mx-icon-double-right:before,
  .mx-icon-double-left:after,
  .mx-icon-double-right:after {
    width: 12px;
    height: 12px;
    border-width: 3px 0 0 3px;
  }

  .not-current-month {
    user-select: none;
    pointer-events: none;
    color: $color-hover-gray !important;
  }

  &--es,
  &--ru {
    .mx-date-row .cell:nth-child(6),
    .mx-date-row .cell:nth-child(7) {
      color: $color-outline-gray;
    }
  }

  &--en {
    .mx-date-row .cell:nth-child(1),
    .mx-date-row .cell:nth-child(7) {
      color: $color-outline-gray;
    }
  }
}
</style>

<docs>
    ```jsx
    let text = null
    let value = '2019-06-20T00:00:00+00:00'
    let test = [`2020, 1, 8`,`2020, 10, 8`]
    <div style="max-width: 700px">

        <mc-date-picker :disabled-date="date => date < new Date()" title="datePickerInlineRu" lang="ru" name="date" type="date" placeholder="Дата1" inline />

        <br>

        <mc-date-picker title="datePickerInlineYear" lang="ru" name="date" type="year" placeholder="Дата1" inline />

        <br>


        <br>

        <mc-date-picker title="Custom datepicker"
                        custom
                        range
                        lang="ru"
                        name="date"
                        type="month"
                        placeholder="Дата1"
                        inline
                        v-model="test"
        />

        <br>

        <mc-date-picker
          title="datePickerClearableAndEditable"
          name="date"
          type="date"
          clearable
          editable
          format="MM/DD/YYYY"
          placeholder="Дата2"
          help-text="Вспомогательный текст"
          v-model="value"
        >
            <template slot="header">
                <mc-button variation="blue-invert">Сегодня</mc-button>
            </template>
        </mc-date-picker>

        <br>

        <mc-date-picker
          title="datePickerRange"
          name="date" type="date"
          placeholder="Дата3"
          range
          :errors="['Необходимо выбрать период.']"
        >
            <div slot="footer">
                <mc-button variation="blue-invert" full-width>Сегодня</mc-button>
            </div>
        </mc-date-picker>

        <br>
    </div>
    ```
</docs>
