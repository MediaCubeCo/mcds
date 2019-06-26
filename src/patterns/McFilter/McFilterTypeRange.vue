<template>
  <McCollapse no-border>
    <template>
      {{ filter.name }}
    </template>
    <template slot="body">
      <McFilterChip
        v-for="(values, name) in value"
        :key="name"
        :type="filter.type"
        :name="name"
        :value="values"
        :closable="true"
        @click="handleInput(name)"
        style="margin-left: 10px"
      />
      <div class="mc-filter-type-range">
        <McGridRow :gutter-x="10">
          <McGridCol :span="6">
            <McFieldText
              :value="value.more || ''"
              @input="value => handleInput('more', value)"
              :type="filter.type === 'date' ? 'date' : 'text'"
              placeholder="От"
              name="more"
            />
          </McGridCol>
          <McGridCol :span="6">
            <McFieldText
              :value="value.less || ''"
              @input="value => handleInput('less', value)"
              :type="filter.type === 'date' ? 'date' : 'text'"
              placeholder="До"
              name="less"
            />
          </McGridCol>
        </McGridRow>
      </div>
    </template>
  </McCollapse>
</template>

<script>
import McGridRow from "../McGrid/McGridRow"
import McGridCol from "../McGrid/McGridCol"
import McButton from "../../elements/McButton"
import McCollapse from "../../patterns/McCollapse"
import McFieldText from "../../elements/McField/McFieldText"
import McFilterChip from "./McFilterChip"

export default {
  name: "McFilterTypeRange",
  components: { McFilterChip, McFieldText, McButton, McGridCol, McGridRow, McCollapse },
  props: {
    value: {
      type: Object,
      required: true,
    },
    filter: {
      type: Object,
      required: true,
    },
  },
  methods: {
    handleInput(type, value) {
      const currentValue = { ...this.value }
      if (value) {
        currentValue[type] = this.filter.type === "date" ? value : +value
      } else {
        delete currentValue[type]
      }
      this.emitInput(currentValue)
    },
    emitInput(value) {
      this.$emit("input", value)
    },
  },
}
</script>