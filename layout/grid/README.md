# CommonGridView 组件

一个通用的网格视图组件，支持大中小三种尺寸，具备框选、多选、右键菜单等功能。

## 特性

- ✅ 支持大、中、小三种网格尺寸
- ✅ 支持框选功能
- ✅ 支持多选（Ctrl/Cmd + 点击、Shift + 点击）
- ✅ 支持右键菜单
- ✅ 支持自定义图标和内容
- ✅ 支持深色主题
- ✅ 响应式布局
- ✅ Vue 2.7 兼容

## 基本用法

```vue
<template>
  <CommonGridView
    :items="items"
    :selected-items="selectedItems"
    :grid-size="gridSize"
    :multi-select="true"
    :enable-box-selection="true"
    @selection-changed="handleSelectionChanged"
    @item-click="handleItemClick"
  >
    <!-- 自定义图标 -->
    <template #icon="{ item }">
      <FileIcon :file="item" :size="32" />
    </template>
  </CommonGridView>
</template>

<script>
import CommonGridView from '@/el-ui/layout/grid/CommonGridView.vue'

export default {
  components: {
    CommonGridView
  },
  data() {
    return {
      items: [
        { id: 1, name: '文件1.txt', size: 1024 },
        { id: 2, name: '文件2.jpg', size: 2048 },
      ],
      selectedItems: [],
      gridSize: 'medium'
    }
  },
  methods: {
    handleSelectionChanged(items) {
      this.selectedItems = items
    },
    handleItemClick(item, event) {
      console.log('点击了:', item)
    }
  }
}
</script>
```

## Props

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `items` | Array | `[]` | 要显示的项目数组 |
| `selectedItems` | Array | `[]` | 当前选中的项目数组 |
| `gridSize` | String | `'medium'` | 网格尺寸：`'small'`、`'medium'`、`'large'` |
| `getItemKey` | Function | `(item) => item.id \|\| item.path \|\| item.name` | 获取项目唯一键的函数 |
| `getItemName` | Function | `(item) => item.name \|\| item.title \|\| '未命名'` | 获取项目名称的函数 |
| `getItemSize` | Function | `(item) => formatFileSize(item.size)` | 获取项目大小的函数 |
| `multiSelect` | Boolean | `true` | 是否启用多选 |
| `enableBoxSelection` | Boolean | `true` | 是否启用框选 |

## Events

| 事件名 | 参数 | 说明 |
|--------|------|------|
| `selection-changed` | `(items: Array)` | 选择变化时触发 |
| `item-click` | `(item: Object, event: Event)` | 项目点击时触发 |
| `item-double-click` | `(item: Object)` | 项目双击时触发 |
| `item-context-menu` | `(item: Object, selectedItems: Array, event: Event)` | 项目右键菜单时触发 |
| `context-menu` | `(item: Object, items: Array, event: Event)` | 空白区域右键菜单时触发 |
| `empty-click` | `(event: Event)` | 空白区域点击时触发 |
| `selection-finished` | `(items: Array)` | 框选完成时触发 |
| `selection-cleared` | `(items: Array)` | 框选清空时触发 |

## Slots

### `icon` 插槽

用于自定义项目图标。

```vue
<template #icon="{ item }">
  <FileIcon :file="item" :size="32" />
</template>
```

### `item` 插槽

用于完全自定义项目内容。

```vue
<template #item="{ item, index, selected }">
  <div class="custom-item">
    <img :src="item.thumbnail" />
    <span>{{ item.name }}</span>
  </div>
</template>
```

### `context-menu` 插槽

用于自定义右键菜单。

```vue
<template #context-menu="{ visible, position, items }">
  <div v-if="visible" class="custom-menu" :style="{ left: position.x + 'px', top: position.y + 'px' }">
    <div @click="handleAction('open', items)">打开</div>
    <div @click="handleAction('copy', items)">复制</div>
  </div>
</template>
```

## 网格尺寸说明

### 小尺寸 (small)
- 网格列宽：最小 75px
- 图标尺寸：24px
- 文字大小：12px
- 不显示详细信息

### 中尺寸 (medium) - 默认
- 网格列宽：最小 110px
- 图标尺寸：32px
- 文字大小：14px
- 显示文件大小

### 大尺寸 (large)
- 网格列宽：最小 150px
- 图标尺寸：48px
- 文字大小：16px
- 显示文件大小
- 支持图片缩略图

## 选择操作

### 单选
- 直接点击项目

### 多选
- **Ctrl/Cmd + 点击**：切换选择状态
- **Shift + 点击**：范围选择（从上次选中的项目到当前项目）

### 框选
- 在空白区域按住鼠标左键并拖动
- 框选范围内的项目会被选中

### 全选
- **Ctrl/Cmd + A**：选择所有项目

## 样式定制

组件使用 Tailwind CSS 类名，可以通过以下方式定制样式：

```vue
<style>
/* 自定义网格项样式 */
.grid-item {
  @apply border-2 border-transparent;
}

.grid-item.selected {
  @apply border-blue-500 bg-blue-50;
}

/* 自定义网格尺寸 */
.grid-size-custom .grid-container {
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
}
</style>
```

## 迁移指南

### 从旧版 GridView 迁移

1. 替换组件引用：
```vue
// 旧版
import GridView from './GridView.vue'

// 新版
import CommonGridView from '@/el-ui/layout/grid/CommonGridView.vue'
```

2. 更新模板：
```vue
<!-- 旧版 -->
<GridView :files="files" />

<!-- 新版 -->
<CommonGridView 
  :items="files"
  :selected-items="selectedFiles"
  :grid-size="gridSize"
  @selection-changed="handleSelectionChanged"
/>
```

3. 更新事件处理：
```javascript
// 旧版
handleFileClick(file, event) {
  // 处理文件点击
}

// 新版
handleSelectionChanged(items) {
  this.selectedFiles = items
}
```

## 示例

查看 `CommonGridViewExample.vue` 和 `CommonGridViewTest.vue` 文件获取完整的使用示例。 