<template>
  <div
    v-click-outside.capture="handleClose"
    @click="handleOpen"
    :class="classes">
    <div :class="valueClasses">
      <span v-if="!selected"><slot></slot></span>
      {{ selected }}

      <o-icon
        @on-click.stop="handleClear"
        :class="clearClasses"
        v-if="canClear"
        type="clear">
      </o-icon>

      <o-icon type="down" :class="arrowsClasses" :size="12"></o-icon>

    </div>
    <transition :name="dropName">
      <div :class="listClasses" v-show="visible">
        <div
          :class="[$LEO.prefix + 'select-item', { active: item.value === value }]"
          @click="handleChange(item.value)"
          v-for="item in list">
          {{ item.label }}
        </div>
        <div v-if="!list.length" :class="textClasses">{{ notFoundText }}</div>
      </div>
    </transition>
  </div>
</template>

<script>
  import { oneOf } from '../../utils/assist';
  import { directive as clickOutside } from 'v-click-outside-x';

  export default {
    name: 'Select',
    directives: { clickOutside },
    model: {
      prop: 'value',
      event: 'change'
    },
    props: {
      size: {
        validator (value) {
          return oneOf(value, ['large', 'small', 'mini', ''])
        },
        default: ''
      },
      placement: {
        validator (value) {
          return oneOf(value, ['top', 'bottom'])
        },
        default: 'bottom'
      },
      options: {
        type: Array,
        default () { return [] }
      },
      value: {
        type: [Number, String],
        default: ''
      },
      notFoundText: {
        type: String,
        default: '无匹配数据'
      },
      clearable: {
        type: Boolean,
        default: false
      },
      disabled: {
        type: Boolean,
        default: false
      }
    },
    inject: {
      form: { default: null }
    },
    data () {
      return {
        visible: false
      }
    },
    computed: {
      classes () {
        return [
          this.$LEO.prefix + 'select',
          this.placement,
          this.form ? this.form.size : this.size,
          {
            open    : this.visible,
            selected: this.selected && !this.disabled
          }
        ]
      },
      valueClasses () {
        return [
          this.$LEO.prefix + 'select-value',
          {
            disabled: this.disabled
          }
        ]
      },
      clearClasses () {
        return [
          this.$LEO.prefix + 'select-icon',
          {
            clear: this.canClear
          }
        ]
      },
      arrowsClasses () {
        return [this.$LEO.prefix + 'select-icon', 'arrows']
      },
      listClasses () {
        return this.$LEO.prefix + 'select-list'
      },
      textClasses () {
        return this.$LEO.prefix + 'select-text'
      },
      dropName () {
        return this.$LEO.prefix + 'drop'
      },
      list () {
        let options = this.options;

        return !options.some(option => typeof option !== 'object')
          ? options
          : options.map(option => {
              return { label: option, value: option }
            })
      },
      selected () {
        let selected = '';

        this.list.forEach(item => {
          if (item.value === this.value) selected = item.label
        });

        return selected
      },
      canClear () {
        return !this.disabled && this.clearable && this.selected !== ''
      }
    },
    methods: {
      handleOpen () {
        if (!this.disabled) this.visible = !this.visible
      },
      handleClose () {
        if (this.visible) this.visible = false
      },
      handleClear () {
        if (this.canClear) {
          this.handleChange('');
          this.$emit('on-clear')
        }
      },
      handleChange (val) {
        if (!this.disabled) {
          this.$emit('change', val);
          this.$emit('on-change', val)
        }
      }
    }
  }
</script>