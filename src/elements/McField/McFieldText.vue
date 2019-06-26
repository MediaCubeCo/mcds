<template>
  <div class="mc-field-text" :class="classes">
    <label :for="name" class="mc-field-text__header">
      <slot name="header">
        <McTitle :ellipsis="false" v-if="title" :level="4">{{ title }}</McTitle>
      </slot>
    </label>
    <div class="mc-field-text__main">
      <div class="mc-field-text__prepend">
        <slot name="prepend" />
      </div>
      <div class="mc-field-text__input-wrapper">
        <flat-pickr
          v-if="isDate"
          class="mc-field-text__input"
          :style="inputStyles"
          :placeholder="placeholder"
          :value="value"
          @input="value => handleInput(value)"
          :name="name"
          ref="input"
          :disabled="disabled"
          :id="name"
        ></flat-pickr>
        <textarea
          v-else-if="isTextarea"
          class="mc-field-text__input"
          :style="inputStyles"
          :placeholder="placeholder"
          :value="value"
          @input="$event => handleInput($event.target.value)"
          :disabled="disabled"
          :name="name"
          :id="name"
        ></textarea>
        <input
          v-else
          class="mc-field-text__input"
          :style="inputStyles"
          :disabled="disabled"
          :type="type"
          :placeholder="placeholder"
          :value="value"
          @input="$event => handleInput($event.target.value)"
          ref="input"
          :name="name"
          :id="name"
        />
      </div>
      <div class="mc-field-text__append">
        <slot name="append" />
      </div>
    </div>
    <div class="mc-field-text__footer">
      <McTitle tag-name="div" :ellipsis="false" color="danger" size="s" v-if="errorText">
        {{ errorText }}
      </McTitle>
      <br v-if="errorText" />
      <slot name="footer">
        <McTitle tag-name="div" :ellipsis="false" size="s" v-if="helpText">{{ helpText }}</McTitle>
      </slot>
    </div>
  </div>
</template>

<script>
import tokens from "../../assets/tokens/tokens"
import flatPickr from "vue-flatpickr-component"

import McTitle from "../McTitle"
import McSvgIcon from "../McSvgIcon"
import McButton from "../McButton"
import McTooltip from "../McTooltip"
export default {
  name: "McFieldText",
  components: { McTooltip, McButton, McSvgIcon, McTitle, flatPickr },
  status: "ready",
  release: "1.0.0",
  props: {
    /**
     *  Тип:
     * `text, password, email и т.д.`
     */
    type: {
      type: String,
      default: "text",
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
  },

  data() {
    return {
      prependWidth: 0,
      appendWidth: 0,
    }
  },

  mounted() {
    this.calculatePadding()
  },

  computed: {
    classes() {
      return {
        "mc-field-text--error": this.errorText,
        "mc-field-text--textarea": this.isTextarea,
        "mc-field-text--date": this.isDate,
        "mc-field-text--disabled": this.disabled,
      }
    },

    isTextarea() {
      return this.type === "textarea"
    },

    isDate() {
      return this.type === "date"
    },

    errorText() {
      if (this.errors == null || this.errors.length === 0) return null
      return this.errors.join(", ")
    },

    inputStyles() {
      return {
        paddingLeft: this.prependWidth && `${this.prependWidth + parseInt(tokens.space_m)}px`,
        paddingRight: this.appendWidth && `${this.appendWidth + parseInt(tokens.space_m)}px`,
      }
    },
  },

  methods: {
    handleInput(value) {
      this.$emit("input", value)
    },

    calculatePadding() {
      this.prependWidth = this.calculateSlotPadding("prepend")
      this.appendWidth = this.calculateSlotPadding("append")
    },

    calculateSlotPadding(name) {
      return (
        this.$slots[name] &&
        this.$slots[name].reduce((acc, cur) => {
          const $el = cur.elm ? cur.elm : cur
          return acc + $el.getBoundingClientRect().width
        }, 0)
      )
    },
  },
}
</script>

<style lang="scss">
.mc-field-text {
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

  &__main {
    position: relative;
  }

  &__prepend,
  &__append {
    @include reset-text-indents();
    position: absolute;
    top: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    padding-left: $space-xs;
    padding-right: $space-xs;
    min-width: $tappable-element-m;
    height: 100%;
    margin-left: -$space-xxxs;
    margin-right: -$space-xxxs;
    padding-top: $space-xxs + 2;
    padding-bottom: $space-xxs + 2;

    &:empty {
      display: none;
    }

    > * {
      margin-left: $space-xxxs;
      margin-right: $space-xxxs;
    }
  }

  &__prepend {
    left: 0;
  }

  &__append {
    right: 0;
  }

  &__input-wrapper {
  }

  &__input {
    font-family: $font-heading;
    display: inline-block;
    vertical-align: middle;
    width: 100%;
    min-height: $tappable-element-m;
    margin: 0;
    border: 1px solid $color-gray-lighter;
    border-radius: $radius-m;
    padding: ($space-s / 2) + 1 $space-s;
    line-height: $line-height-s;
    font-size: $size-m;
    background-color: $color-white;
    -moz-appearance: textfield;
    appearance: textfield;
    transition: background-color $transition-time-fast, border-color $transition-time-fast;
    color: $color-text;

    &:focus {
      outline: 0;
      border-color: $color-primary;
    }

    &::-webkit-search-cancel-button,
    &::-webkit-search-decoration,
    &::-webkit-inner-spin-button,
    &::-webkit-outer-spin-button {
      -webkit-appearance: none;
    }

    @include input-placeholder() {
      color: $color-gray-dark;
    }
  }

  textarea#{$block-name} {
    &__input {
      $textarea-height: 94px;

      height: auto;
      min-height: calc(#{$textarea-height} + 2px);
      resize: vertical;
    }
  }

  &__footer {
    margin-top: $space-xxxs;
    &:empty {
      display: none;
    }
  }

  &--error {
    #{$block-name} {
      &__input {
        border-color: $color-danger;
      }
    }
  }

  &--textarea {
    #{$block-name} {
      &__append,
      &__prepend {
        align-items: flex-start;
      }
    }
  }

  &--disabled {
    #{$block-name} {
      &__input {
        cursor: not-allowed;
        background-color: $color-gray-lightest;
        border-color: $color-gray-lightest;
      }
    }
  }
}
</style>

<docs>
    ```jsx
    let text = null
    <div style="max-width: 700px">
        <McFieldText
                placeholder="Введите сообщение"
                name="login"
                help-text="Используйте электронный адрес, указанный при регистрации аккаунта MediaCube."
        >
            <McTitle :ellipsis="false" :level="4" slot="header">
                <McTooltip placement="right" slot="icon-append" size="s" content="Используйте электронный адрес, указанный при регистрации аккаунта MediaCube.">
                    <McSvgIcon name="help"/>
                </McTooltip>
                Электронная почта
            </McTitle>
            <McSvgIcon slot="prepend" name="face"/>
            <McSvgIcon slot="prepend" name="access_time"/>
            <McSvgIcon slot="append" name="attach_money"/>

            <McTooltip slot="append" size="s" content="Вывести все средства">
                <McButton uppercase variation="gray-darkest-invert" size="s">
                    Все
                </McButton>
            </McTooltip>
            <McButton slot="append" uppercase variation="gray-darkest-invert" size="s">
                Фонды
            </McButton>
        </McFieldText>

        <br>

        <McFieldText
                placeholder="Disabled"
                disabled
                name="pass"
                title="Заголовок"
                help-text="Используйте электронный адрес, указанный при регистрации аккаунта MediaCube."
        />

        <br>

        <McFieldText
                :errors="['Имя пользователя и пароль не совпадают', 'Поле обязательно для заполнения.']"
                name="message"
                type="textarea"
                placeholder="Введите сообщение"
                title="Textarea"
        >
            <McButton
                    variation="primary-link"
                    slot="append"
                    size="s-compact"
            >
                <McSvgIcon slot="icon-append" name="send" />
            </McButton>
        </McFieldText>

        <br>

        <McFieldText title="Флэтпицкер" name="date" type="date" placeholder="Дата"></McFieldText>
    </div>
    ```
</docs>