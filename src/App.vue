<!--
 * @Version    : v1.00
 * @Author     : Wang Chao
 * @Date       : 2025-04-24 00:28
 * @LastAuthor : Wang Chao
 * @LastTime   : 2025-04-24 00:28
 * @desc       : 
-->
<template>
  <div class="container">
    <div class="json-container">
      <h3>原来的 JSON</h3>
      <codemirror
        v-model="originalJson"
        :extensions="extensions"
        :indent-with-tab="true"
        :tab-size="2"
        placeholder="请输入原来的JSON"
        class="editor"
      />
    </div>
    <div class="json-container">
      <h3>新增的 JSON</h3>
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
</template>

<script setup>
  // 这是一个简单的Vue3组件，使用组合式API
  import { ref } from 'vue';
  import { Codemirror } from 'vue-codemirror';
  import { json } from '@codemirror/lang-json';
  import { oneDark } from '@codemirror/theme-one-dark';
  import { basicSetup } from 'codemirror';

  const originalJson = ref('');
  const newJson = ref('');

  // 配置编辑器扩展
  const extensions = [basicSetup, json(), oneDark];
</script>

<style scoped>
  .container {
    display: flex;
    width: 100%;
    min-height: 100vh;
    padding: 20px;
    box-sizing: border-box;
  }

  .json-container {
    flex: 1;
    padding: 10px;
    display: flex;
    flex-direction: column;
  }

  h3 {
    margin-top: 0;
    margin-bottom: 10px;
    text-align: center;
  }

  .editor {
    flex: 1;
    height: 500px;
    border: 1px solid #ccc;
    border-radius: 4px;
    overflow: hidden;
    font-family: monospace;
  }

  :deep(.cm-editor) {
    height: 100%;
  }

  :deep(.cm-scroller) {
    overflow: auto;
  }
</style>
