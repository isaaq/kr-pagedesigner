<template>
  <div class="toolbar">
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
      <div class="tool-category">
        <div class="tool-category-title">
          表单元素
          <label class="checkbox-label" style="float: right;">
            <input type="checkbox" v-model="showLabels"> 标签
          </label>
        </div>
        <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'input')">
          <i class="fas fa-keyboard"></i> 输入框
        </div>
        <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'textarea')">
          <i class="fas fa-align-left"></i> 多行文本
        </div>
        <div class="tool-item" draggable="true" @dragstart="onDragStart($event, 'select')">
          <i class="fas fa-caret-square-down"></i> 下拉选择
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LeftToolbar',
  methods: {
    onDragStart(event, type) {
      event.dataTransfer.setData('type', type);
      event.dataTransfer.effectAllowed = 'move';
    }
  }
}
</script>

<style scoped>
.toolbar {
  width: 200px;
  background-color: #f5f5f5;
  border-right: 1px solid #e8e8e8;
  padding: 16px;
  height: 100%;
  overflow-y: auto;
}

.tool-items {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.tool-category {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.tool-category-title {
  font-size: 14px;
  font-weight: bold;
  color: #333;
  margin-bottom: 8px;
}

.tool-item {
  padding: 8px 12px;
  border: 1px solid #e8e8e8;
  border-radius: 4px;
  cursor: move;
  transition: all 0.2s;
  font-size: 14px;
}

.tool-item:hover {
  background-color: #e6f7ff;
  border-color: #1890ff;
}

.tool-item i {
  margin-right: 8px;
}
</style>
