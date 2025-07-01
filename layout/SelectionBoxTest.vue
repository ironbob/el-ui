<template>
  <div class="selection-box-test">
    <h2>框选控件测试</h2>
    
    <!-- 基本框选测试 -->
    <div class="test-section">
      <h3>基本框选测试</h3>
      <SelectionBox
        :enabled="true"
        :get-selectable-elements="getSelectableElements"
        :get-element-key="getElementKey"
        @selection-changed="handleSelectionChanged"
        @empty-click="handleEmptyClick"
      >
        <div class="test-container">
          <div
            v-for="item in testItems"
            :key="item.id"
            class="test-item"
            :data-selectable="true"
            :data-key="item.id"
            :class="{ selected: selectedItems.includes(item.id) }"
            @click="handleItemClick(item, $event)"
          >
            <div class="item-icon">{{ item.icon }}</div>
            <div class="item-name">{{ item.name }}</div>
          </div>
        </div>
      </SelectionBox>
      
      <div class="selection-info">
        选中项目: {{ selectedItems.join(', ') || '无' }}
      </div>
    </div>

    <!-- CommonListView 测试 -->
    <div class="test-section">
      <h3>CommonListView 框选测试</h3>
      <CommonListView
        :items="listItems"
        :selected-items="selectedListItems"
        :enable-box-selection="true"
        :multi-select="true"
        @selection-changed="handleListSelectionChanged"
        @item-click="handleListItemClick"
        @empty-click="handleListEmptyClick"
        @context-menu="handleListContextMenu"
        @item-context-menu="handleListItemContextMenu"
      >
        <template #item="{ item, selected }">
          <div class="item-content">
            <div class="item-cell name-cell">
              <div class="item-icon">
                <span>{{ item.icon }}</span>
              </div>
              <span class="item-name">{{ item.name }}</span>
            </div>
            <div class="item-cell size-cell">
              {{ item.size }}
            </div>
            <div class="item-cell type-cell">
              {{ item.type }}
            </div>
            <div class="item-cell date-cell">
              {{ item.date }}
            </div>
          </div>
        </template>
      </CommonListView>
      
      <div class="selection-info">
        选中项目: {{ selectedListItems.map(item => item.name).join(', ') || '无' }}
      </div>
    </div>

    <!-- 右键菜单测试 -->
    <div class="test-section">
      <h3>右键菜单测试</h3>
      <div class="context-menu-info">
        <p>在列表项上右键点击测试上下文菜单</p>
        <p>在空白区域右键点击测试空白区域菜单</p>
      </div>
    </div>
  </div>
</template>

<script>
import SelectionBox from './SelectionBox.vue'
import CommonListView from './CommonListView.vue'

export default {
  name: 'SelectionBoxTest',
  components: {
    SelectionBox,
    CommonListView
  },

  data() {
    return {
      // 基本框选测试数据
      testItems: [
        { id: '1', name: '文件1', icon: '📄' },
        { id: '2', name: '文件2', icon: '📄' },
        { id: '3', name: '文件夹1', icon: '📁' },
        { id: '4', name: '文件3', icon: '📄' },
        { id: '5', name: '文件夹2', icon: '📁' },
        { id: '6', name: '文件4', icon: '📄' },
        { id: '7', name: '文件5', icon: '📄' },
        { id: '8', name: '文件夹3', icon: '📁' }
      ],
      selectedItems: [],

      // CommonListView 测试数据
      listItems: [
        { id: '1', name: '文档1.txt', size: '1.2 KB', type: '文本文件', date: '2024-01-15', icon: '📄' },
        { id: '2', name: '图片1.jpg', size: '2.5 MB', type: '图片文件', date: '2024-01-14', icon: '🖼️' },
        { id: '3', name: '项目文件夹', size: '-', type: '文件夹', date: '2024-01-13', icon: '📁' },
        { id: '4', name: '视频1.mp4', size: '15.7 MB', type: '视频文件', date: '2024-01-12', icon: '🎥' },
        { id: '5', name: '音乐1.mp3', size: '3.8 MB', type: '音频文件', date: '2024-01-11', icon: '🎵' },
        { id: '6', name: '压缩包.zip', size: '8.2 MB', type: '压缩文件', date: '2024-01-10', icon: '📦' },
        { id: '7', name: '代码.js', size: '2.1 KB', type: '代码文件', date: '2024-01-09', icon: '💻' },
        { id: '8', name: '数据.xlsx', size: '1.8 MB', type: '表格文件', date: '2024-01-08', icon: '📊' }
      ],
      selectedListItems: []
    }
  },

  methods: {
    // 基本框选测试方法
    getSelectableElements() {
      return this.$el.querySelectorAll('[data-selectable]')
    },

    getElementKey(element) {
      return element.dataset.key
    },

    handleSelectionChanged(selectedElements) {
      this.selectedItems = selectedElements.map(({ key }) => key)
      console.log('基本框选变化:', this.selectedItems)
    },

    handleEmptyClick(event) {
      console.log('基本框选空白区域点击')
      this.selectedItems = []
    },

    handleItemClick(item, event) {
      console.log('基本框选项目点击:', item.name)
    },

    // CommonListView 测试方法
    handleListSelectionChanged(selectedItems) {
      this.selectedListItems = selectedItems
      console.log('列表选择变化:', selectedItems.map(item => item.name))
    },

    handleListItemClick(item, event) {
      console.log('列表项目点击:', item.name)
    },

    handleListEmptyClick(event) {
      console.log('列表空白区域点击')
      this.selectedListItems = []
    },

    handleListContextMenu(items, selectedItems, event) {
      console.log('列表空白区域右键菜单:', { items, selectedItems })
      this.$message.info('空白区域右键菜单')
    },

    handleListItemContextMenu(item, selectedItems, event) {
      console.log('列表项目右键菜单:', { item: item.name, selectedItems: selectedItems.map(i => i.name) })
      this.$message.info(`项目右键菜单: ${item.name}`)
    }
  }
}
</script>

<style scoped>
.selection-box-test {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.test-section {
  margin-bottom: 40px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 20px;
}

.test-section h3 {
  margin-top: 0;
  margin-bottom: 16px;
  color: #333;
}

.test-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 16px;
  padding: 16px;
  background: #f8f9fa;
  border-radius: 8px;
  min-height: 200px;
}

.test-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px 8px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  background: white;
  border: 2px solid transparent;
  min-height: 80px;
}

.test-item:hover {
  background: #e3f2fd;
  border-color: #2196f3;
}

.test-item.selected {
  background: #2196f3 !important;
  color: white;
  border-color: #1976d2;
}

.item-icon {
  font-size: 24px;
  margin-bottom: 8px;
}

.item-name {
  font-size: 12px;
  text-align: center;
  word-wrap: break-word;
  line-height: 1.2;
}

.selection-info {
  margin-top: 16px;
  padding: 12px;
  background: #e8f5e8;
  border-radius: 4px;
  font-size: 14px;
  color: #2e7d32;
}

.context-menu-info {
  padding: 16px;
  background: #fff3cd;
  border-radius: 4px;
  border: 1px solid #ffeaa7;
}

.context-menu-info p {
  margin: 8px 0;
  color: #856404;
}

/* CommonListView 样式调整 */
:deep(.common-list-view) {
  height: 300px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

:deep(.list-body) {
  height: 250px;
}

:deep(.list-item) {
  padding: 8px 12px;
}

:deep(.item-content) {
  display: flex;
  align-items: center;
  width: 100%;
}

:deep(.item-cell) {
  padding: 4px 8px;
  font-size: 12px;
}

:deep(.name-cell) {
  flex: 1;
  min-width: 200px;
}

:deep(.size-cell) {
  width: 80px;
}

:deep(.type-cell) {
  width: 100px;
}

:deep(.date-cell) {
  width: 120px;
}
</style> 