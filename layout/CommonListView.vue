<template>
  <div class="common-list-view">
    <!-- 列表头部 -->
    <div v-if="showHeader" class="list-header">
      <slot name="header">
        <div class="header-content">
          <div class="header-cell name-cell">
            <span class="header-text">名称</span>
          </div>
          <div class="header-cell size-cell">
            <span class="header-text">大小</span>
          </div>
          <div class="header-cell type-cell">
            <span class="header-text">类型</span>
          </div>
          <div class="header-cell date-cell">
            <span class="header-text">修改时间</span>
          </div>
        </div>
      </slot>
    </div>
    
    <!-- 列表主体容器 -->
    <div class="list-body-container">
      <!-- 使用独立的框选控件 -->
      <SelectionBox
        :enabled="enableBoxSelection"
        :get-selectable-elements="getSelectableElements"
        :get-element-key="getElementKey"
        :multi-select="multiSelect"
        @selection-changed="handleBoxSelectionChanged"
        @empty-click="handleEmptyClick"
      >
        <!-- 列表内容 -->
        <div 
          ref="listBody"
          class="list-body" 
          @contextmenu="handleContextMenu"
        >
          <!-- 列表项 -->
          <div
            v-for="(item, index) in items"
            :key="getItemKey(item)"
            class="list-item"
            :class="{
              'even-row': index % 2 === 1,
              'selected': isItemSelected(item),
              'hover:bg-gray-50 dark:hover:bg-gray-700': !isItemSelected(item)
            }"
            :data-item-key="getItemKey(item)"
            :data-selectable="true"
            @click="handleItemClick(item, $event)"
            @dblclick="$emit('item-double-click', item)"
            @contextmenu="handleItemContextMenu(item, $event)"
          >
            <slot name="item" :item="item" :index="index" :selected="isItemSelected(item)">
              <!-- 默认列表项内容 -->
              <div class="item-content">
                <div class="item-cell name-cell">
                  <div class="item-icon">
                    <slot name="icon" :item="item">
                      <div class="w-5 h-5 bg-gray-300 rounded"></div>
                    </slot>
                  </div>
                  <span class="item-name">{{ getItemName(item) }}</span>
                </div>
                <div class="item-cell size-cell">
                  {{ getItemSize(item) }}
                </div>
                <div class="item-cell type-cell">
                  {{ getItemType(item) }}
                </div>
                <div class="item-cell date-cell">
                  {{ getItemDate(item) }}
                </div>
              </div>
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
import SelectionBox from './SelectionBox.vue'

export default {
  name: 'CommonListView',
  components: {
    SelectionBox
  },
  props: {
    // 列表数据
    items: {
      type: Array,
      default: () => []
    },
    // 选中的项目
    selectedItems: {
      type: Array,
      default: () => []
    },
    // 是否显示头部
    showHeader: {
      type: Boolean,
      default: true
    },
    // 获取项目唯一键的函数
    getItemKey: {
      type: Function,
      default: (item) => item.id || item.path || item.name
    },
    // 获取项目名称的函数
    getItemName: {
      type: Function,
      default: (item) => item.name || item.title || '未命名'
    },
    // 获取项目大小的函数
    getItemSize: {
      type: Function,
      default: function(item) { 
        return item.size ? this.formatFileSize(item.size) : '-'
      }
    },
    // 获取项目类型的的函数
    getItemType: {
      type: Function,
      default: (item) => item.type || '文件'
    },
    // 获取项目日期的函数
    getItemDate: {
      type: Function,
      default: function(item) { 
        return item.date ? this.formatDate(item.date) : '-'
      }
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
    // 格式化文件大小
    formatFileSize(size) {
      if (!size || size === 0) return '-'
      
      const units = ['B', 'KB', 'MB', 'GB', 'TB']
      let unitIndex = 0
      let fileSize = size
      
      while (fileSize >= 1024 && unitIndex < units.length - 1) {
        fileSize /= 1024
        unitIndex++
      }
      
      return `${fileSize.toFixed(unitIndex === 0 ? 0 : 1)} ${units[unitIndex]}`
    },

    // 格式化日期
    formatDate(date) {
      if (!date) return '-'

      const d = new Date(date)
      const now = new Date()
      const diffTime = now.getTime() - d.getTime()
      const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24))

      if (diffDays === 0) {
        return d.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' })
      } else if (diffDays < 7) {
        return `${diffDays}天前`
      } else {
        return d.toLocaleDateString('zh-CN')
      }
    },

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

    // 处理框选变化
    handleBoxSelectionChanged(selectedElements) {
      const selectedItems = []
      
      selectedElements.forEach(({ key }) => {
        const item = this.items.find(item => this.getItemKey(item) === key)
        if (item) {
          selectedItems.push(item)
        }
      })

      this.$emit('selection-changed', selectedItems)
      console.log("handleBoxSelectionChanged", selectedItems)
    },

    // 处理空白区域点击
    handleEmptyClick(event) {
      this.$emit('empty-click', event)
      this.$emit('selection-changed', [])
    },

    // 处理右键菜单
    handleContextMenu(event) {
      const itemElement = event.target.closest('.list-item')
      if (itemElement) {
        return // 如果点击在列表项上，不处理空白区域右键
      }

      event.preventDefault()
      event.stopPropagation()

      // 设置右键菜单位置
      const listBodyElement = this.$refs.listBody
      if (listBodyElement) {
        const rect = listBodyElement.getBoundingClientRect()
        this.contextMenuPosition = {
          x: event.clientX - rect.left,
          y: event.clientY - rect.top
        }
        this.contextMenuVisible = true
      }

      this.$emit('context-menu', null, [], event)
    },

    // 处理列表项点击
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

    // 处理列表项右键菜单
    handleItemContextMenu(item, event) {
      event.preventDefault()
      
      // 设置右键菜单位置
      const listBodyElement = this.$refs.listBody
      if (listBodyElement) {
        const rect = listBodyElement.getBoundingClientRect()
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
.common-list-view {
  @apply flex flex-col h-full min-h-0;
}

.list-header {
  @apply flex-shrink-0 bg-gray-50 dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700;
}

.header-content {
  @apply flex;
}

.header-cell {
  @apply px-3 py-2 text-xs font-medium text-gray-600 dark:text-gray-400 cursor-pointer select-none;
  @apply hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors duration-200;
}

.header-text {
  @apply flex items-center gap-1;
}

.name-cell {
  @apply flex-1 min-w-48;
}

.size-cell {
  @apply w-24;
}

.type-cell {
  @apply w-32;
}

.date-cell {
  @apply w-36;
}

.list-body-container {
  @apply flex-1 min-h-0 relative;
}

.list-body {
  @apply w-full h-full overflow-y-auto relative;
}

.list-item {
  @apply flex min-h-0 border-b border-gray-100 dark:border-gray-700 cursor-pointer;
  @apply transition-colors duration-200;
}

.list-item.even-row {
  @apply bg-gray-50 dark:bg-gray-800;
}

.list-item.selected {
  @apply bg-blue-500 text-white;
}

.list-item.selected:hover {
  @apply bg-blue-600;
}

.list-item.selected * {
  @apply text-white;
}

.item-content {
  @apply flex w-full;
}

.item-cell {
  @apply px-3 py-2 flex items-center text-sm overflow-hidden;
}

.item-cell.name-cell {
  @apply gap-2;
}

.item-icon {
  @apply flex-shrink-0;
}

.item-name {
  @apply whitespace-nowrap overflow-hidden text-ellipsis;
}

/* 深色主题下的选中样式 */
.dark .list-item.selected {
  @apply bg-blue-600 text-gray-900;
}

.dark .list-item.selected:hover {
  @apply bg-blue-700;
}

.dark .list-item.selected * {
  @apply text-gray-900;
}

/* 右键菜单样式 */
.context-menu {
  @apply absolute bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg shadow-lg;
  @apply py-1 z-50 min-w-48;
}
</style>
