<template>
  <div class="mc-chat">
    <mc-title v-if="!!title" size="s">
      {{ title }}
    </mc-title>
    <div class="mc-chat__source" v-if="sources.length">
      <mc-chat-source :sources="sources" :value="source" @input="handleSourceInput" />
    </div>
    <div class="mc-chat__form" v-if="showInput">
      <mc-chat-form
        :value="value"
        :loading="loading"
        :errors="errors"
        :avatar="avatar"
        :placeholder="placeholder"
        @input="handleInput"
        @submit="handleSubmit"
      />
    </div>
    <div class="mc-chat__comments" v-if="comments.length">
      <div class="mc-chat__comment" v-for="comment in sortedComments" :key="comment.key">
        <mc-chat-comment :comment="comment" />
        <mc-separator v-if="separator" />
      </div>
    </div>
  </div>
</template>

<script>
import McChatForm from "./McChatForm"
import McChatComment from "./McChatComment"
import McChatSource from "./McChatSource"
import McTitle from "../../elements/McTitle"
import McSeparator from "../../elements/McSeparator"

import _sortBy from "lodash/sortBy"
import _reverse from "lodash/reverse"

export default {
  name: "McChat",
  components: { McTitle, McChatSource, McChatComment, McChatForm, McSeparator },
  status: "ready",
  release: "1.0.0",
  props: {
    value: {
      type: String,
    },
    comments: {
      type: Array,
      default() {
        return []
      },
    },
    separator: {
      type: Boolean,
      default: false,
    },
    avatar: {
      type: String,
      default: null,
    },
    loading: {
      type: Boolean,
      default: false,
    },
    errors: {
      type: Object,
      default() {
        return {}
      },
    },
    title: {
      type: String,
      default: null,
    },
    placeholder: {
      type: String,
      default: null,
    },
    showInput: {
      type: Boolean,
      default: true,
    },
    sources: {
      type: Array,
      default() {
        return []
      },
    },
    source: {
      default: null,
    },
  },

  computed: {
    sortedComments() {
      return _reverse(_sortBy(this.comments, this.handleSort))
    },
  },

  methods: {
    handleInput(value) {
      /**
       * Событие ввода
       * @property {string}
       */
      this.$emit("input", value)
    },
    handleSubmit() {
      /**
       * Событие по отправке
       */
      this.$emit("submit")
    },
    handleSourceInput(value) {
      /**
       * Событие по выбору источника
       * @property {Object}
       */
      this.$emit("sourceInput", value)
    },
    handleSort(comment) {
      return this.$moment ? this.$moment(comment.date) : Date.parse(comment.date)
    },
  },
}
</script>

<style lang="scss">
.mc-chat {
  $block-name: &;
  padding: $space-xs;
  border-top: 1px solid $color-outline-gray;

  &__source {
    margin-bottom: $space-xs;
  }
  &__comment {
    margin-top: $space-m;
  }
}
</style>

<docs>
    ```jsx
    let text = ''
    let sources = [
        { name: 'Источник 1', value: '1' },
        { name: 'Другой чат', value: '2' },
    ]
    let source = '1';
    let comments = [
    { content: 'Статус (recruiting): Присвоено', date: '2018-10-19 20:30',
    user: { name: 'Имя пользователя', avatar: 'https://lorempixel.com/640/480/?64646' },
    by_user: { name: 'Имя изменившего пользователя' }, },
    { content: 'Статус (recruiting): Отмена', date: '2014-12-19 10:22', user: { name: 'пользователя Имя' }, color: 'red' },
    ]
    let input = value => {
    text = value
    }
    let sourceInput = value => {
    source = value
    }
    let submit = () => {
    comments.push({ content: text, date: new Date().toString(), user: { name: 'Тест' }, color: 'blue',})
    text = ''
    }
    <div>
        <mc-chat
            style="width: 500px"
            :value="text"
            :comments="comments"
            :sources="sources"
            :source="source"
            @submit="submit"
            @input="input"
            @sourceInput="sourceInput"
        />
    </div>
    ```
</docs>
