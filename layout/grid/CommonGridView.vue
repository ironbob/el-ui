<template>
  <div class="common-grid-view" :class="`grid-size-${gridSize}`">
    <!-- 网格主体容器 -->
    <div class="grid-body-container">
      <!-- 使用独立的框选控件 -->
      <SelectionBox
        :enabled="enableBoxSelection"
        :get-selectable-elements="getSelectableElements"
        :get-element-key="getElementKey"
        :multi-select="multiSelect"
        @selection-finished="handleBoxSelectionFinished"
        @selection-cleared="handleBoxSelectionCleared"
        @empty-click="handleEmptyClick"
      >
        <!-- 网格内容 -->
        <div 
          ref="gridBody"
          class="grid-container" 
          @contextmenu="handleContextMenu"
        >
          <!-- 网格项 -->
          <div
            v-for="(item, index) in items"
            :key="getItemKey(item)"
            class="grid-item"
            :class="{
              'selected': isItemSelected(item),
              'hover:bg-gray-50 dark:hover:bg-gray-700': !isItemSelected(item)
            }"
            :data-item-key="getItemKey(item)"
            :data-selectable="true"
            @click="handleItemClick(item, $event)"
            @dblclick="$emit('item-double-click', item)"
            @contextmenu="handleItemContextMenu(item, $event)"
          >
            <!-- 完全由插槽控制项目内容 -->
            <slot name="item" :item="item" :index="index" :selected="isItemSelected(item)">
              <!-- 如果没有提供插槽内容，显示空内容 -->
            </slot>
          </div>
        </div>
      </SelectionBox>

      <!-- 右键菜单 -->
      <slot name="context-menu" :visible="contextMenuVisible" :position="contextMenuPosition" :items="contextMenuItems">
        <!-- 默认右键菜单 -->
        <div
          v-if="contextMenuVisible"
          class="context-menu"
          :style="{
            left: contextMenuPosition.x + 'px',
            top: contextMenuPosition.y + 'px'
          }"
        >
          <slot name="context-menu-items" :items="contextMenuItems">
            <!-- 默认菜单项 -->
          </slot>
        </div>
      </slot>
    </div>
  </div>
</template>

<script>
import SelectionBox from '../SelectionBox.vue'

export default {
  name: 'CommonGridView',
  components: {
    SelectionBox
  },
  props: {
    // 网格数据
    items: {
      type: Array,
      default: () => []
    },
    // 选中的项目
    selectedItems: {
      type: Array,
      default: () => []
    },
    // 网格尺寸：small, medium, large
    gridSize: {
      type: String,
      default: 'medium',
      validator: value => ['small', 'medium', 'large'].includes(value)
    },
    // 获取项目唯一键的函数
    getItemKey: {
      type: Function,
      default: (item) => item.id || item.path || item.name
    },
    // 是否启用多选
    multiSelect: {
      type: Boolean,
      default: true
    },
    // 是否启用框选
    enableBoxSelection: {
      type: Boolean,
      default: true
    }
  },

  data() {
    return {
      // 右键菜单状态
      contextMenuVisible: false,
      contextMenuPosition: { x: 0, y: 0 },
      contextMenuItems: []
    }
  },

  mounted() {
    // 添加全局点击事件监听器来隐藏右键菜单
    document.addEventListener('click', this.handleGlobalClick)
  },

  beforeDestroy() {
    // 移除全局事件监听器
    document.removeEventListener('click', this.handleGlobalClick)
  },

  methods: {
    // 检查项目是否被选中
    isItemSelected(item) {
      return this.selectedItems.some(selected => this.getItemKey(selected) === this.getItemKey(item))
    },

    // 获取可选择元素列表（供 SelectionBox 使用）
    getSelectableElements() {
      return this.$el.querySelectorAll('[data-selectable]')
    },

    // 获取元素唯一标识（供 SelectionBox 使用）
    getElementKey(element) {
      return element.dataset.itemKey
    },

    // 处理框选完成事件
    handleBoxSelectionFinished(selectedElements) {
      const selectedItems = []
      
      selectedElements.forEach(({ key }) => {
        const item = this.items.find(item => this.getItemKey(item) === key)
        if (item) {
          selectedItems.push(item)
        }
      })

      // 发送选择完成事件
      this.$emit('selection-finished', selectedItems)
    },

    // 处理框选清空事件
    handleBoxSelectionCleared() {
      // 框选开始时清空选择
      this.$emit('selection-cleared', [])
    },

    // 处理空白区域点击
    handleEmptyClick(event) {
      this.$emit('empty-click', event)
      this.$emit('selection-changed', [])
    },

    // 处理右键菜单
    handleContextMenu(event) {
      const itemElement = event.target.closest('.grid-item')
      if (itemElement) {
        return // 如果点击在网格项上，不处理空白区域右键
      }

      event.preventDefault()
      event.stopPropagation()

      // 设置右键菜单位置
      const gridBodyElement = this.$refs.gridBody
      if (gridBodyElement) {
        const rect = gridBodyElement.getBoundingClientRect()
        this.contextMenuPosition = {
          x: event.clientX - rect.left,
          y: event.clientY - rect.top
        }
        this.contextMenuVisible = true
      }

      this.$emit('context-menu', null, [], event)
    },

    // 处理网格项点击
    handleItemClick(item, event) {
      if (!this.multiSelect) {
        // 单选模式
        this.$emit('selection-changed', [item])
        this.$emit('item-click', item, event)
        return
      }

      // 多选模式
      if (event.ctrlKey || event.metaKey) {
        // Ctrl/Cmd + 点击：切换选择状态
        const isSelected = this.isItemSelected(item)
        if (isSelected) {
          // 取消选择
          const newSelection = this.selectedItems.filter(selected => 
            this.getItemKey(selected) !== this.getItemKey(item)
          )
          this.$emit('selection-changed', newSelection)
        } else {
          // 添加到选择
          const newSelection = [...this.selectedItems, item]
          this.$emit('selection-changed', newSelection)
        }
      } else if (event.shiftKey && this.selectedItems.length > 0) {
        // Shift + 点击：范围选择
        const lastSelected = this.selectedItems[this.selectedItems.length - 1]
        const lastIndex = this.items.findIndex(f => this.getItemKey(f) === this.getItemKey(lastSelected))
        const currentIndex = this.items.findIndex(f => this.getItemKey(f) === this.getItemKey(item))

        if (lastIndex !== -1 && currentIndex !== -1) {
          const startIndex = Math.min(lastIndex, currentIndex)
          const endIndex = Math.max(lastIndex, currentIndex)
          const rangeSelection = this.items.slice(startIndex, endIndex + 1)
          this.$emit('selection-changed', rangeSelection)
        }
      } else {
        // 普通点击：单选
        this.$emit('selection-changed', [item])
      }

      this.$emit('item-click', item, event)
    },

    // 处理网格项右键菜单
    handleItemContextMenu(item, event) {
      event.preventDefault()
      
      // 设置右键菜单位置
      const gridBodyElement = this.$refs.gridBody
      if (gridBodyElement) {
        const rect = gridBodyElement.getBoundingClientRect()
        this.contextMenuPosition = {
          x: event.clientX - rect.left,
          y: event.clientY - rect.top
        }
        this.contextMenuVisible = true
      }
      
      this.$emit('item-context-menu', item, this.selectedItems, event)
    },

    // 滚动到指定项目
    scrollToItem(item) {
      const itemElement = this.$el.querySelector(`[data-item-key="${this.getItemKey(item)}"]`)
      if (itemElement) {
        itemElement.scrollIntoView({ behavior: 'smooth', block: 'nearest' })
      }
    },

    // 隐藏右键菜单
    hideContextMenu() {
      this.contextMenuVisible = false
    },

    // 处理全局点击事件
    handleGlobalClick(event) {
      // 如果点击的不是右键菜单区域，则隐藏菜单
      if (this.contextMenuVisible && !event.target.closest('.context-menu')) {
        this.hideContextMenu()
      }
    }
  }
}
</script>

<style scoped>
.common-grid-view {
  @apply flex flex-col h-full min-h-0;
}

.grid-body-container {
  @apply flex-1 min-h-0 relative;
}

.grid-container {
  @apply w-full h-full overflow-y-auto relative p-4;
  display: grid;
  gap: 12px;
}

/* 小图标网格 - 接近正方形 */
.grid-size-small .grid-container {
  grid-template-columns: repeat(auto-fill, minmax(75px, 1fr));
  gap: 6px;
  padding: 4px;
}

.grid-size-small .grid-item {
  aspect-ratio: 1;
  max-height: 75px;
}

/* 中等图标网格 */
.grid-size-medium .grid-container {
  grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
  gap: 8px;
  padding: 4px;
}

/* 大图标网格 */
.grid-size-large .grid-container {
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 10px;
  padding: 4px;
}

.grid-item {
  @apply flex flex-col items-center p-2 rounded-lg cursor-pointer transition-all duration-200;
  @apply bg-transparent border border-transparent;
  @apply hover:bg-gray-50 dark:hover:bg-gray-700;
  @apply hover:transform hover:scale-105;
  min-height: 0;
}

.grid-item.selected {
  @apply bg-blue-500 text-white border-blue-500;
  @apply transform scale-105;
}

.grid-item.selected:hover {
  @apply bg-blue-600 border-blue-600;
}

.grid-item.selected * {
  @apply text-white;
}

/* 深色主题下的选中样式 */
.dark .grid-item.selected {
  @apply bg-blue-600 text-white;
}

.dark .grid-item.selected:hover {
  @apply bg-blue-700;
}

.dark .grid-item.selected * {
  @apply text-white;
}

/* 右键菜单样式 */
.context-menu {
  @apply absolute bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg shadow-lg;
  @apply py-1 z-50 min-w-48;
}
</style> 