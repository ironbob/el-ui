<template>
  <div class="example-container">
    <h2 class="text-xl font-bold mb-4">CommonListView 示例</h2>
    
    <div class="list-container">
      <CommonListView
        :items="sampleItems"
        :selected-items="selectedItems"
        :show-header="true"
        :get-item-key="(item) => item.id"
        :get-item-name="(item) => item.name"
        :get-item-size="(item) => formatFileSize(item.size)"
        :get-item-type="(item) => item.type"
        :get-item-date="(item) => formatDate(item.date)"
        :multi-select="true"
        :enable-box-selection="true"
        @selection-changed="handleSelectionChanged"
        @item-click="handleItemClick"
        @item-double-click="handleItemDoubleClick"
        @item-context-menu="handleItemContextMenu"
        @empty-click="handleEmptyClick"
        @context-menu="handleContextMenu"
      >
        <!-- 自定义头部 -->
        <template #header>
          <div class="header-content">
            <div class="header-cell name-cell">
              <span>文件名</span>
            </div>
            <div class="header-cell size-cell">
              <span>大小</span>
            </div>
            <div class="header-cell type-cell">
              <span>类型</span>
            </div>
            <div class="header-cell date-cell">
              <span>修改时间</span>
            </div>
          </div>
        </template>

        <!-- 自定义列表项 -->
        <template #item="{ item, index, selected }">
          <div class="item-content">
            <div class="item-cell name-cell">
              <div class="item-icon">
                <div class="w-5 h-5 rounded" :class="getIconClass(item.type)"></div>
              </div>
              <span class="item-name">{{ item.name }}</span>
            </div>
            <div class="item-cell size-cell">
              {{ formatFileSize(item.size) }}
            </div>
            <div class="item-cell type-cell">
              {{ item.type }}
            </div>
            <div class="item-cell date-cell">
              {{ formatDate(item.date) }}
            </div>
          </div>
        </template>
      </CommonListView>
    </div>

    <div class="info-panel mt-4 p-4 bg-gray-100 dark:bg-gray-800 rounded">
      <h3 class="font-semibold mb-2">功能说明：</h3>
      <ul class="text-sm space-y-1">
        <li>• 点击列表项进行选择</li>
        <li>• Ctrl/Cmd + 点击：切换选择状态</li>
        <li>• Shift + 点击：范围选择</li>
        <li>• 在空白区域拖动鼠标进行框选</li>
        <li>• 右键点击显示上下文菜单</li>
      </ul>
      
      <div class="mt-4">
        <p class="text-sm"><strong>选中的项目：</strong></p>
        <p class="text-xs text-gray-600 dark:text-gray-400">
          {{ selectedItems.map(item => item.name).join(', ') || '无' }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import CommonListView from './CommonListView.vue'

export default {
  name: 'CommonListViewExample',
  components: {
    CommonListView
  },
  data() {
    return {
      selectedItems: [],
      sampleItems: [
        {
          id: 1,
          name: '文档.docx',
          size: 1024000,
          type: '文档',
          date: new Date('2024-01-15')
        },
        {
          id: 2,
          name: '图片.jpg',
          size: 2048000,
          type: '图片',
          date: new Date('2024-01-14')
        },
        {
          id: 3,
          name: '视频.mp4',
          size: 52428800,
          type: '视频',
          date: new Date('2024-01-13')
        },
        {
          id: 4,
          name: '音乐.mp3',
          size: 5120000,
          type: '音频',
          date: new Date('2024-01-12')
        },
        {
          id: 5,
          name: '压缩包.zip',
          size: 10485760,
          type: '压缩包',
          date: new Date('2024-01-11')
        }
      ]
    }
  },
  methods: {
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

    formatDate(date) {
      if (!date) return '-'
      return date.toLocaleDateString('zh-CN')
    },

    getIconClass(type) {
      const iconMap = {
        '文档': 'bg-blue-500',
        '图片': 'bg-green-500',
        '视频': 'bg-purple-500',
        '音频': 'bg-yellow-500',
        '压缩包': 'bg-red-500'
      }
      return iconMap[type] || 'bg-gray-500'
    },

    handleSelectionChanged(items) {
      this.selectedItems = items
      console.log('选择变化:', items)
    },

    handleItemClick(item, event) {
      console.log('点击项目:', item)
    },

    handleItemDoubleClick(item) {
      console.log('双击项目:', item)
    },

    handleItemContextMenu(item, selectedItems, event) {
      console.log('右键项目:', item, selectedItems)
    },

    handleEmptyClick(event) {
      console.log('点击空白区域')
    },

    handleContextMenu(item, items, event) {
      console.log('右键空白区域')
    }
  }
}
</script>

<style scoped>
.example-container {
  @apply p-6 max-w-4xl mx-auto;
}

.list-container {
  @apply h-96 border border-gray-200 dark:border-gray-700 rounded-lg overflow-hidden;
}

.header-content {
  @apply flex bg-gray-50 dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700;
}

.header-cell {
  @apply px-3 py-2 text-xs font-medium text-gray-600 dark:text-gray-400;
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

.item-content {
  @apply flex border-b border-gray-100 dark:border-gray-700;
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
</style> 