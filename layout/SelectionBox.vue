<template>
  <div 
    ref="container"
    class="selection-box-container"
    @mousedown="handleMouseDown"
    @click="handleClick"
  >
    <!-- 框选区域 -->
    <div
      v-if="selectionBox.active"
      class="selection-box"
      :style="{
        left: Math.min(selectionBox.startX, selectionBox.currentX) + 'px',
        top: Math.min(selectionBox.startY, selectionBox.currentY) + 'px',
        width: Math.abs(selectionBox.currentX - selectionBox.startX) + 'px',
        height: Math.abs(selectionBox.currentY - selectionBox.startY) + 'px'
      }"
    ></div>

    <!-- 插槽内容 -->
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: 'SelectionBox',
  props: {
    // 是否启用框选
    enabled: {
      type: Boolean,
      default: true
    },
    // 选择器函数，用于获取容器内的可选择元素
    getSelectableElements: {
      type: Function,
      default: () => []
    },
    // 获取元素唯一标识的函数
    getElementKey: {
      type: Function,
      default: (element) => element.dataset?.key || element.id || null
    },
    // 是否允许多选
    multiSelect: {
      type: Boolean,
      default: true
    },
    // 当前选中的元素
    selectedElements: {
      type: Array,
      default: () => []
    }
  },

  data() {
    return {
      // 框选状态
      selectionBox: {
        active: false,
        startX: 0,
        startY: 0,
        currentX: 0,
        currentY: 0
      }
    }
  },

  methods: {
    // 处理鼠标按下事件
    handleMouseDown(event) {
      if (!this.enabled || event.button !== 0) return

      // 检查是否点击在可选择元素上
      // const selectableElement = event.target.closest('[data-selectable]')
      // if (selectableElement) {
      //   // 如果点击在可选择元素上，不开始框选
      //   return
      // }

      // 检查是否点击在容器内
      if (!this.$refs.container.contains(event.target)) {
        return
      }

      this.startSelection(event)
    },

    // 处理点击事件
    handleClick(event) {
      // 检查是否点击在可选择元素上
      const selectableElement = event.target.closest('[data-selectable]')
      if (selectableElement) {
        // 如果点击在可选择元素上，不处理空白区域点击
        return
      }

      // 检查是否点击在容器内
      if (this.$refs.container.contains(event.target)) {
        this.$emit('empty-click', event)
        this.$emit('selection-changed', [])
      }
    },

    // 开始框选
    startSelection(event) {
      const containerElement = this.$refs.container
      if (!containerElement) return

      const rect = containerElement.getBoundingClientRect()
      
      this.selectionBox.active = true
      this.selectionBox.startX = event.clientX - rect.left
      this.selectionBox.startY = event.clientY - rect.top
      this.selectionBox.currentX = event.clientX - rect.left
      this.selectionBox.currentY = event.clientY - rect.top

      // 添加全局事件监听器
      document.addEventListener('mousemove', this.handleMouseMove)
      document.addEventListener('mouseup', this.handleMouseUp)

      event.preventDefault()
    },

    // 处理鼠标移动
    handleMouseMove(event) {
      if (!this.selectionBox.active) return

      const containerElement = this.$refs.container
      if (!containerElement) return

      const rect = containerElement.getBoundingClientRect()
      
      this.selectionBox.currentX = event.clientX - rect.left
      this.selectionBox.currentY = event.clientY - rect.top

      // 计算选择区域内的元素
      this.updateSelection()
    },

    // 处理鼠标释放
    handleMouseUp() {
      if (!this.selectionBox.active) return

      this.selectionBox.active = false

      // 移除全局事件监听器
      document.removeEventListener('mousemove', this.handleMouseMove)
      document.removeEventListener('mouseup', this.handleMouseUp)

      // 最终更新选择
      this.updateSelection()
    },

    // 更新选择
    updateSelection() {
      const selectionRect = this.getSelectionRect()
      const selectedElements = []

      // 获取所有可选择元素
      const selectableElements = this.getSelectableElements()
      
      selectableElements.forEach(element => {
        if (this.isElementInSelection(element, selectionRect)) {
          const key = this.getElementKey(element)
          if (key) {
            selectedElements.push({ element, key })
          }
        }
      })

      this.$emit('selection-changed', selectedElements)
      // console.log('SelectionBox: Selection changed:', selectedElements)
    },

    // 获取选择矩形
    getSelectionRect() {
      const startX = Math.min(this.selectionBox.startX, this.selectionBox.currentX)
      const startY = Math.min(this.selectionBox.startY, this.selectionBox.currentY)
      const endX = Math.max(this.selectionBox.startX, this.selectionBox.currentX)
      const endY = Math.max(this.selectionBox.startY, this.selectionBox.currentY)

      return {
        left: startX,
        top: startY,
        right: endX,
        bottom: endY,
        width: endX - startX,
        height: endY - startY
      }
    },

    // 检查元素是否在选择区域内
    isElementInSelection(element, selectionRect) {
      const containerElement = this.$refs.container
      if (!containerElement) return false

      const containerRect = containerElement.getBoundingClientRect()
      const elementRect = element.getBoundingClientRect()

      // 将元素位置转换为相对于容器的坐标
      const relativeElementRect = {
        left: elementRect.left - containerRect.left,
        right: elementRect.right - containerRect.left,
        top: elementRect.top - containerRect.top,
        bottom: elementRect.bottom - containerRect.top
      }

      return !(relativeElementRect.right < selectionRect.left ||
               relativeElementRect.left > selectionRect.right ||
               relativeElementRect.bottom < selectionRect.top ||
               relativeElementRect.top > selectionRect.bottom)
    },

    // 清除选择框
    clearSelection() {
      this.selectionBox.active = false
    }
  }
}
</script>

<style scoped>
.selection-box-container {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.selection-box {
  position: absolute;
  border: 2px solid #409EFF;
  background: rgba(64, 158, 255, 0.1);
  pointer-events: none;
  z-index: 1000;
}

/* 深色主题下的选择框样式 */
[data-theme="dark"] .selection-box {
  border-color: #79BBFF;
  background: rgba(121, 187, 255, 0.1);
}
</style> 