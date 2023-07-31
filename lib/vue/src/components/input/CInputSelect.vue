<template>
    <vue-select
      :value="value"
      v-bind="$attrs"
      :clearable="clearable"
      :options="options"
      :searchable="searchable"
      :calculate-position="calculateDropdownPosition"
      v-on="$listeners"
    >
      <template
        v-for="(_, name) in $scopedSlots"
        v-slot:[name]="data"
      >
        <slot
          :name="name"
          v-bind="data"
        />
      </template>
    </vue-select>
</template>

<script>
import { VueSelect } from 'vue-select'
import calculateDropdownPosition from '../../mixins/vue-select-position'
import 'vue-select/dist/vue-select.css';
export default {
  name: 'CInputSelect',

  components: {
    VueSelect,
  },

  mixins: [
    calculateDropdownPosition,
  ],

  props: {
    value: {
      type: String,
      default: '',
    },

    options: {
      type: Array,
      default: () => {
        return []
      },
    },

    clearable: {
      type: Boolean,
      default: true,
    },

    searchable: {
      type: Boolean,
      default: true,
    },
  },
}
</script>

<style lang="scss">

.v-select {
  min-width: auto;
  position: relative;
  -ms-flex: 1 1 auto;
  flex: 1 1 auto;
  margin-bottom: 0;
  font-size: .9rem;
  border-radius: 0.25rem;
  background-color: var(--white);

  .vs__selected-options {
    // do not allow growing
    width: 0;
  }

  .vs__selected {
    display: block;
    white-space: nowrap;
    text-overflow: ellipsis;
    max-width: 100%;
    overflow: hidden;
  }

  .vs__search {
    margin-top: 0.375rem;
  }

  &:not(.vs--open) .vs__selected + .vs__search {
    // force this to not use any space
    // we still need it to be rendered for the focus
    width: 0;
    padding: 0;
    margin: 0;
    border: none;
    height: 0;
  }

  .vs__dropdown-toggle {
    padding: 0.375rem;
    padding-top: 0;
    border-width: 2px;
    border-color: var(--light);

    .vs__selected {
      margin-top: 0.375rem;
    }

    .vs__actions {
      padding-top: 0.375rem;
    }
  }
}

</style>
