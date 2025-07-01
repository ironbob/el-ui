<template>
  <div class="common-grid-view-example">
    <!-- 控制面板 -->
    <div class="control-panel">
      <div class="control-group">
        <label>网格尺寸：</label>
        <select v-model="currentGridSize" class="control-select">
          <option value="small">小</option>
          <option value="medium">中</option>
          <option value="large">大</option>
        </select>
      </div>
      <div class="control-group">
        <label>
          <input type="checkbox" v-model="enableMultiSelect" />
          多选
        </label>
      </div>
      <div class="control-group">
        <label>
          <input type="checkbox" v-model="enableBoxSelection" />
          框选
        </label>
      </div>
    </div>

    <!-- CommonGridView 组件 -->
    <CommonGridView
      :items="items"
      :selected-items="selectedItems"
      :grid-size="currentGridSize"
      :multi-select="enableMultiSelect"
      :enable-box-selection="enableBoxSelection"
      :get-item-key="getItemKey"
      :get-item-name="getItemName"
      :get-item-size="getItemSize"
      @selection-changed="handleSelectionChanged"
      @item-click="handleItemClick"
      @item-double-click="handleItemDoubleClick"
      @context-menu="handleContextMenu"
      @item-context-menu="handleItemContextMenu"
      @empty-click="handleEmptyClick"
    >
      <!-- 自定义图标插槽 -->
      <template #icon="{ item }">
        <FileIcon :file="item" :size="getIconSize()" />
      </template>

      <!-- 自定义右键菜单 -->
      <template #context-menu="{ visible, position, items }">
        <div
          v-if="visible"
          class="custom-context-menu"
          :style="{
            left: position.x + 'px',
            top: position.y + 'px'
          }"
        >
          <div class="menu-item" @click="handleMenuAction('open', items)">
            打开
          </div>
          <div class="menu-item" @click="handleMenuAction('copy', items)">
            复制
          </div>
          <div class="menu-item" @click="handleMenuAction('delete', items)">
            删除
          </div>
        </div>
      </template>
    </CommonGridView>

    <!-- 状态信息 -->
    <div class="status-info">
      <p>选中项目数量: {{ selectedItems.length }}</p>
      <p>当前网格尺寸: {{ currentGridSize }}</p>
    </div>
  </div>
</template>

<script>
import CommonGridView from './CommonGridView.vue'
import FileIcon from '../../../components/common/FileIcon.vue'

export default {
  name: 'CommonGridViewExample',
  components: {
    CommonGridView,
    FileIcon
  },
  data() {
    return {
      // 网格数据
      items: [
        { id: 1, name: '文档.docx', size: 1024000, type: 'document', path: '/documents/doc.docx' },
        { id: 2, name: '图片.jpg', size: 2048000, type: 'image', path: '/images/photo.jpg' },
        { id: 3, name: '视频.mp4', size: 52428800, type: 'video', path: '/videos/movie.mp4' },
        { id: 4, name: '音乐.mp3', size: 5120000, type: 'audio', path: '/music/song.mp3' },
        { id: 5, name: '压缩包.zip', size: 10485760, type: 'archive', path: '/archives/data.zip' },
        { id: 6, name: '代码.js', size: 10240, type: 'code', path: '/code/script.js' },
        { id: 7, name: '文件夹', size: 0, type: 'folder', path: '/folders/documents', isDirectory: true },
        { id: 8, name: 'PDF文件.pdf', size: 3072000, type: 'pdf', path: '/documents/report.pdf' },
      ],
      // 选中的项目
      selectedItems: [],
      // 当前网格尺寸
      currentGridSize: 'medium',
      // 是否启用多选
      enableMultiSelect: true,
      // 是否启用框选
      enableBoxSelection: true
    }
  },
  methods: {
    // 获取项目唯一键
    getItemKey(item) {
      return item.id
    },

    // 获取项目名称
    getItemName(item) {
      return item.name
    },

    // 获取项目大小
    getItemSize(item) {
      if (item.isDirectory) return ''
      return this.formatFileSize(item.size)
    },

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

    // 获取图标尺寸
    getIconSize() {
      const sizes = {
        small: 24,
        medium: 32,
        large: 48
      }
      return sizes[this.currentGridSize] || sizes.medium
    },

    // 处理选择变化
    handleSelectionChanged(items) {
      this.selectedItems = items
      console.log('选择变化:', items)
    },

    // 处理项目点击
    handleItemClick(item, event) {
      console.log('项目点击:', item)
    },

    // 处理项目双击
    handleItemDoubleClick(item) {
      console.log('项目双击:', item)
    },

    // 处理右键菜单
    handleContextMenu(item, items, event) {
      console.log('右键菜单:', item, items)
    },

    // 处理项目右键菜单
    handleItemContextMenu(item, selectedItems, event) {
      console.log('项目右键菜单:', item, selectedItems)
    },

    // 处理空白区域点击
    handleEmptyClick(event) {
      console.log('空白区域点击')
    },

    // 处理菜单操作
    handleMenuAction(action, items) {
      console.log('菜单操作:', action, items)
    }
  }
}
</script>

<style scoped>
.common-grid-view-example {
  @apply flex flex-col h-full;
}

.control-panel {
  @apply flex items-center gap-4 p-4 bg-gray-50 dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700;
}

.control-group {
  @apply flex items-center gap-2;
}

.control-select {
  @apply px-2 py-1 border border-gray-300 dark:border-gray-600 rounded;
  @apply bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100;
}

.status-info {
  @apply p-4 bg-gray-50 dark:bg-gray-800 border-t border-gray-200 dark:border-gray-700;
}

.status-info p {
  @apply text-sm text-gray-600 dark:text-gray-400;
}

.custom-context-menu {
  @apply absolute bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg shadow-lg;
  @apply py-1 z-50 min-w-32;
}

.menu-item {
  @apply px-3 py-2 text-sm text-gray-700 dark:text-gray-300 cursor-pointer;
  @apply hover:bg-gray-100 dark:hover:bg-gray-700;
}
</style> 