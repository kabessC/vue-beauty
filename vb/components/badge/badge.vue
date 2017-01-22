<template>
  <span>
    <slot></slot>
    <transition>
    </transition>
    <span :class="ant-badge-status-text">{text}</span>
  </span>
</template>
<script>
export default {
  name: 'vBadge',
  data: () => ({
    prefixCls: 'ant-badge',
    status: '',
  }),
  props: {
    count: {
      type: Number,
      default: 0
    },
    overflowCount: {
      type: Number,
      default: 99
    },
    dot: {
      type: Boolean,
      default: false
    },
    status: {
      type: String,
      default: '',
      validator: function(value) {
        return ['', 'success', 'processing', 'default', 'error', 'warning'].indexOf(value) > -1;
      }
    },
    text: {
      type: String,
      default: ''
    }
  },
  computed: {
    badgeCls: function() {
      return [
        this.prefixCls,
        {
          [`${this.prefixCls}-status`]: !!status,
          [`${this.prefixCls}-not-a-wrapper`]: !!this.$slots.default
        }
      ];
    }
  }
}
</script>