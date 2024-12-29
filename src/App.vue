<template>
  <div class="app">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <div class="main-toolbar">
      <button class="toolbar-btn" @click="saveDesign"><i class="fas fa-save"></i> 保存</button>
      <button class="toolbar-btn" @click="openDesign"><i class="fas fa-folder-open"></i> 打开</button>
      <button class="toolbar-btn" :class="{ active: currentTool === 'pointer' }" @click="setTool('pointer')">
        <i class="fas fa-mouse-pointer"></i> 指针
      </button>
      <button class="toolbar-btn" :class="{ active: currentTool === 'boxSelect' }" @click="setTool('boxSelect')">
        <i class="fas fa-vector-square"></i> 框选
      </button>
      <button class="toolbar-btn" @click="showSourceCode"><i class="fas fa-code"></i> 源代码</button>
    </div>
    <div class="design-container">
      <div class="toolbar">
        <div class="tool-header">
          <button class="toggle-button" :class="{ active: enableSnap }" @click="toggleSnap">
            {{ enableSnap ? '关闭对齐' : '开启对齐' }}
          </button>
        </div>
        <div class="tool-items">
          <div class="tool-category">
            <div class="tool-category-title">基本元素</div>
            <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'text')">
              <i class="fas fa-font"></i> 文本
            </div>
            <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'button')">
              <i class="fas fa-square"></i> 按钮
            </div>
            <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'input')">
              <i class="fas fa-keyboard"></i> 输入框
            </div>
            <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'div')">
              <i class="fas fa-border-all"></i> 容器/分组
            </div>
          </div>
        </div>
      </div>
      <div class="design-panel" @dragover.prevent @drop="onDrop">
        <!-- 对齐辅助线 -->
        <div v-if="enableSnap && snapLines.vertical" 
             class="snap-line vertical"
             :style="{ left: snapLines.vertical + 'px' }">
        </div>
        <div v-if="enableSnap && snapLines.horizontal" 
             class="snap-line horizontal"
             :style="{ top: snapLines.horizontal + 'px' }">
        </div>
        <template v-for="(item, index) in elements" :key="index">
          <div v-if="item.type === 'text'" 
               :style="item.style"
               class="design-element"
               draggable="true"
               @dragstart="onElementDragStart($event, index)"
               @drag="onElementDrag($event, index)"
               @dragend="onElementDragEnd($event, index)">
            示例文本
          </div>
          <div v-else-if="item.type === 'button'" 
               :style="item.style"
               class="design-element"
               draggable="true"
               @dragstart="onElementDragStart($event, index)"
               @drag="onElementDrag($event, index)"
               @dragend="onElementDragEnd($event, index)">
            <button style="width: 100%; height: 100%;">按钮</button>
          </div>
          <div v-else-if="item.type === 'input'" 
               :style="item.style"
               class="design-element"
               draggable="true"
               @dragstart="onElementDragStart($event, index)"
               @drag="onElementDrag($event, index)"
               @dragend="onElementDragEnd($event, index)">
            <input type="text" placeholder="请输入..." style="width: 100%; height: 100%;" />
          </div>
          <div v-else-if="item.type === 'div'" 
               :style="item.style"
               class="design-element"
               :draggable="!resizeState.isResizing"
               @dragstart="onElementDragStart($event, index)"
               @drag="onElementDrag($event, index)"
               @dragend="onElementDragEnd($event, index)">
            <div class="lock-icon" @click.stop="toggleLock(index)">
              {{ item.locked ? '🔒' : '🔓' }}
            </div>
            <!-- 缩放控制点 -->
            <div class="resize-handle top-left" @mousedown.prevent.stop="startResize($event, index, 'top-left')"></div>
            <div class="resize-handle top" @mousedown.prevent.stop="startResize($event, index, 'top')"></div>
            <div class="resize-handle top-right" @mousedown.prevent.stop="startResize($event, index, 'top-right')"></div>
            <div class="resize-handle right" @mousedown.prevent.stop="startResize($event, index, 'right')"></div>
            <div class="resize-handle bottom-right" @mousedown.prevent.stop="startResize($event, index, 'bottom-right')"></div>
            <div class="resize-handle bottom" @mousedown.prevent.stop="startResize($event, index, 'bottom')"></div>
            <div class="resize-handle bottom-left" @mousedown.prevent.stop="startResize($event, index, 'bottom-left')"></div>
            <div class="resize-handle left" @mousedown.prevent.stop="startResize($event, index, 'left')"></div>
          </div>
        </template>
      </div>
    </div>
    <div v-if="showSourceDialog" class="source-dialog">
      <pre>{{ sourceCode }}</pre>
      <button @click="closeSourceDialog">关闭</button>
      <button @click="copySourceCode">复制代码</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      elements: [],
      dragState: {
        isDragging: false,
        startX: 0,
        startY: 0,
        elementIndex: -1,
        originalX: 0,
        originalY: 0,
        childrenOriginalPositions: null // 用于存储子元素的初始位置
      },
      resizeState: {
        isResizing: false,
        elementIndex: -1,
        handle: null,
        startX: 0,
        startY: 0,
        originalWidth: 0,
        originalHeight: 0,
        originalLeft: 0,
        originalTop: 0
      },
      currentTool: 'pointer',
      showSourceDialog: false,
      sourceCode: '',
      enableSnap: false,
      snapLines: {
        vertical: null,
        horizontal: null
      }
    }
  },
  mounted() {
    // 添加全局鼠标事件监听
    window.addEventListener('mousemove', this.handleResize)
    window.addEventListener('mouseup', this.stopResize)
  },
  beforeDestroy() {
    // 移除全局事件监听
    window.removeEventListener('mousemove', this.handleResize)
    window.removeEventListener('mouseup', this.stopResize)
  },
  methods: {
    onDragStart(event, type) {
      event.dataTransfer.setData('type', type)
    },
    onDrop(event) {
      const type = event.dataTransfer.getData('type')
      if (!type) return // 如果没有type，说明是元素的拖动，不需要创建新元素
      
      const rect = event.target.getBoundingClientRect()
      const x = event.clientX - rect.left
      const y = event.clientY - rect.top

      const element = {
        type,
        style: {
          position: 'absolute',
          left: x + 'px',
          top: y + 'px',
          cursor: 'move'
        }
      }

      // 为 DIV 添加特殊样式
      if (type === 'div') {
        Object.assign(element.style, {
          width: '200px',
          height: '150px',
          border: '2px dashed #4CAF50',
          backgroundColor: 'rgba(76, 175, 80, 0.1)',
          zIndex: 0  // 确保 div 在最底层
        })
        element.locked = false
        this.elements.unshift(element)
      } else {
        // 其他元素添加更高的 z-index
        element.style.zIndex = 1
        this.elements.push(element)
      }
    },
    onElementDragStart(event, index) {
      // 如果是从缩放控制点开始的，不触发拖拽
      if (event.target.classList.contains('resize-handle')) {
        event.preventDefault();
        return;
      }

      const rect = event.target.getBoundingClientRect()
      const style = this.elements[index].style
      
      // 清除type数据，这样可以区分是新建还是移动
      event.dataTransfer.setData('text', '') 
      
      this.dragState = {
        isDragging: true,
        startX: event.clientX,
        startY: event.clientY,
        elementIndex: index,
        originalX: parseInt(style.left),
        originalY: parseInt(style.top),
        childrenOriginalPositions: null // 用于存储子元素的初始位置
      }

      // 如果是锁定状态的 div，记录内部元素的初始位置
      const element = this.elements[index]
      if (element.type === 'div' && element.locked) {
        this.dragState.childrenOriginalPositions = this.elements.map((el, idx) => {
          if (idx !== index && this.isElementInside(el, element)) {
            return {
              index: idx,
              left: parseInt(el.style.left),
              top: parseInt(el.style.top)
            }
          }
          return null
        }).filter(pos => pos !== null)
      }
    },
    onElementDrag(event, index) {
      if (!this.dragState.isDragging || this.dragState.elementIndex !== index) return

      const dx = event.clientX - this.dragState.startX
      const dy = event.clientY - this.dragState.startY
      
      const element = this.elements[index]
      
      let newLeft = this.dragState.originalX + dx
      let newTop = this.dragState.originalY + dy

      // 如果启用了自动对齐
      if (this.enableSnap) {
        const snapThreshold = 5; // 吸附阈值
        const currentRect = {
          left: newLeft,
          right: newLeft + parseInt(element.style.width || 100),
          top: newTop,
          bottom: newTop + parseInt(element.style.height || 30),
          center: newLeft + parseInt(element.style.width || 100) / 2,
          middle: newTop + parseInt(element.style.height || 30) / 2
        }

        // 重置辅助线
        this.snapLines.vertical = null
        this.snapLines.horizontal = null

        // 遍历其他元素进行对齐
        this.elements.forEach((other, otherIndex) => {
          if (otherIndex === index) return

          const otherRect = {
            left: parseInt(other.style.left),
            right: parseInt(other.style.left) + parseInt(other.style.width || 100),
            top: parseInt(other.style.top),
            bottom: parseInt(other.style.top) + parseInt(other.style.height || 30),
            center: parseInt(other.style.left) + parseInt(other.style.width || 100) / 2,
            middle: parseInt(other.style.top) + parseInt(other.style.height || 30) / 2
          }

          // 左对齐
          if (Math.abs(currentRect.left - otherRect.left) < snapThreshold) {
            newLeft = otherRect.left
            this.snapLines.vertical = otherRect.left
          }
          // 右对齐
          else if (Math.abs(currentRect.right - otherRect.right) < snapThreshold) {
            newLeft = otherRect.right - parseInt(element.style.width || 100)
            this.snapLines.vertical = otherRect.right
          }
          // 中心对齐
          else if (Math.abs(currentRect.center - otherRect.center) < snapThreshold) {
            newLeft = otherRect.center - parseInt(element.style.width || 100) / 2
            this.snapLines.vertical = otherRect.center
          }

          // 顶部对齐
          if (Math.abs(currentRect.top - otherRect.top) < snapThreshold) {
            newTop = otherRect.top
            this.snapLines.horizontal = otherRect.top
          }
          // 底部对齐
          else if (Math.abs(currentRect.bottom - otherRect.bottom) < snapThreshold) {
            newTop = otherRect.bottom - parseInt(element.style.height || 30)
            this.snapLines.horizontal = otherRect.bottom
          }
          // 中间对齐
          else if (Math.abs(currentRect.middle - otherRect.middle) < snapThreshold) {
            newTop = otherRect.middle - parseInt(element.style.height || 30) / 2
            this.snapLines.horizontal = otherRect.middle
          }
        })
      }

      // 更新元素位置
      element.style.left = newLeft + 'px'
      element.style.top = newTop + 'px'

      // 如果是锁定状态的 div，同时移动内部元素
      if (element.type === 'div' && element.locked && this.dragState.childrenOriginalPositions) {
        const adjustedDx = newLeft - this.dragState.originalX
        const adjustedDy = newTop - this.dragState.originalY
        this.dragState.childrenOriginalPositions.forEach(pos => {
          const childElement = this.elements[pos.index]
          childElement.style.left = (pos.left + adjustedDx) + 'px'
          childElement.style.top = (pos.top + adjustedDy) + 'px'
        })
      }
    },
    onElementDragEnd(event, index) {
      this.snapLines.vertical = null
      this.snapLines.horizontal = null
      this.dragState.isDragging = false
    },
    isElementInside(element1, element2) {
      const rect1 = {
        left: parseInt(element1.style.left),
        top: parseInt(element1.style.top),
        right: parseInt(element1.style.left) + (element1.type === 'div' ? parseInt(element1.style.width) : 100),
        bottom: parseInt(element1.style.top) + (element1.type === 'div' ? parseInt(element1.style.height) : 30)
      }
      
      const rect2 = {
        left: parseInt(element2.style.left),
        top: parseInt(element2.style.top),
        right: parseInt(element2.style.left) + parseInt(element2.style.width),
        bottom: parseInt(element2.style.top) + parseInt(element2.style.height)
      }
      
      return rect1.left >= rect2.left && 
             rect1.right <= rect2.right && 
             rect1.top >= rect2.top && 
             rect1.bottom <= rect2.bottom
    },
    saveDesign() {
      // 保存设计的逻辑
    },
    openDesign() {
      // 打开设计的逻辑
    },
    setTool(tool) {
      this.currentTool = tool
    },
    showSourceCode() {
      let code = this.elements.map(element => {
        const { type, style } = element;
        let elementCode = '';
        
        switch(type) {
          case 'text':
            elementCode = `<div style="${this.styleToString(style)}">示例文本</div>`;
            break;
          case 'button':
            elementCode = `<button style="${this.styleToString(style)}">按钮</button>`;
            break;
          case 'input':
            elementCode = `<input style="${this.styleToString(style)}" placeholder="请输入..."/>`;
            break;
          case 'div':
            elementCode = `<div style="${this.styleToString(style)}"></div>`;
            break;
        }
        
        return elementCode;
      }).join('\n');
      
      this.sourceCode = code;
      this.showSourceDialog = true;
    },
    closeSourceDialog() {
      this.showSourceDialog = false;
    },
    async copySourceCode() {
      try {
        await navigator.clipboard.writeText(this.sourceCode);
        alert('代码已复制到剪贴板');
      } catch (err) {
        alert('复制失败，请手动复制');
      }
    },
    styleToString(style) {
      return Object.entries(style)
        .map(([key, value]) => `${key}: ${value}`)
        .join('; ');
    },
    toggleLock(index) {
      this.elements[index].locked = !this.elements[index].locked
    },
    startResize(event, index, handle) {
      event.preventDefault(); // 阻止默认事件
      event.stopPropagation(); // 阻止事件冒泡
      
      const element = this.elements[index]
      this.resizeState = {
        isResizing: true,
        elementIndex: index,
        handle,
        startX: event.clientX,
        startY: event.clientY,
        originalWidth: parseInt(element.style.width),
        originalHeight: parseInt(element.style.height),
        originalLeft: parseInt(element.style.left),
        originalTop: parseInt(element.style.top)
      }
    },

    handleResize(event) {
      if (!this.resizeState.isResizing) return

      const dx = event.clientX - this.resizeState.startX
      const dy = event.clientY - this.resizeState.startY
      const element = this.elements[this.resizeState.elementIndex]
      const handle = this.resizeState.handle
      
      let newWidth = this.resizeState.originalWidth
      let newHeight = this.resizeState.originalHeight
      let newLeft = this.resizeState.originalLeft
      let newTop = this.resizeState.originalTop

      // 根据不同的控制点处理缩放
      if (handle.includes('right')) {
        newWidth = Math.max(50, this.resizeState.originalWidth + dx)
      }
      if (handle.includes('left')) {
        const adjustedWidth = Math.max(50, this.resizeState.originalWidth - dx)
        newLeft = this.resizeState.originalLeft + (this.resizeState.originalWidth - adjustedWidth)
        newWidth = adjustedWidth
      }
      if (handle.includes('bottom')) {
        newHeight = Math.max(50, this.resizeState.originalHeight + dy)
      }
      if (handle.includes('top')) {
        const adjustedHeight = Math.max(50, this.resizeState.originalHeight - dy)
        newTop = this.resizeState.originalTop + (this.resizeState.originalHeight - adjustedHeight)
        newHeight = adjustedHeight
      }

      // 更新元素样式
      element.style.width = `${newWidth}px`
      element.style.height = `${newHeight}px`
      element.style.left = `${newLeft}px`
      element.style.top = `${newTop}px`
    },

    stopResize() {
      this.resizeState.isResizing = false
    },
    toggleSnap() {
      this.enableSnap = !this.enableSnap
    }
  }
}
</script>

<style>
.app {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-toolbar {
  padding: 8px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #ddd;
  display: flex;
  gap: 8px;
}

.toolbar-btn {
  padding: 6px 12px;
  border: 1px solid #ddd;
  background-color: white;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 14px;
}

.toolbar-btn:hover {
  background-color: #f0f0f0;
}

.toolbar-btn.active {
  background-color: #e6f7ff;
  border-color: #1890ff;
  color: #1890ff;
}

.toolbar-btn i {
  font-size: 14px;
}

.toolbar {
  width: 240px;
  padding: 0;
  background-color: #f5f5f5;
  border-right: 1px solid #ddd;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.tool-header {
  padding: 12px;
  border-bottom: 1px solid #ddd;
}

.tool-items {
  padding: 12px;
}

.tool-category {
  margin-bottom: 16px;
}

.tool-category-title {
  font-size: 13px;
  color: #666;
  margin-bottom: 8px;
  padding: 0 4px;
}

.tool-item {
  display: flex;
  align-items: center;
  margin-bottom: 4px;
  padding: 8px 12px;
  background-color: white;
  border: 1px solid #e8e8e8;
  border-radius: 4px;
  cursor: move;
  transition: all 0.2s;
  font-size: 14px;
}

.tool-item:hover {
  border-color: #1890ff;
  color: #1890ff;
  background-color: #e6f7ff;
}

.tool-item i {
  margin-right: 8px;
  font-size: 16px;
  color: #666;
}

.tool-item:hover i {
  color: #1890ff;
}

.toggle-button {
  width: 100%;
  padding: 8px 12px;
  background-color: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  outline: none;
  transition: all 0.2s;
  font-size: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.toggle-button i {
  margin-right: 6px;
  font-size: 16px;
}

.toggle-button.active {
  background-color: #1890ff;
  color: white;
  border-color: #1890ff;
}

.toggle-button:hover {
  border-color: #1890ff;
  color: #1890ff;
}

.toggle-button.active:hover {
  background-color: #40a9ff;
  color: white;
}

.design-container {
  display: flex;
  height: 100vh;
}

.design-panel {
  flex: 1;
  position: relative;
  background-color: white;
  overflow: auto;
  min-height: 100vh;
}

.design-element {
  cursor: move;
  user-select: none;
  position: relative;
}

.design-element:hover {
  outline: 1px solid #1890ff;
}

.design-element button {
  width: 100%;
  height: 100%;
  border: 1px solid #ddd;
  background-color: white;
  border-radius: 4px;
  cursor: move;
  pointer-events: none;
}

.design-element input {
  width: 100%;
  height: 100%;
  border: 1px solid #ddd;
  background-color: white;
  border-radius: 4px;
  padding: 4px 8px;
  pointer-events: none;
}

.lock-icon {
  position: absolute;
  top: 5px;
  right: 5px;
  cursor: pointer;
  font-size: 16px;
  user-select: none;
  z-index: 1;
}

.resize-handle {
  position: absolute;
  width: 8px;
  height: 8px;
  background-color: white;
  border: 1px solid #1890ff;
  z-index: 2;
}

.resize-handle.top-left {
  top: -4px;
  left: -4px;
  cursor: nw-resize;
}

.resize-handle.top {
  top: -4px;
  left: 50%;
  transform: translateX(-50%);
  cursor: n-resize;
}

.resize-handle.top-right {
  top: -4px;
  right: -4px;
  cursor: ne-resize;
}

.resize-handle.right {
  top: 50%;
  right: -4px;
  transform: translateY(-50%);
  cursor: e-resize;
}

.resize-handle.bottom-right {
  bottom: -4px;
  right: -4px;
  cursor: se-resize;
}

.resize-handle.bottom {
  bottom: -4px;
  left: 50%;
  transform: translateX(-50%);
  cursor: s-resize;
}

.resize-handle.bottom-left {
  bottom: -4px;
  left: -4px;
  cursor: sw-resize;
}

.resize-handle.left {
  top: 50%;
  left: -4px;
  transform: translateY(-50%);
  cursor: w-resize;
}

.snap-line {
  position: absolute;
  background-color: #1890ff;
  pointer-events: none;
  z-index: 1000;
}

.snap-line.vertical {
  width: 1px;
  height: 100%;
  top: 0;
}

.snap-line.horizontal {
  height: 1px;
  width: 100%;
  left: 0;
}

.source-dialog {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.source-dialog pre {
  background-color: white;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 80%;
  height: 80%;
  overflow: auto;
}

.source-dialog button {
  margin-top: 20px;
  padding: 10px 20px;
  border: 1px solid #ddd;
  background-color: white;
  border-radius: 4px;
  cursor: pointer;
}
</style>
