<template>
  <ul :class="treeCls">
    <li
      v-for="(item, index) in dataSource"
      :class="treeNodeCls(item)"
    >
      <span
        :class="switcherCls(item)"
        @click="setExpand(item.disabled, index)"
      ></span>
      <span
        v-if="checkable"
        :class="checkboxCls(item)"
        @click.prevent="setCheck(item.disabled || item.disableCheckbox, index)"
      >
          <span :class="prefixCls + '-checkbox-inner'"></span>
      </span>
      <a
        :title="item.title"
        :class="selectHandleCls(item)"
        @click="setSelect(item.disabled, index)"
      >
          <span :class="prefixCls + '-title'" v-html="item.title"></span>
      </a>
      <tree
        v-if="!item.isLeaf"
        :data-source="item.node"
        :eventKey="`${eventKey}-${index}`"
        :multiple="multiple"
        :checkable="checkable"
        :class="{[prefixCls+'-child-tree-open']: item.expand}"
        v-show="item.expand"
        transition="slide-up"></tree>
    </li>
  </ul>
</template>
<script>
import emitter from '../../mixins/emitter';

export default {
  name: 'Tree',
  mixins: [emitter],
  props: {
    eventKey: {
      type: String,
      default: '0'
    },
    dataSource: {
      type: Array,
      default: () => []
    },
    multiple: {
      type: Boolean,
      default: false
    },
    checkable: {
      type: Boolean,
      default: false
    },
    onSelect: Function,
    onCheck: Function
  },
  data: () => ({
    prefixCls: 'ant-tree'
  }),
  computed: {
    treeCls() {
      if (this.eventKey === '0') {
        return this.prefixCls;
      } else {
        return `${this.prefixCls}-child-tree`;
      }
    }
  },
  watch: {
    dataSource() {
      if (this.eventKey === '0') {
        this.setKey();
        this.preHandle();
      }
    }
  },
  mounted() {
    this.setKey();
    this.preHandle();
    
    this.$on('nodeSelected', (ori, selected) => {
      if (this.eventKey !== '0') {
        return true;
      }
      if (!this.multiple && selected){
        if (this !== ori) {
          for(let i = 0; i < this.dataSource.length; i++) {
            this.$set(this.dataSource[i], 'selected', false);
          }
        }
        this.broadcast('cancelSelected', ori);
      }
      if(this.onSelect) {
        this.$nextTick(() =>{
          this.onSelect(this.getSelectedNodes());
        });
      }
    });

    this.$on('cancelSelected', ori => {
      this.broadcast('cancelSelected', ori);

      if(this !== ori) {
        for(let i = 0; i < this.dataSource.length; i++) {
          this.$set(this.dataSource[i], 'selected', false);
        }
      }
    });

    this.$on('parentChecked', (params) => {
      if(this.eventKey == params.eventKey || this.eventKey.startsWith(params.eventKey + '-')) {
        for(let i = 0; i < this.dataSource.length; i++) {
          this.$set(this.dataSource[i], 'checked', params.status);
          this.$set(this.dataSource[i], 'childrenCheckedStatus', params.status ? 2 : 0);
        }
        this.broadcast('Tree', 'parentChecked', params);
      }
    });

    this.$on('childChecked', (params) => {
      if (this.eventKey === '0' && this.onCheck) {
        this.$nextTick(() => {
          this.onCheck(this.getCheckedNodes());
        });
      }
      if (this === params.origin) {
        return;
      }

      for (let [i, item] of this.dataSource.entries()) {
        if (`${this.eventKey}-${i}` == params.eventKey) {
          let temp = this.getChildrenCheckedStatus(item.node);

          if (temp != item.childrenCheckedStatus) {
            this.$set(this.dataSource[i], 'checked', temp ? true : false);
            this.$set(this.dataSource[i], 'childrenCheckedStatus', temp);

            if (this.eventKey !== '0') {
              this.dispatch('Tree', 'childChecked', params);
            }
          }
        }
      }
    });
  },
  methods: {
    treeNodeCls(item) {
      return [
        {
          [`${this.prefixCls}-treenode-disabled`]: item.disabled
        }
      ];
    },
    switcherCls(item) {
      const expandedState = item.expand ? 'open' : 'close';

      return [
        `${this.prefixCls}-switcher`,
        {
          [`${this.prefixCls}-switcher-disabled`]: item.disabled,
          [`${this.prefixCls}-switcher-noop`]: item.isLeaf,
          [`${this.prefixCls}-noline_docu`]: item.isLeaf,
          [`${this.prefixCls}-noline_${expandedState}`]: !item.isLeaf
        }
      ];
    },
    checkboxCls(item) {
      return [
        `${this.prefixCls}-checkbox`,
        {
          [`${this.prefixCls}-checkbox-disabled`]: item.disabled || item.disableCheckbox,
          [`${this.prefixCls}-checkbox-checked`]: item.checked && item.childrenCheckedStatus === 2,
          [`${this.prefixCls}-checkbox-indeterminate`]: item.checked && item.childrenCheckedStatus === 1
        }
      ];
    },
    selectHandleCls(item) {
      const wrap = `${this.prefixCls}-node-content-wrapper`;

      return [
        wrap,
        `${wrap}-normal`,
        {
          [`${this.prefixCls}-node-selected`]: !item.disable && item.selected
        }
      ];
    },
    setKey() {
      for (let i = 0; i < this.dataSource.length; i++) {
        this.dataSource[i].eventKey = `${this.eventKey}-${i}`;
      }
    },
    preHandle() {
      for (let [i, item] of this.dataSource.entries()) {
        if (!item.node || !item.node.length) {
          this.$set(this.dataSource[i], 'isLeaf', true);
          this.$set(this.dataSource[i], 'childrenCheckedStatus', 2);
          continue;
        }
        if (item.checked && !item.childrenCheckedStatus) {
          this.$set(this.dataSource[i], 'childrenCheckedStatus', 2);
          this.broadcast('Tree', 'parentChecked', { status: true, eventKey: `${this.eventKey}-${i}` });
        } else {
          let status = this.getChildrenCheckedStatus(item.node);
          this.$set(this.dataSource[i], 'childrenCheckedStatus', status);
          
          if (status !== 0) {
            this.$set(this.dataSource[i] ,'checked', true);
          }
        }
      }
    },
    setExpand(disabled, index) {
      if (!disabled) {
        this.$set(this.dataSource[index], 'expand', !this.dataSource[index].expand);
      }
    },
    setSelect(disabled, index) {
      if (!disabled) {
        const selected = !this.dataSource[index].selected;

        if (this.multiple || !selected){
          this.$set(this.dataSource[index], 'selected', selected);
        } else {
          for (let i = 0; i < this.dataSource.length; i++) {
            if (i == index) {
              this.$set(this.dataSource[i], 'selected', true);
            } else {
              this.$set(this.dataSource[i], 'selected', false);
            }
          }
        }
        this.$emit('nodeSelected', this,selected);
      }
    },
    setCheck(disabled, index) {
      if (disabled) {
        return;
      }

      const checked = !this.dataSource[index].checked;
      this.$set(this.dataSource[index], 'checked', checked);
      this.$set(this.dataSource[index], 'childrenCheckedStatus', checked ? 2 : 0);
      this.dispatch('Tree', 'childChecked', { origin: this, eventKry: this.eventKey });
      this.broadcast('Tree', 'parentChecked', { status: checked, eventKey: `${this.eventKey}-${index}` });
    },
    getNodes(data, opt) {
      data = data || this.dataSource;
      let res = [];

      for (let node of data) {
        let tmp = true;
        for (let [key, value] of Object.entries(opt)) {
          if (node[key] !== value) {
            tmp = false;
            break;
          }
        }
        if (tmp) {
          res.push(node);
        }
        if (node.node && node.node.length) {
          res = res.concat(this.getNodes(node.node, opt));
        }
      }
      return res;
    },
    getSelectedNodes() {
      return this.getNodes(this.dataSource, { selected: true });
    },
    getCheckedNodes() {
      return this.getNodes(this.dataSource, { checked: true, childrenCheckedStatus: 2 });
    },
    getChildrenCheckedStatus(children) {
      let checkNum = 0,
          child_childrenAllChecked = true;

      for (let child of children) {
        if (child.checked) {
          checkNum++;
        }
        if (child.childrenCheckedStatus !== 2) {
          child_childrenAllChecked = false;
        }
      }
      //全选
      if (checkNum === children.length) {
        return child_childrenAllChecked ? 2 : 1;
      //部分选择
      } else if (checkNum > 0) {
        return 1;
      } else {
        return 0;
      }
    }
  }
}
</script>