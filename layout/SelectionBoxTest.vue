<template>
  <div class="selection-box-test">
    <h2>SelectionBox 测试</h2>
    
    <div class="test-info">
      <p><strong>测试说明：</strong></p>
      <ul>
        <li>✅ 框选过程中不会发送选择变化事件，节省资源</li>
        <li>✅ 只在框选结束时发送最终结果</li>
        <li>✅ 支持清空选择事件</li>
        <li>✅ 支持空白区域点击事件</li>
      </ul>
    </div>

    <div class="test-controls">
      <button @click="clearLogs" class="btn">清空日志</button>
      <button @click="toggleEnabled" class="btn">
        {{ enabled ? '禁用' : '启用' }} 框选
      </button>
    </div>

    <div class="test-container">
      <!-- 使用 SelectionBox 组件 -->
      <SelectionBox
        :enabled="enabled"
        :get-selectable-elements="getSelectableElements"
        :get-element-key="getElementKey"
        :multi-select="true"
        @selection-finished="handleSelectionFinished"
        @selection-cleared="handleSelectionCleared"
        @empty-click="handleEmptyClick"
      >
        <div class="test-grid">
          <div
            v-for="item in testItems"
            :key="item.id"
            class="test-item"
            :data-selectable="true"
            :data-item-key="item.id"
          >
            <div class="item-icon">{{ item.icon }}</div>
            <div class="item-name">{{ item.name }}</div>
          </div>
        </div>
      </SelectionBox>
    </div>

    <div class="test-logs">
      <h3>事件日志：</h3>
      <div class="log-container">
        <div
          v-for="(log, index) in logs"
          :key="index"
          class="log-item"
          :class="log.type"
        >
          <span class="log-time">{{ log.time }}</span>
          <span class="log-event">{{ log.event }}</span>
          <span class="log-data">{{ log.data }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SelectionBox from './SelectionBox.vue'

export default {
  name: 'SelectionBoxTest',
  components: {
    SelectionBox
  },
  data() {
    return {
      enabled: true,
      logs: [],
      testItems: [
        { id: '1', name: '文件1.txt', icon: '📄' },
        { id: '2', name: '文件2.jpg', icon: '🖼️' },
        { id: '3', name: '文件夹1', icon: '📁' },
        { id: '4', name: '文件3.pdf', icon: '📕' },
        { id: '5', name: '文件4.mp3', icon: '🎵' },
        { id: '6', name: '文件5.mp4', icon: '🎬' },
        { id: '7', name: '文件夹2', icon: '📁' },
        { id: '8', name: '文件6.zip', icon: '📦' },
        { id: '9', name: '文件7.doc', icon: '📘' },
        { id: '10', name: '文件8.xls', icon: '📊' },
        { id: '11', name: '文件夹3', icon: '📁' },
        { id: '12', name: '文件9.html', icon: '🌐' }
      ]
    }
  },
  methods: {
    // 获取可选择元素
    getSelectableElements() {
      return this.$el.querySelectorAll('[data-selectable]')
    },

    // 获取元素唯一标识
    getElementKey(element) {
      return element.dataset.itemKey
    },

    // 处理框选结束事件
    handleSelectionFinished(selectedElements) {
      const selectedItems = selectedElements.map(({ key }) => {
        return this.testItems.find(item => item.id === key)
      }).filter(Boolean)

      this.addLog('selection-finished', `选择了 ${selectedItems.length} 个项目`, 
        selectedItems.map(item => item.name).join(', '))
      
      console.log('SelectionBoxTest: 框选结束，最终选择:', selectedItems.map(item => item.name))
    },

    // 处理选择清空事件
    handleSelectionCleared() {
      this.addLog('selection-cleared', '清空选择', '')
      console.log('SelectionBoxTest: 选择已清空')
    },

    // 处理空白区域点击
    handleEmptyClick(event) {
      this.addLog('empty-click', '空白区域点击', `坐标: (${event.clientX}, ${event.clientY})`)
      console.log('SelectionBoxTest: 空白区域点击')
    },

    // 添加日志
    addLog(type, event, data) {
      const now = new Date()
      const time = now.toLocaleTimeString()
      
      this.logs.unshift({
        type,
        time,
        event,
        data
      })

      // 限制日志数量
      if (this.logs.length > 50) {
        this.logs = this.logs.slice(0, 50)
      }
    },

    // 清空日志
    clearLogs() {
      this.logs = []
    },

    // 切换框选启用状态
    toggleEnabled() {
      this.enabled = !this.enabled
      this.addLog('toggle', `框选${this.enabled ? '启用' : '禁用'}`, '')
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

.test-info {
  background: #f8f9fa;
  border: 1px solid #e9ecef;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 20px;
}

.test-info ul {
  margin: 10px 0 0 20px;
}

.test-controls {
  margin-bottom: 20px;
  display: flex;
  gap: 10px;
}

.btn {
  padding: 8px 16px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.btn:hover {
  background: #0056b3;
}

.test-container {
  border: 2px solid #dee2e6;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
  background: white;
}

.test-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 15px;
  padding: 20px;
}

.test-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 15px;
  border: 1px solid #dee2e6;
  border-radius: 6px;
  background: #f8f9fa;
  cursor: pointer;
  transition: all 0.2s;
  user-select: none;
}

.test-item:hover {
  background: #e9ecef;
  border-color: #adb5bd;
}

.item-icon {
  font-size: 24px;
  margin-bottom: 8px;
}

.item-name {
  font-size: 12px;
  text-align: center;
  word-break: break-word;
}

.test-logs {
  border: 1px solid #dee2e6;
  border-radius: 8px;
  background: white;
}

.test-logs h3 {
  margin: 0;
  padding: 15px;
  background: #f8f9fa;
  border-bottom: 1px solid #dee2e6;
  border-radius: 8px 8px 0 0;
}

.log-container {
  max-height: 300px;
  overflow-y: auto;
  padding: 10px;
}

.log-item {
  display: flex;
  gap: 10px;
  padding: 8px;
  margin-bottom: 5px;
  border-radius: 4px;
  font-family: monospace;
  font-size: 12px;
}

.log-item.selection-finished {
  background: #d4edda;
  border: 1px solid #c3e6cb;
  color: #155724;
}

.log-item.selection-cleared {
  background: #f8d7da;
  border: 1px solid #f5c6cb;
  color: #721c24;
}

.log-item.empty-click {
  background: #d1ecf1;
  border: 1px solid #bee5eb;
  color: #0c5460;
}

.log-item.toggle {
  background: #fff3cd;
  border: 1px solid #ffeaa7;
  color: #856404;
}

.log-time {
  color: #6c757d;
  min-width: 80px;
}

.log-event {
  font-weight: bold;
  min-width: 120px;
}

.log-data {
  flex: 1;
  word-break: break-all;
}

/* 深色主题支持 */
[data-theme="dark"] .test-info {
  background: #2d3748;
  border-color: #4a5568;
  color: #e2e8f0;
}

[data-theme="dark"] .test-container {
  background: #2d3748;
  border-color: #4a5568;
}

[data-theme="dark"] .test-item {
  background: #4a5568;
  border-color: #718096;
  color: #e2e8f0;
}

[data-theme="dark"] .test-item:hover {
  background: #718096;
  border-color: #a0aec0;
}

[data-theme="dark"] .test-logs {
  background: #2d3748;
  border-color: #4a5568;
}

[data-theme="dark"] .test-logs h3 {
  background: #4a5568;
  border-color: #718096;
  color: #e2e8f0;
}

[data-theme="dark"] .log-item.selection-finished {
  background: #22543d;
  border-color: #38a169;
  color: #9ae6b4;
}

[data-theme="dark"] .log-item.selection-cleared {
  background: #742a2a;
  border-color: #e53e3e;
  color: #feb2b2;
}

[data-theme="dark"] .log-item.empty-click {
  background: #2a4365;
  border-color: #3182ce;
  color: #90cdf4;
}

[data-theme="dark"] .log-item.toggle {
  background: #744210;
  border-color: #d69e2e;
  color: #faf089;
}
</style> 