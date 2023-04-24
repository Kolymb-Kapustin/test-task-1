<template>
<div class="filter">
  <div
    v-for="(select, index) in selects"
    :key="index"
    class="filter__select"
  >
    <VueMultiselect
      :model-value="selectsValues[index]"
      :options="select.options"
      :searchable="false"
      :preselectFirst="true"
      :placeholder="select.placeholder"
      label="label"
      @update:modelValue="selectsValues[index] = $event"
    />
  </div>

  <input
    class="filter__search-input input"
    v-model="searchString"
    type="text"
    placeholder="Search"
  >
</div>
</template>

<script>
import VueMultiselect from 'vue-multiselect'

export default {
  name: 'MultipleFilter',

  components: {
    VueMultiselect
  },

  props: {
    items: Array,
    selects: Array,
    search: Boolean
  },

  emits: ['update'],

  data() {
    return {
      selectsValues: [],
      searchString: ''
    }
  },

  computed: {
    filteredData() {
      let result = this.items;

      if (this.selectsValues.length) {
        this.selectsValues.forEach((selectedValue, index) => {
          if (selectedValue && this.selects[index].options[0] !== selectedValue) {
            if (selectedValue.value !== null) {
              result = result.filter(el => el[this.selects[index].filter] === selectedValue.value)
            }
          }
        })
      }

      if (this.searchString.length) {
        result = result.filter(el => el.title.includes(this.searchString))
      }

      return result;
    }
  },

  watch: {
    filteredData() {
      this.$emit('update', this.filteredData)
    }
  }
} 
</script>

<style lang="less">
.filter {
  display: flex;
  flex-wrap: wrap;
  padding: 0 16px;
  margin-top: 16px;

  &__select {
    min-width: 240px;
    margin: 8px;

    @media screen and (max-width: 640px) {
      width: 100%;
    }
  }

  &__search-input.input {
    margin: 8px;
    max-width: 640px;
  }
  
  .multiselect {
    &__content-wrapper {
      color: #fff;
      border-radius: 5px;
      background: #474747;
      border: 1px solid #519944;
    }

    &__option--selected {
      color: #fff;
      background: #545454;
    }

    &__tags {
      cursor: pointer;
      color: #fff;
      padding-top: 10px;
      border-radius: 5px;
      background: #474747;
      border: 1px solid #519944;
    }

    &__single {
      background: none;
    }
  }
}
</style>
