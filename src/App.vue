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
               :class="{ selected: selectedElement === item }"
               @mousedown="startDrag($event, index)"
               @click="selectElement(item)">
            示例文本
          </div>
          <div v-else-if="item.type === 'button'" 
               :style="item.style"
               class="design-element"
               :class="{ selected: selectedElement === item }"
               @mousedown="startDrag($event, index)"
               @click="selectElement(item)">
            <button style="width: 100%; height: 100%;">按钮</button>
          </div>
          <div v-else-if="item.type === 'input'" 
               :style="item.style"
               class="design-element"
               :class="{ selected: selectedElement === item }"
               @mousedown="startDrag($event, index)"
               @click="selectElement(item)">
            <input type="text" placeholder="请输入..." style="width: 100%; height: 100%;" />
          </div>
          <div v-else-if="item.type === 'div'" 
               :style="item.style"
               class="design-element"
               :class="{ selected: selectedElement === item }"
               @mousedown="startDrag($event, index)"
               @click="selectElement(item)">
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
      <div class="attribute-panel" :class="{ collapsed: isPanelCollapsed }">
        <div class="panel-header">
          <div class="collapse-button" @click="togglePanel">
            <i class="fas" :class="isPanelCollapsed ? 'fa-chevron-left' : 'fa-chevron-right'"></i>
          </div>
          <h3>属性设置</h3>
        </div>
        <div class="panel-content" v-if="selectedElement">
          <div class="property-group">
            <div class="property-title">位置和大小</div>
            <div class="property-item">
              <label>X 坐标</label>
              <div class="input-with-unit">
                <input type="number" :value="elementLeft" @input="updateElementLeft($event.target.value)">
                <span class="unit">px</span>
              </div>
            </div>
            <div class="property-item">
              <label>Y 坐标</label>
              <div class="input-with-unit">
                <input type="number" :value="elementTop" @input="updateElementTop($event.target.value)">
                <span class="unit">px</span>
              </div>
            </div>
            <div class="property-item">
              <label>宽度</label>
              <div class="input-with-unit">
                <input type="number" :value="elementWidth" @input="updateElementWidth($event.target.value)">
                <span class="unit">px</span>
              </div>
            </div>
            <div class="property-item">
              <label>高度</label>
              <div class="input-with-unit">
                <input type="number" :value="elementHeight" @input="updateElementHeight($event.target.value)">
                <span class="unit">px</span>
              </div>
            </div>
          </div>

          <div class="property-group">
            <div class="property-title">样式</div>
            <div class="property-item">
              <label>背景色</label>
              <input type="color" :value="elementBgColor" @input="updateElementBgColor($event.target.value)" class="color-input">
            </div>
            <div class="property-item">
              <label>边框</label>
              <div class="input-with-unit">
                <input type="number" :value="elementBorderWidth" @input="updateElementBorderWidth($event.target.value)" min="0">
                <span class="unit">px</span>
              </div>
              <input type="color" :value="elementBorderColor" @input="updateElementBorderColor($event.target.value)" class="color-input">
            </div>
            <div class="property-item">
              <label>圆角</label>
              <div class="input-with-unit">
                <input type="number" :value="elementBorderRadius" @input="updateElementBorderRadius($event.target.value)" min="0">
                <span class="unit">px</span>
              </div>
            </div>
          </div>
        </div>
        <div class="panel-content" v-else>
          <div class="no-selection">
            请选择一个元素
          </div>
        </div>
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
        elementIndex: -1,
        startX: 0,
        startY: 0,
        originalX: 0,
        originalY: 0,
        childrenOriginalPositions: null // 用于存储子元素的初始位置
      },
      resizeState: {
        isResizing: false,
        handle: null,
        startX: 0,
        startY: 0,
        originalWidth: 0,
        originalHeight: 0,
        elementIndex: -1
      },
      currentTool: 'pointer',
      showSourceDialog: false,
      sourceCode: '',
      enableSnap: true,
      snapLines: {
        vertical: null,
        horizontal: null
      },
      isPanelCollapsed: false,
      selectedElement: null
    }
  },
  computed: {
    elementLeft: {
      get() {
        return this.selectedElement ? parseInt(this.selectedElement.style.left) || 0 : 0;
      },
      set(value) {
        if (this.selectedElement) {
          this.selectedElement.style.left = value + 'px';
        }
      }
    },
    elementTop: {
      get() {
        return this.selectedElement ? parseInt(this.selectedElement.style.top) || 0 : 0;
      },
      set(value) {
        if (this.selectedElement) {
          this.selectedElement.style.top = value + 'px';
        }
      }
    },
    elementWidth: {
      get() {
        return this.selectedElement ? parseInt(this.selectedElement.style.width) || 100 : 100;
      },
      set(value) {
        if (this.selectedElement) {
          this.selectedElement.style.width = value + 'px';
        }
      }
    },
    elementHeight: {
      get() {
        return this.selectedElement ? parseInt(this.selectedElement.style.height) || 30 : 30;
      },
      set(value) {
        if (this.selectedElement) {
          this.selectedElement.style.height = value + 'px';
        }
      }
    },
    elementBgColor: {
      get() {
        return this.selectedElement ? this.selectedElement.style.backgroundColor || '#ffffff' : '#ffffff';
      },
      set(value) {
        if (this.selectedElement) {
          this.selectedElement.style.backgroundColor = value;
        }
      }
    },
    elementBorderWidth: {
      get() {
        return this.selectedElement ? parseInt(this.selectedElement.style.borderWidth) || 0 : 0;
      },
      set(value) {
        if (this.selectedElement) {
          this.selectedElement.style.borderWidth = value + 'px';
          this.selectedElement.style.borderStyle = value > 0 ? 'solid' : 'none';
        }
      }
    },
    elementBorderColor: {
      get() {
        return this.selectedElement ? this.selectedElement.style.borderColor || '#000000' : '#000000';
      },
      set(value) {
        if (this.selectedElement) {
          this.selectedElement.style.borderColor = value;
        }
      }
    },
    elementBorderRadius: {
      get() {
        return this.selectedElement ? parseInt(this.selectedElement.style.borderRadius) || 0 : 0;
      },
      set(value) {
        if (this.selectedElement) {
          this.selectedElement.style.borderRadius = value + 'px';
        }
      }
    }
  },
  mounted() {
    // 添加全局鼠标事件监听
    window.addEventListener('mousemove', this.handleResize)
    window.addEventListener('mouseup', this.handleMouseUp)
  },
  beforeDestroy() {
    // 移除全局事件监听
    window.removeEventListener('mousemove', this.handleResize)
    window.removeEventListener('mouseup', this.handleMouseUp)
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
    startDrag(event, index) {
      // 如果是从缩放控制点开始的，不触发拖拽
      if (event.target.classList.contains('resize-handle')) {
        return;
      }

      const element = this.elements[index];
      
      // 如果元素被锁定且不是div，阻止拖动
      if (element.locked && element.type !== 'div') {
        return;
      }

      this.dragState.isDragging = true;
      this.dragState.elementIndex = index;
      this.dragState.startX = event.clientX;
      this.dragState.startY = event.clientY;
      this.dragState.originalX = parseInt(element.style.left) || 0;
      this.dragState.originalY = parseInt(element.style.top) || 0;

      // 存储所有子元素的初始位置
      if (element.type === 'div' && element.locked) {
        this.dragState.childrenOriginalPositions = this.elements
          .map((el, i) => {
            if (this.isElementInside(el, element) && i !== index) {
              return {
                index: i,
                left: parseInt(el.style.left) || 0,
                top: parseInt(el.style.top) || 0
              };
            }
            return null;
          })
          .filter(pos => pos !== null);
      }

      // 选中当前拖拽的元素
      this.selectElement(element);

      // 添加临时的全局鼠标移动事件
      window.addEventListener('mousemove', this.handleDrag);
      window.addEventListener('mouseup', this.handleDragEnd);

      // 阻止默认行为和事件冒泡
      event.preventDefault();
      event.stopPropagation();
    },

    handleDrag(event) {
      if (!this.dragState.isDragging) return;

      const index = this.dragState.elementIndex;
      const element = this.elements[index];
      
      // 如果元素被锁定且不是div，不处理移动
      if (!element || (element.locked && element.type !== 'div')) return;

      // 计算新位置
      let deltaX = event.clientX - this.dragState.startX;
      let deltaY = event.clientY - this.dragState.startY;
      
      let newLeft = this.dragState.originalX + deltaX;
      let newTop = this.dragState.originalY + deltaY;

      // 防止元素完全拖出设计面板
      const designPanel = document.querySelector('.design-panel');
      if (designPanel) {
        const panelRect = designPanel.getBoundingClientRect();
        const elementRect = {
          width: parseInt(element.style.width) || 100,
          height: parseInt(element.style.height) || 30
        };

        // 确保至少有20px在面板内
        const minVisible = 20;
        newLeft = Math.max(-elementRect.width + minVisible, Math.min(newLeft, panelRect.width - minVisible));
        newTop = Math.max(-elementRect.height + minVisible, Math.min(newTop, panelRect.height - minVisible));
      }

      // 如果是锁定的div，只处理对齐但不处理子元素的对齐
      if (!(element.type === 'div' && element.locked)) {
        // 处理对齐
        if (this.enableSnap) {
          this.snapLines.vertical = null;
          this.snapLines.horizontal = null;

          const snapThreshold = 5;
          const currentRect = {
            left: newLeft,
            right: newLeft + (parseInt(element.style.width) || 100),
            top: newTop,
            bottom: newTop + (parseInt(element.style.height) || 30),
            center: newLeft + (parseInt(element.style.width) || 100) / 2,
            middle: newTop + (parseInt(element.style.height) || 30) / 2
          };

          this.elements.forEach((other, otherIndex) => {
            if (otherIndex === index || (element.type === 'div' && this.isElementInside(other, element))) {
              return;
            }

            const otherRect = {
              left: parseInt(other.style.left) || 0,
              right: (parseInt(other.style.left) || 0) + (parseInt(other.style.width) || 100),
              top: parseInt(other.style.top) || 0,
              bottom: (parseInt(other.style.top) || 0) + (parseInt(other.style.height) || 30),
              center: (parseInt(other.style.left) || 0) + (parseInt(other.style.width) || 100) / 2,
              middle: (parseInt(other.style.top) || 0) + (parseInt(other.style.height) || 30) / 2
            };

            // 左对齐
            if (Math.abs(currentRect.left - otherRect.left) < snapThreshold) {
              newLeft = otherRect.left;
              this.snapLines.vertical = newLeft;
            }
            // 右对齐
            else if (Math.abs(currentRect.right - otherRect.right) < snapThreshold) {
              newLeft = otherRect.right - (parseInt(element.style.width) || 100);
              this.snapLines.vertical = otherRect.right;
            }
            // 中心对齐
            else if (Math.abs(currentRect.center - otherRect.center) < snapThreshold) {
              newLeft = otherRect.center - (parseInt(element.style.width) || 100) / 2;
              this.snapLines.vertical = otherRect.center;
            }

            // 顶部对齐
            if (Math.abs(currentRect.top - otherRect.top) < snapThreshold) {
              newTop = otherRect.top;
              this.snapLines.horizontal = newTop;
            }
            // 底部对齐
            else if (Math.abs(currentRect.bottom - otherRect.bottom) < snapThreshold) {
              newTop = otherRect.bottom - (parseInt(element.style.height) || 30);
              this.snapLines.horizontal = otherRect.bottom;
            }
            // 中间对齐
            else if (Math.abs(currentRect.middle - otherRect.middle) < snapThreshold) {
              newTop = otherRect.middle - (parseInt(element.style.height) || 30) / 2;
              this.snapLines.horizontal = otherRect.middle;
            }
          });
        }
      }

      // 更新元素位置
      element.style.left = newLeft + 'px';
      element.style.top = newTop + 'px';

      // 如果是锁定的div容器，同时移动其中的元素
      if (element.type === 'div' && element.locked && this.dragState.childrenOriginalPositions) {
        this.dragState.childrenOriginalPositions.forEach(pos => {
          const child = this.elements[pos.index];
          child.style.left = (pos.left + deltaX) + 'px';
          child.style.top = (pos.top + deltaY) + 'px';
        });
      }
    },

    handleDragEnd() {
      if (!this.dragState.isDragging) return;

      // 移除临时的全局事件监听
      window.removeEventListener('mousemove', this.handleDrag);
      window.removeEventListener('mouseup', this.handleDragEnd);

      // 重置拖拽状态
      this.dragState.isDragging = false;
      this.dragState.elementIndex = -1;
      this.dragState.childrenOriginalPositions = null;

      // 清除对齐线
      this.snapLines.vertical = null;
      this.snapLines.horizontal = null;
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
    },
    togglePanel() {
      this.isPanelCollapsed = !this.isPanelCollapsed;
    },
    updateElementLeft(value) {
      this.elementLeft = parseInt(value);
    },
    updateElementTop(value) {
      this.elementTop = parseInt(value);
    },
    updateElementWidth(value) {
      this.elementWidth = parseInt(value);
    },
    updateElementHeight(value) {
      this.elementHeight = parseInt(value);
    },
    updateElementBgColor(value) {
      this.elementBgColor = value;
    },
    updateElementBorderWidth(value) {
      this.elementBorderWidth = parseInt(value);
    },
    updateElementBorderColor(value) {
      this.elementBorderColor = value;
    },
    updateElementBorderRadius(value) {
      this.elementBorderRadius = parseInt(value);
    },
    selectElement(element) {
      this.selectedElement = element;
    },
    handleMouseUp(event) {
      // 如果正在拖拽，处理拖拽结束
      if (this.dragState.isDragging) {
        this.handleDragEnd(event);
      }
      // 如果正在调整大小，处理调整结束
      if (this.resizeState.isResizing) {
        this.stopResize();
      }
    },
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
  position: absolute;
  min-width: 20px;
  min-height: 20px;
}

.design-element.selected {
  outline: 2px solid #1890ff;
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

.attribute-panel {
  position: fixed;
  right: 0;
  top: 0;
  bottom: 0;
  width: 300px;
  background-color: #fff;
  border-left: 1px solid #ddd;
  display: flex;
  flex-direction: column;
  transition: transform 0.3s ease;
  z-index: 999;
}

.attribute-panel.collapsed {
  transform: translateX(100%);
}

.collapse-button {
  position: fixed;
  right: 300px;
  top: 50%;
  transform: translateY(-50%);
  width: 16px;
  height: 32px;
  background-color: #fff;
  border: 1px solid #ddd;
  border-right: none;
  border-radius: 4px 0 0 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 1000;
  transition: right 0.3s ease;
}

.collapse-button.collapsed {
  right: 0;
}

.collapse-button:hover {
  background-color: #f0f0f0;
}

.collapse-button i {
  font-size: 12px;
  color: #666;
}

.panel-header {
  padding: 16px;
  border-bottom: 1px solid #ddd;
}

.panel-header h3 {
  margin: 0;
  font-size: 14px;
  color: #333;
  flex: 1;
}

.panel-content {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
}

.property-group {
  margin-bottom: 24px;
}

.property-title {
  font-size: 13px;
  color: #666;
  margin-bottom: 12px;
}

.property-item {
  margin-bottom: 12px;
  display: flex;
  align-items: center;
}

.property-item label {
  width: 60px;
  font-size: 14px;
  color: #333;
}

.input-with-unit {
  flex: 1;
  display: flex;
  align-items: center;
  position: relative;
}

.input-with-unit input {
  width: 100%;
  height: 28px;
  padding: 4px 24px 4px 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  outline: none;
}

.input-with-unit input:focus {
  border-color: #1890ff;
}

.input-with-unit .unit {
  position: absolute;
  right: 8px;
  color: #999;
  font-size: 12px;
}

.no-selection {
  text-align: center;
  color: #999;
  padding: 20px;
  font-size: 14px;
}

.color-input {
  width: 32px;
  height: 28px;
  padding: 0 4px;
  margin-left: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
}

.color-input::-webkit-color-swatch-wrapper {
  padding: 0;
}

.color-input::-webkit-color-swatch {
  border: none;
  border-radius: 2px;
}
</style>
