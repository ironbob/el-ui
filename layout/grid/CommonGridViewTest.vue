<template>
  <div class="common-grid-view-test">
    <h2>CommonGridView 测试</h2>
    
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
    <div class="grid-container">
      <CommonGridView
        :items="testItems"
        :selected-items="selectedItems"
        :grid-size="currentGridSize"
        :multi-select="enableMultiSelect"
        :enable-box-selection="enableBoxSelection"
        :get-item-key="getItemKey"
        @selection-finished="handleSelectionFinished"
        @selection-cleared="handleSelectionCleared"
        @item-click="handleItemClick"
        @item-double-click="handleItemDoubleClick"
        @context-menu="handleContextMenu"
        @item-context-menu="handleItemContextMenu"
        @empty-click="handleEmptyClick"
      >
        <!-- 自定义项目内容插槽 -->
        <template #item="{ item, index, selected }">
          <div class="test-item-content">
            <div class="test-icon" :style="{ backgroundColor: getIconColor(item.type) }">
              {{ item.type.charAt(0).toUpperCase() }}
            </div>
            <div class="test-item-info">
              <div class="test-item-name" :title="item.name">{{ item.name }}</div>
              <div class="test-item-details" v-if="currentGridSize !== 'small'">
                <span class="test-item-size">{{ formatFileSize(item.size) }}</span>
              </div>
            </div>
          </div>
        </template>

        <!-- 自定义右键菜单 -->
        <template #context-menu="slotProps">
          <div
            v-if="slotProps.visible"
            class="test-context-menu"
            :style="{
              left: slotProps.position.x + 'px',
              top: slotProps.position.y + 'px'
            }"
          >
            <div class="menu-item" @click="handleMenuAction('open', slotProps.items)">
              打开
            </div>
            <div class="menu-item" @click="handleMenuAction('copy', slotProps.items)">
              复制
            </div>
            <div class="menu-item" @click="handleMenuAction('delete', slotProps.items)">
              删除
            </div>
          </div>
        </template>
      </CommonGridView>
    </div>

    <!-- 状态信息 -->
    <div class="status-info">
      <p>选中项目数量: {{ selectedItems.length }}</p>
      <p>当前网格尺寸: {{ currentGridSize }}</p>
      <p>选中项目: {{ selectedItems.map(item => item.name).join(', ') || '无' }}</p>
    </div>
  </div>
</template>

<script>
import CommonGridView from './CommonGridView.vue'

export default {
  name: 'CommonGridViewTest',
  components: {
    CommonGridView
  },
  data() {
    return {
      // 测试数据
      testItems: [
        { id: 1, name: '文档.docx', size: 1024000, type: 'document' },
        { id: 2, name: '图片.jpg', size: 2048000, type: 'image' },
        { id: 3, name: '视频.mp4', size: 52428800, type: 'video' },
        { id: 4, name: '音乐.mp3', size: 5120000, type: 'audio' },
        { id: 5, name: '压缩包.zip', size: 10485760, type: 'archive' },
        { id: 6, name: '代码.js', size: 10240, type: 'code' },
        { id: 7, name: '文件夹', size: 0, type: 'folder' },
        { id: 8, name: 'PDF文件.pdf', size: 3072000, type: 'pdf' },
        { id: 9, name: '表格.xlsx', size: 1536000, type: 'spreadsheet' },
        { id: 10, name: '演示文稿.pptx', size: 4096000, type: 'presentation' },
        { id: 11, name: '文本文件.txt', size: 512, type: 'text' },
        { id: 12, name: '数据库.db', size: 2048576, type: 'database' },
        { id: 13, name: '配置文件.ini', size: 256, type: 'config' },
        { id: 14, name: '日志文件.log', size: 1024, type: 'log' },
        { id: 15, name: '备份文件.bak', size: 8192, type: 'backup' },
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

    // 获取图标颜色
    getIconColor(type) {
      const colors = {
        document: '#4285f4',
        image: '#34a853',
        video: '#ea4335',
        audio: '#fbbc04',
        archive: '#ff6d01',
        code: '#9c27b0',
        folder: '#ff9800',
        pdf: '#f44336',
        spreadsheet: '#4caf50',
        presentation: '#2196f3',
        text: '#607d8b',
        database: '#795548',
        config: '#9e9e9e',
        log: '#ff5722',
        backup: '#673ab7'
      }
      return colors[type] || '#757575'
    },

    // 处理选择完成事件
    handleSelectionFinished(items) {
      this.selectedItems = items
      console.log('选择完成:', items)
    },

    // 处理选择清空事件
    handleSelectionCleared() {
      this.selectedItems = []
      console.log('选择清空')
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
      alert(`执行操作: ${action}, 项目数量: ${items.length}`)
    }
  }
}
</script>

<style scoped>
.common-grid-view-test {
  @apply flex flex-col h-full p-4;
}

.control-panel {
  @apply flex items-center gap-4 p-4 bg-gray-50 dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg mb-4;
}

.control-group {
  @apply flex items-center gap-2;
}

.control-select {
  @apply px-2 py-1 border border-gray-300 dark:border-gray-600 rounded;
  @apply bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100;
}

.grid-container {
  @apply flex-1 min-h-0 border border-gray-200 dark:border-gray-700 rounded-lg overflow-hidden;
}

.status-info {
  @apply p-4 bg-gray-50 dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg mt-4;
}

.status-info p {
  @apply text-sm text-gray-600 dark:text-gray-400 mb-1;
}

.test-item-content {
  @apply flex flex-col items-center w-full h-full;
  min-height: 0;
}

.test-icon {
  @apply flex items-center justify-center text-white font-bold rounded;
  width: 32px;
  height: 32px;
  flex-shrink: 0;
}

.test-item-info {
  @apply text-center w-full;
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.test-item-name {
  @apply font-normal text-gray-900 dark:text-gray-100 mb-1;
  @apply whitespace-nowrap overflow-hidden text-ellipsis;
  @apply text-center max-w-full;
  line-height: 1.2;
}

.test-item-details {
  @apply text-xs text-gray-600 dark:text-gray-400 text-center;
  flex-shrink: 0;
}

.test-item-size {
  @apply whitespace-nowrap;
}

.test-context-menu {
  @apply absolute bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg shadow-lg;
  @apply py-1 z-50 min-w-32;
}

.menu-item {
  @apply px-3 py-2 text-sm text-gray-700 dark:text-gray-300 cursor-pointer;
  @apply hover:bg-gray-100 dark:hover:bg-gray-700;
}
</style> 