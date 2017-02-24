<script>
export default {
  data: function() {
    return {
      treeData: [{
        title: 'parent 1',
        selected: true,
        node: [{
          title: 'parent 1-0',
          expand: true,
          disabled: true,
          node: [{
            title: 'leaf',
            disableCheckbox: true
          }, {
            title: 'leaf',
          }]
        }, {
          title: 'parent 1-1',
          checked: true,
          node: [{
            title: "<span style='color: #08c'>sss</span>",
          }]
        }]
      }]
    }
  },
  methods: {
    selectFn(data) {
      console.log(data);
    },
    checkFn(data) {
      console.log(data);
    }
  }
}
</script>

# Tree 树形控件

## 何时使用
文件夹、组织架构、生物分类、国家地区等等，世间万物的大多数结构都是树形结构。使用树控件可以完整展现其中的层级关系，并具有展开收起选择等交互功能。

## 代码演示

:::demo
<summary>
  #### 基本
  最简单的用法，展示可勾选，可选中，禁用，默认展开等功能。
</summary>

```html
<template>
  <v-tree
    :dataSource="treeData"
    :checkable="true"
    :multiple="true"
    :on-select="selectFn"
    :on-check="checkFn"
  >
  </v-tree>
</template>
<script>
export default {
  data: function() {
    return {
      treeData: [{
        title: 'parent 1',
        selected: true,
        node: [{
          title: 'parent 1-0',
          expand: true,
          disabled: true,
          node: [{
            title: 'leaf',
            disableCheckbox: true
          }, {
            title: 'leaf',
          }]
        }, {
          title: 'parent 1-1',
          checked: true,
          node: [{
            title: "<span style='color: #08c'>sss</span>",
          }]
        }]
      }]
    }
  },
  methods: {
    selectFn(data) {
      console.log(data);
    },
    checkFn(data) {
      console.log(data);
    }
  }
}
</script>
```

:::