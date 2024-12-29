<template>
  <div class="app">
    <div class="top-toolbar">
      <div class="toolbar-group">
        <button class="toolbar-btn" @click="addElement('text')">
          <i class="fas fa-font"></i> 文本
        </button>
        <button class="toolbar-btn" @click="addElement('button')">
          <i class="fas fa-square"></i> 按钮
        </button>
        <button class="toolbar-btn" @click="addElement('input')">
          <i class="fas fa-keyboard"></i> 输入框
        </button>
        <button class="toolbar-btn" @click="addElement('div')">
          <i class="fas fa-vector-square"></i> 框选
        </button>
        <button class="toolbar-btn" @click="addElement('textarea')">
          <i class="fas fa-text-height"></i> 文本框
        </button>
        <button class="toolbar-btn" @click="addElement('select')">
          <i class="fas fa-caret-down"></i> 下拉框
        </button>
      </div>
      <div class="toolbar-group">
        <select class="screen-select" v-model="currentScreen">
          <option value="desktop">Desktop (1920px)</option>
          <option value="laptop">Laptop (1366px)</option>
          <option value="tablet">Tablet (768px)</option>
          <option value="mobile">Mobile (375px)</option>
        </select>
        <div class="zoom-controls">
          <button class="toolbar-btn" @click="zoomOut">
            <i class="fas fa-search-minus"></i>
          </button>
          <span class="zoom-level">{{ Math.round(scale * 100) }}%</span>
          <button class="toolbar-btn" @click="zoomIn">
            <i class="fas fa-search-plus"></i>
          </button>
        </div>
      </div>
      <div class="toolbar-group">
        <button class="toolbar-btn" @click="showSourceCode">
          <i class="fas fa-code"></i> 源代码
        </button>
        <button class="toolbar-btn" @click="toggleSnap">
          <i class="fas fa-magnet"></i> {{ enableSnap ? '关闭对齐' : '开启对齐' }}
        </button>
        <button class="toolbar-btn" @click="togglePanel">
          <i class="fas" :class="isPanelCollapsed ? 'fa-chevron-left' : 'fa-chevron-right'"></i>
        </button>
      </div>
    </div>
    <div class="design-container">
      <LeftToolbar />
      <div class="design-panel-wrapper">
        <div class="design-panel" 
          :style="panelStyle"
          @dragover.prevent 
          @drop="onDrop"
          @mousemove="handleMouseMove"
          @mouseup="handleMouseUp">
          <!-- 对齐辅助线 -->
          <div v-if="enableSnap && snapLines.vertical" 
               class="snap-line vertical"
               :style="{ left: snapLines.vertical + 'px' }">
          </div>
          <div v-if="enableSnap && snapLines.horizontal" 
               class="snap-line horizontal"
               :style="{ top: snapLines.horizontal + 'px' }">
          </div>
          <template v-for="(item, index) in elements" :key="item.id">
            <div v-if="item.type === 'text'" 
                 :style="item.style"
                 class="design-element"
                 :class="{ selected: selectedElement === item }"
                 :data-id="item.id"
                 @mousedown="handleMouseDown"
                 @click="selectElement(item)">
              示例文本
            </div>
            <div v-else-if="item.type === 'button'" 
                 :style="item.style"
                 class="design-element"
                 :class="{ selected: selectedElement === item }"
                 :data-id="item.id"
                 @mousedown="handleMouseDown"
                 @click="selectElement(item)">
              <button style="width: 100%; height: 100%; border-radius: 4px; border: 1px solid #ccc;">按钮</button>
            </div>
            <div v-else-if="item.type === 'input'" 
                 :style="item.style"
                 class="design-element"
                 :class="{ selected: selectedElement === item }"
                 :data-id="item.id"
                 @mousedown="handleMouseDown"
                 @click="selectElement(item)">
              <input type="text" placeholder="请输入..." style="width: 100%; height: 100%; border-radius: 4px; border: 1px solid #ccc; box-sizing: border-box;" />
            </div>
            <div v-else-if="item.type === 'textarea'" 
                 :style="item.style"
                 class="design-element"
                 :class="{ selected: selectedElement === item }"
                 :data-id="item.id"
                 @mousedown="handleMouseDown"
                 @click="selectElement(item)">
              <textarea placeholder="请输入..." style="width: 100%; height: 100%; border-radius: 4px; border: 1px solid #ccc; padding: 4px 8px; box-sizing: border-box;"></textarea>
            </div>
            <div v-else-if="item.type === 'select'" 
                 :style="item.style"
                 class="design-element"
                 :class="{ selected: selectedElement === item }"
                 :data-id="item.id"
                 @mousedown="handleMouseDown"
                 @click="selectElement(item)">
              <select style="width: 100%; height: 100%; border-radius: 4px; border: 1px solid #ccc; padding: 4px 8px; box-sizing: border-box;">
                <option value="">请选择</option>
              </select>
            </div>
            <div v-else-if="item.type === 'div'" 
                 :style="item.style"
                 class="design-element"
                 :class="{ selected: selectedElement === item }"
                 :data-id="item.id"
                 @mousedown="handleMouseDown"
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
    <div v-if="showSourceDialog" class="dialog" style="position: absolute; z-index: 1000; top: 50%; left: 50%; transform: translate(-50%, -50%); background-color: white; border-radius: 8px; padding: 20px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); width: 600px; height: 400px;">
      <textarea class="dialog-textarea" v-model="sourceCode" rows="10" style="width: 100%; height: 300px; border-radius: 4px; padding: 10px; border: 1px solid #ccc; resize: none;"></textarea>
      <div class="dialog-buttons" style="display: flex; justify-content: space-between; margin-top: 10px;">
        <button @click="saveCode" style="flex: 1; margin-right: 5px;">保存</button>
        <button @click="closeDialog" style="flex: 1; margin-left: 5px;">关闭</button>
      </div>
    </div>
  </div>
</template>

<script>
import LeftToolbar from './components/LeftToolbar.vue'

export default {
  name: 'App',
  components: {
    LeftToolbar
  },
  data() {
    return {
      elements: [],
      dragState: {
        isDragging: false,
        elementIndex: -1,
        startX: 0,
        startY: 0,
        originalX: 0,
        originalY: 0
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
      selectedElement: null,
      currentScreen: 'desktop',
      screenSizes: {
        desktop: { width: 1920, height: 1080 },
        laptop: { width: 1366, height: 768 },
        tablet: { width: 768, height: 1024 },
        mobile: { width: 375, height: 667 }
      },
      scale: 0.5,
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
    },
    panelStyle() {
      const size = this.screenSizes[this.currentScreen];
      return {
        width: `${size.width}px`,
        height: `${size.height}px`,
        transform: `scale(${this.scale})`,
        transformOrigin: 'top left',
        backgroundColor: '#ffffff',
        position: 'relative',
        transition: 'width 0.3s, height 0.3s, transform 0.3s'
      }
    }
  },
  mounted() {
    // 添加全局鼠标事件监听
    window.addEventListener('mousemove', this.handleMouseMove)
    window.addEventListener('mouseup', this.handleMouseUp)
    // 添加自适应缩放
    this.$nextTick(() => {
      this.adjustScaleToFit();
      window.addEventListener('resize', this.adjustScaleToFit);
    });
  },
  beforeDestroy() {
    // 移除全局事件监听
    window.removeEventListener('mousemove', this.handleMouseMove)
    window.removeEventListener('mouseup', this.handleMouseUp)
    window.removeEventListener('resize', this.adjustScaleToFit)
  },
  methods: {
    onDrop(event) {
      const type = event.dataTransfer.getData('type');
      if (!type) return;

      const rect = event.target.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;

      this.addElement(type, x, y);
    },
    addElement(type, x = 100, y = 100) {
      const element = {
        id: Date.now(),
        type,
        style: {
          position: 'absolute',
          left: `${x}px`,
          top: `${y}px`,
          cursor: 'move'
        }
      };

      // 为 DIV 添加特殊样式
      if (type === 'div') {
        Object.assign(element.style, {
          width: '200px',
          height: '150px',
          border: '2px dashed #4CAF50',
          backgroundColor: 'rgba(76, 175, 80, 0.1)',
          zIndex: 0  // 确保 div 在最底层
        });
        element.locked = false;
        this.elements.unshift(element);
      } else if (type === 'textarea') {
        Object.assign(element.style, {
          width: '200px',
          height: '100px',
          border: '1px solid #ccc',
          borderRadius: '4px',
          backgroundColor: '#fff'
        });
        this.elements.push(element);
      } else if (type === 'select') {
        Object.assign(element.style, {
          width: '200px',
          height: '40px',
          border: '1px solid #ccc',
          borderRadius: '4px',
          backgroundColor: '#fff'
        });
        this.elements.push(element);
      } else {
        // 其他元素添加更高的 z-index
        element.style.zIndex = 1;
        this.elements.push(element);
      }

      // 选中新创建的元素
      this.selectElement(element);
    },
    handleMouseDown(event) {
      // 如果点击的是锁定图标，不处理拖拽
      if (event.target.classList.contains('lock-icon')) {
        return;
      }

      // 找到被点击的元素的DOM节点
      const elementNode = event.target.closest('.design-element');
      if (!elementNode) return;

      // 找到对应的元素数据
      const elementId = elementNode.dataset.id;
      const elementIndex = this.elements.findIndex(el => el.id === Number(elementId));
      if (elementIndex === -1) return;

      const element = this.elements[elementIndex];
      
      // 如果元素被锁定且不是div，不允许拖动
      if (element.locked && element.type !== 'div') return;

      // 如果是锁定的div，记录子元素的初始位置
      let childrenPositions = null;
      if (element.type === 'div' && element.locked) {
        childrenPositions = this.elements
          .map((el, i) => {
            if (this.isElementInside(el, element) && i !== elementIndex) {
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

      this.dragState = {
        isDragging: true,
        elementIndex,
        startX: event.clientX,
        startY: event.clientY,
        originalX: parseInt(element.style.left),
        originalY: parseInt(element.style.top),
        childrenPositions // 保存子元素位置信息
      };

      // 选中当前拖拽的元素
      this.selectElement(element);

      // 阻止默认行为和事件冒泡
      event.preventDefault();
      event.stopPropagation();
    },

    handleMouseMove(event) {
      if (!this.dragState.isDragging) return;

      const element = this.elements[this.dragState.elementIndex];
      if (!element) return;

      const dx = event.clientX - this.dragState.startX;
      const dy = event.clientY - this.dragState.startY;

      let newX = this.dragState.originalX + dx;
      let newY = this.dragState.originalY + dy;

      // 如果启用了对齐功能，检查是否需要对齐
      if (this.enableSnap) {
        const snapResult = this.checkSnap(newX, newY, element);
        newX = snapResult.x;
        newY = snapResult.y;
      }

      // 更新元素位置
      element.style.left = `${newX}px`;
      element.style.top = `${newY}px`;

      // 如果是锁定的div，同时移动其中的元素
      if (element.type === 'div' && element.locked && this.dragState.childrenPositions) {
        this.dragState.childrenPositions.forEach(pos => {
          const child = this.elements[pos.index];
          if (child) {
            child.style.left = `${pos.left + dx}px`;
            child.style.top = `${pos.top + dy}px`;
          }
        });
      }
    },

    handleMouseUp() {
      // 如果正在拖拽，处理拖拽结束
      if (this.dragState.isDragging) {
        this.dragState = {
          isDragging: false,
          elementIndex: -1,
          startX: 0,
          startY: 0,
          originalX: 0,
          originalY: 0
        };
      }

      // 如果正在调整大小，处理调整结束
      if (this.resizeState.isResizing) {
        this.stopResize();
      }

      // 清除对齐线
      this.snapLines.vertical = null;
      this.snapLines.horizontal = null;
    },

    checkSnap(x, y, element) {
      const snapThreshold = 5;
      const currentRect = {
        left: x,
        right: x + (parseInt(element.style.width) || 100),
        top: y,
        bottom: y + (parseInt(element.style.height) || 30),
        center: x + (parseInt(element.style.width) || 100) / 2,
        middle: y + (parseInt(element.style.height) || 30) / 2
      };

      this.elements.forEach((other, otherIndex) => {
        if (otherIndex === this.dragState.elementIndex) {
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
          x = otherRect.left;
          this.snapLines.vertical = otherRect.left;
        }
        // 右对齐
        else if (Math.abs(currentRect.right - otherRect.right) < snapThreshold) {
          x = otherRect.right - (parseInt(element.style.width) || 100);
          this.snapLines.vertical = otherRect.right;
        }
        // 中心对齐
        else if (Math.abs(currentRect.center - otherRect.center) < snapThreshold) {
          x = otherRect.center - (parseInt(element.style.width) || 100) / 2;
          this.snapLines.vertical = otherRect.center;
        }

        // 顶部对齐
        if (Math.abs(currentRect.top - otherRect.top) < snapThreshold) {
          y = otherRect.top;
          this.snapLines.horizontal = otherRect.top;
        }
        // 底部对齐
        else if (Math.abs(currentRect.bottom - otherRect.bottom) < snapThreshold) {
          y = otherRect.bottom - (parseInt(element.style.height) || 30);
          this.snapLines.horizontal = otherRect.bottom;
        }
        // 中间对齐
        else if (Math.abs(currentRect.middle - otherRect.middle) < snapThreshold) {
          y = otherRect.middle - (parseInt(element.style.height) || 30) / 2;
          this.snapLines.horizontal = otherRect.middle;
        }
      });

      return { x, y };
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
          case 'textarea':
            elementCode = `<textarea style="${this.styleToString(style)}" placeholder="请输入..."></textarea>`;
            break;
          case 'select':
            elementCode = `<select style="${this.styleToString(style)}"><option value="">请选择</option></select>`;
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
    closeDialog() {
      this.showSourceDialog = false;
    },
    async saveCode() {
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
    isElementInside(element1, element2) {
      const rect1 = {
        left: parseInt(element1.style.left) || 0,
        top: parseInt(element1.style.top) || 0,
        right: (parseInt(element1.style.left) || 0) + (parseInt(element1.style.width) || 100),
        bottom: (parseInt(element1.style.top) || 0) + (parseInt(element1.style.height) || 30)
      };
      
      const rect2 = {
        left: parseInt(element2.style.left) || 0,
        top: parseInt(element2.style.top) || 0,
        right: (parseInt(element2.style.left) || 0) + (parseInt(element2.style.width) || 200),
        bottom: (parseInt(element2.style.top) || 0) + (parseInt(element2.style.height) || 150)
      };
      
      return rect1.left >= rect2.left && 
             rect1.right <= rect2.right && 
             rect1.top >= rect2.top && 
             rect1.bottom <= rect2.bottom;
    },
    adjustScaleToFit() {
      const wrapper = document.querySelector('.design-panel-wrapper');
      if (!wrapper) return;
      const containerWidth = wrapper.clientWidth - 40;
      const containerHeight = wrapper.clientHeight - 40;
      const size = this.screenSizes[this.currentScreen];
      const scaleX = containerWidth / size.width;
      const scaleY = containerHeight / size.height;
      this.scale = Math.min(Math.min(scaleX, scaleY), 2); // 确保缩放比例不超过1
    },
    zoomIn() {
      this.scale += 0.1;
    },
    zoomOut() {
      this.scale = Math.max(this.scale - 0.1, 0.1);
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

.top-toolbar {
  padding: 8px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #ddd;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.toolbar-group {
  display: flex;
  align-items: center;
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

.toolbar-btn i {
  font-size: 14px;
}

.design-container {
  display: flex;
  height: calc(100vh - 50px);
  overflow: hidden;
}

.design-panel-wrapper {
  flex: 1;
  overflow: auto;
  padding: 20px;
  background-color: #f0f0f0;
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
}

.design-panel {
  background-color: white;
  border: 1px solid #ddd;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  position: relative;
  overflow: hidden;
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
  box-sizing: border-box;
}

.design-element textarea {
  width: 100%;
  height: 100%;
  border: 1px solid #ddd;
  background-color: white;
  border-radius: 4px;
  padding: 4px 8px;
  pointer-events: none;
  box-sizing: border-box;
}

.design-element select {
  width: 100%;
  height: 100%;
  border: 1px solid #ddd;
  background-color: white;
  border-radius: 4px;
  padding: 4px 8px;
  pointer-events: none;
  box-sizing: border-box;
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

.dialog {
  position: absolute;
  z-index: 1000;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  width: 600px;
  height: 400px;
}

.dialog-textarea {
  width: 100%;
  height: 150px;
  border-radius: 4px;
  padding: 10px;
  border: 1px solid #ccc;
  resize: none;
}

.dialog-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
}

.dialog-buttons button {
  flex: 1;
  margin: 0 5px;
  padding: 8px 12px;
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

.screen-select {
  padding: 6px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: white;
  font-size: 14px;
  cursor: pointer;
}

.zoom-controls {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 0 8px;
  border-left: 1px solid #ddd;
  margin-left: 8px;
}

.zoom-level {
  font-size: 14px;
  color: #666;
  min-width: 50px;
  text-align: center;
}
</style>
