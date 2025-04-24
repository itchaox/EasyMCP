<!--
 * @Version    : v1.00
 * @Author     : Wang Chao
 * @Date       : 2025-04-24 00:28
 * @LastAuthor : Wang Chao
 * @LastTime   : 2025-04-24 00:28
 * @desc       : 
-->
<template>
  <div class="app-container">
    <header class="app-header">
      <div class="logo">
        <span class="logo-text">EasyMCP</span>
        <span class="logo-subtitle">JSON配置助手</span>
      </div>
    </header>

    <main class="main-content">
      <div class="editors-container">
        <div class="json-panel">
          <div class="panel-header">
            <h3>原来的 JSON</h3>
          </div>
          <codemirror
            v-model="originalJson"
            :extensions="extensions"
            :indent-with-tab="true"
            :tab-size="2"
            placeholder="请输入原来的JSON"
            class="editor"
          />
        </div>

        <div class="json-panel">
          <div class="panel-header">
            <h3>新增的 JSON</h3>
          </div>
          <codemirror
            v-model="newJson"
            :extensions="extensions"
            :indent-with-tab="true"
            :tab-size="2"
            placeholder="请输入新增的JSON"
            class="editor"
          />
        </div>
      </div>
    </main>

    <footer class="app-footer">
      <p>© 2023 EasyMCP - 让JSON配置更简单</p>
    </footer>
  </div>
</template>

<script setup>
  // 这是一个简单的Vue3组件，使用组合式API
  import { ref, onMounted } from 'vue';
  import { Codemirror } from 'vue-codemirror';
  import { json } from '@codemirror/lang-json';
  import { oneDark } from '@codemirror/theme-one-dark';
  import { basicSetup } from 'codemirror';

  const originalJson = ref('');
  const newJson = ref('');

  // 配置编辑器扩展
  const extensions = [basicSetup, json(), oneDark];

  // 示例JSON数据
  const sampleJson = {
    name: 'EasyMCP',
    version: '1.0.0',
    description: 'JSON配置助手',
    author: 'Wang Chao',
  };

  onMounted(() => {
    // 为了演示，可以设置一些初始值
    originalJson.value = JSON.stringify(sampleJson, null, 2);
  });
</script>

<style scoped>
  /* 全局样式 */
  .app-container {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    background-color: #f5f7fa;
    font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
  }

  /* 头部样式 */
  .app-header {
    background: linear-gradient(135deg, #4b6cb7 0%, #182848 100%);
    color: white;
    padding: 16px 24px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    z-index: 10;
  }

  .logo {
    display: flex;
    flex-direction: column;
  }

  .logo-text {
    font-size: 24px;
    font-weight: 700;
    letter-spacing: 0.5px;
  }

  .logo-subtitle {
    font-size: 14px;
    opacity: 0.8;
  }

  /* 主内容区域 */
  .main-content {
    flex: 1;
    padding: 24px;
    display: flex;
    flex-direction: column;
  }

  .editors-container {
    display: flex;
    gap: 24px;
    height: calc(100vh - 180px);
    margin-bottom: 24px;
  }

  @media (max-width: 768px) {
    .editors-container {
      flex-direction: column;
      height: auto;
    }
  }

  /* JSON面板样式 */
  .json-panel {
    flex: 1;
    display: flex;
    flex-direction: column;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
    overflow: hidden;
    transition: all 0.3s ease;
  }

  .json-panel:hover {
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1);
    transform: translateY(-2px);
  }

  .panel-header {
    padding: 16px;
    border-bottom: 1px solid #eaeaea;
    background-color: #f9fafc;
  }

  .panel-header h3 {
    margin: 0;
    color: #333;
    font-size: 18px;
    font-weight: 600;
  }

  /* 编辑器样式 */
  .editor {
    flex: 1;
    overflow: hidden;
    font-family: 'Fira Code', 'Consolas', monospace;
  }

  :deep(.cm-editor) {
    height: 100%;
    font-size: 14px;
  }

  :deep(.cm-scroller) {
    overflow: auto;
    padding: 8px 0;
  }

  :deep(.cm-gutters) {
    border-right: 1px solid rgba(0, 0, 0, 0.1);
    background-color: #f9fafc;
  }

  :deep(.cm-activeLineGutter) {
    background-color: rgba(75, 108, 183, 0.1);
  }

  /* 底部样式 */
  .app-footer {
    padding: 16px 24px;
    text-align: center;
    color: #666;
    font-size: 14px;
    background-color: white;
    border-top: 1px solid #eaeaea;
  }
</style>
