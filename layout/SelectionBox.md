# SelectionBox 框选控件

一个独立的、可复用的框选控件，支持在任何容器中进行框选操作。

## 功能特性

- ✅ 独立的框选功能，不依赖特定组件
- ✅ 支持自定义可选择元素
- ✅ 支持多选和单选模式
- ✅ 自动处理鼠标事件
- ✅ 支持空白区域点击
- ✅ 可复用于任何场景

## 基本用法

```vue
<template>
  <SelectionBox
    :enabled="true"
    :get-selectable-elements="getSelectableElements"
    :get-element-key="getElementKey"
    @selection-changed="handleSelectionChanged"
    @empty-click="handleEmptyClick"
  >
    <!-- 你的内容 -->
    <div class="container">
      <div
        v-for="item in items"
        :key="item.id"
        class="item"
        :data-selectable="true"
        :data-key="item.id"
        @click="handleItemClick(item, $event)"
      >
        {{ item.name }}
      </div>
    </div>
  </SelectionBox>
</template>

<script>
import SelectionBox from './SelectionBox.vue'

export default {
  components: {
    SelectionBox
  },
  data() {
    return {
      items: [
        { id: '1', name: '项目1' },
        { id: '2', name: '项目2' },
        { id: '3', name: '项目3' }
      ],
      selectedItems: []
    }
  },
  methods: {
    // 获取可选择元素列表
    getSelectableElements() {
      return this.$el.querySelectorAll('[data-selectable]')
    },

    // 获取元素唯一标识
    getElementKey(element) {
      return element.dataset.key
    },

    // 处理选择变化
    handleSelectionChanged(selectedElements) {
      this.selectedItems = selectedElements.map(({ key }) => key)
    },

    // 处理空白区域点击
    handleEmptyClick(event) {
      this.selectedItems = []
    },

    // 处理项目点击
    handleItemClick(item, event) {
      console.log('点击了:', item.name)
    }
  }
}
</script>
```

## Props

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `enabled` | Boolean | `true` | 是否启用框选功能 |
| `getSelectableElements` | Function | `() => []` | 获取可选择元素列表的函数 |
| `getElementKey` | Function | `(element) => element.dataset?.key || element.id || null` | 获取元素唯一标识的函数 |
| `multiSelect` | Boolean | `true` | 是否允许多选 |
| `selectedElements` | Array | `[]` | 当前选中的元素 |

## Events

| 事件名 | 参数 | 说明 |
|--------|------|------|
| `selection-changed` | `selectedElements: Array<{element: HTMLElement, key: string}>` | 选择变化时触发 |
| `empty-click` | `event: MouseEvent` | 点击空白区域时触发 |

## 与 CommonListView 配合使用

SelectionBox 已经集成到 CommonListView 中，可以直接使用：

```vue
<template>
  <CommonListView
    :items="items"
    :selected-items="selectedItems"
    :enable-box-selection="true"
    :multi-select="true"
    @selection-changed="handleSelectionChanged"
    @item-click="handleItemClick"
    @empty-click="handleEmptyClick"
  >
    <!-- 自定义列表项内容 -->
    <template #item="{ item, selected }">
      <div class="custom-item">
        <span>{{ item.name }}</span>
        <span v-if="selected">✓</span>
      </div>
    </template>
  </CommonListView>
</template>
```

## 注意事项

1. **可选择元素标记**: 需要给可选择元素添加 `data-selectable="true"` 属性
2. **唯一标识**: 需要给可选择元素添加唯一标识，通常使用 `data-key` 属性
3. **容器样式**: SelectionBox 容器需要设置 `position: relative`
4. **事件冲突**: 如果容器内有其他点击事件，需要确保不会与框选冲突

## 样式定制

SelectionBox 使用以下 CSS 类名，可以通过深度选择器进行样式定制：

```css
/* 框选容器 */
.selection-box-container {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

/* 框选区域 */
.selection-box {
  position: absolute;
  border: 2px solid #409EFF;
  background: rgba(64, 158, 255, 0.1);
  pointer-events: none;
  z-index: 1000;
}

/* 深色主题 */
[data-theme="dark"] .selection-box {
  border-color: #79BBFF;
  background: rgba(121, 187, 255, 0.1);
}
```

## 测试

使用 `SelectionBoxTest.vue` 组件可以测试框选功能：

```vue
<template>
  <SelectionBoxTest />
</template>

<script>
import SelectionBoxTest from './SelectionBoxTest.vue'

export default {
  components: {
    SelectionBoxTest
  }
}
</script>
``` 