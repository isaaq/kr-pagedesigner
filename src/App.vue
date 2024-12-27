<template>
  <div class="app">
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
        <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'text')">文本</div>
        <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'button')">按钮</div>
        <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'input')">输入框</div>
        <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'div')">DIV</div>
      </div>
      <div class="design-panel" @dragover.prevent @drop="onDrop">
        <template v-for="(item, index) in elements" :key="index">
          <div v-if="item.type === 'text'" 
               :style="item.style"
               class="design-element"
               draggable="true"
               @dragstart="onElementDragStart($event, index)"
               @drag="onElementDrag($event, index)">示例文本</div>
          <button v-else-if="item.type === 'button'" 
                  :style="item.style"
                  class="design-element"
                  draggable="true"
                  @dragstart="onElementDragStart($event, index)"
                  @drag="onElementDrag($event, index)">按钮</button>
          <input v-else-if="item.type === 'input'" 
                 :style="item.style"
                 class="design-element"
                 draggable="true"
                 @dragstart="onElementDragStart($event, index)"
                 @drag="onElementDrag($event, index)"
                 placeholder="请输入..."/>
          <div v-else-if="item.type === 'div'" 
               :style="item.style"
               class="design-element"
               draggable="true"
               @dragstart="onElementDragStart($event, index)"
               @drag="onElementDrag($event, index)"></div>
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
        originalY: 0
      },
      currentTool: 'pointer',
      showSourceDialog: false,
      sourceCode: ''
    }
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
          borderRadius: '4px'
        })
      }

      this.elements.push(element)
    },
    onElementDragStart(event, index) {
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
        originalY: parseInt(style.top)
      }
    },
    onElementDrag(event, index) {
      if (!event.clientX || !event.clientY) return // 忽略无效的拖动事件
      
      if (this.dragState.isDragging && this.dragState.elementIndex === index) {
        const deltaX = event.clientX - this.dragState.startX
        const deltaY = event.clientY - this.dragState.startY
        
        const newX = this.dragState.originalX + deltaX
        const newY = this.dragState.originalY + deltaY
        
        this.elements[index].style.left = `${newX}px`
        this.elements[index].style.top = `${newY}px`
      }
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

.design-container {
  flex: 1;
  display: flex;
  overflow: hidden;
}

.toolbar {
  width: 200px;
  padding: 20px;
  border-right: 1px solid #ddd;
  background-color: #f5f5f5;
}

.tool-item {
  padding: 10px;
  margin-bottom: 10px;
  background-color: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: move;
  user-select: none;
}

.tool-item:hover {
  background-color: #f0f0f0;
}

.design-panel {
  flex: 1;
  position: relative;
  overflow: auto;
  padding: 20px;
}

.design-element {
  cursor: move;
  user-select: none;
}

.design-element:hover {
  outline: 1px solid #1890ff;
}

.source-dialog {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
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
