<!--
 * @Version    : v1.00
 * @Author     : Wang Chao
 * @Date       : 2025-04-24 00:28
 * @LastAuthor : Wang Chao
 * @LastTime   : 2025-04-24 10:58
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
            <h3>原始配置</h3>
            <p class="panel-description">请在此粘贴您的原始MCP配置JSON</p>
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
            <h3>新增配置</h3>
            <p class="panel-description">请在此输入需要新增的服务器配置</p>
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

      <div class="button-container">
        <button
          class="get-config-btn"
          @click="getMergedConfig"
        >
          合并配置
        </button>
      </div>

      <!-- 合并结果对话框 -->
      <div
        class="modal-overlay"
        v-if="showMergeResult"
        @click="closeMergeResult"
      >
        <div
          class="modal-container"
          @click.stop
        >
          <div class="modal-header">
            <h3>合并结果</h3>
            <button
              class="close-modal-btn"
              @click="closeMergeResult"
            >
              ×
            </button>
          </div>
          <div class="modal-content">
            <p class="modal-description">新增的配置已成功合并，下方显示合并后的完整配置：</p>
            <div class="merged-json-container">
              <pre
                class="merged-json"
                v-html="highlightedJson"
              ></pre>
            </div>
            <div class="modal-actions">
              <button
                class="copy-btn"
                @click="copyToClipboard"
              >
                复制到剪贴板
              </button>
              <button
                class="apply-btn"
                @click="applyMergedConfig"
              >
                应用到原始配置
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- 提示消息 -->
      <div
        class="message-container"
        v-if="showMessage"
      >
        <div
          class="message-box"
          :class="messageType"
        >
          {{ message }}
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
  import { ref, onMounted, computed } from 'vue';
  import { Codemirror } from 'vue-codemirror';
  import { json } from '@codemirror/lang-json';
  import { oneDark } from '@codemirror/theme-one-dark';
  import { basicSetup } from 'codemirror';

  const originalJson = ref('');
  const newJson = ref('');
  const mergedJson = ref('');
  const message = ref('');
  const showMessage = ref(false);
  const messageType = ref('success');
  const showMergeResult = ref(false);
  const addedServerNames = ref([]);

  // 配置编辑器扩展
  const extensions = [basicSetup, json(), oneDark];

  // 示例JSON数据
  const sampleJson = {
    mcpServers: {
      'mcp-server-hotnews': {
        command: 'npx',
        args: [
          '-y',
          '@smithery/cli@latest',
          'run',
          '@wopal/mcp-server-hotnews',
          '--key',
          '86f8c61b-8362-42ca-bd58-72d93b2b67ae',
        ],
      },
    },
  };

  // 显示消息提示
  const showMessageTip = (msg, type = 'success') => {
    message.value = msg;
    messageType.value = type;
    showMessage.value = true;

    // 3秒后自动隐藏消息
    setTimeout(() => {
      showMessage.value = false;
    }, 3000);
  };

  // 高亮显示JSON中新增的部分
  const highlightedJson = computed(() => {
    if (!mergedJson.value) return '';

    try {
      const jsonObj = JSON.parse(mergedJson.value);

      // 将JSON转为格式化的字符串并添加高亮
      const lines = JSON.stringify(jsonObj, null, 2).split('\n');
      let result = '';

      for (let i = 0; i < lines.length; i++) {
        const line = lines[i];
        let highlighted = line;

        // 检查是否包含新增的服务器名称
        for (const serverName of addedServerNames.value) {
          if (line.includes(`"${serverName}":`)) {
            // 高亮整个服务器配置块
            let j = i;
            let braceCount = 0;
            let foundOpening = false;

            do {
              const currentLine = lines[j];

              if (currentLine.includes('{')) {
                foundOpening = true;
                braceCount += (currentLine.match(/{/g) || []).length;
              }

              if (currentLine.includes('}')) {
                braceCount -= (currentLine.match(/}/g) || []).length;
              }

              if (foundOpening) {
                lines[j] = `<span class="highlight-added">${lines[j]}</span>`;
              }

              j++;
            } while (j < lines.length && (braceCount > 0 || !foundOpening));

            highlighted = lines[i];
            break;
          }
        }

        result += highlighted + '\n';
      }

      return result;
    } catch (e) {
      console.error('高亮JSON出错:', e);
      return mergedJson.value;
    }
  });

  // 复制到剪贴板
  const copyToClipboard = () => {
    navigator.clipboard
      .writeText(mergedJson.value)
      .then(() => {
        showMessageTip('配置已复制到剪贴板！');
      })
      .catch((err) => {
        console.error('复制到剪贴板失败:', err);
        showMessageTip('复制到剪贴板失败，请手动复制。', 'warning');
      });
  };

  // 应用合并结果到原始配置区域
  const applyMergedConfig = () => {
    originalJson.value = mergedJson.value;
    showMergeResult.value = false;
    showMessageTip('已更新原始配置！');
  };

  // 关闭合并结果面板
  const closeMergeResult = () => {
    showMergeResult.value = false;
  };

  // 合并JSON并展示结果
  const getMergedConfig = () => {
    try {
      // 解析两个JSON
      let original = JSON.parse(originalJson.value);
      let newConfig = JSON.parse(newJson.value);

      // 获取新配置的服务器名称和配置
      const newServerEntries = Object.entries(newConfig.mcpServers || {});

      if (newServerEntries.length === 0) {
        throw new Error('新增JSON中没有找到有效的mcpServers配置');
      }

      // 确保原始JSON有mcpServers对象
      if (!original.mcpServers) {
        original.mcpServers = {};
      }

      // 记录新增的服务器名称，用于高亮显示
      addedServerNames.value = [];

      // 将新配置添加到原始配置中
      for (const [serverName, serverConfig] of newServerEntries) {
        original.mcpServers[serverName] = serverConfig;
        addedServerNames.value.push(serverName);
      }

      // 格式化结果并保存到合并结果
      const result = JSON.stringify(original, null, 2);
      mergedJson.value = result;

      // 显示合并结果对话框
      showMergeResult.value = true;

      // 提示成功
      showMessageTip('配置已成功合并！');
    } catch (error) {
      console.error('处理JSON时出错:', error);
      showMessageTip(`错误: ${error.message}`, 'error');
    }
  };

  onMounted(() => {
    // 设置左侧原始配置示例
    originalJson.value = JSON.stringify(sampleJson, null, 2);

    // 设置新增JSON示例
    const newServerExample = {
      mcpServers: {
        'mcp-server-new': {
          command: 'npx',
          args: [
            '-y',
            '@smithery/cli@latest',
            'run',
            '@wopal/mcp-server-new',
            '--key',
            '12345678-1234-1234-1234-123456789abc',
          ],
        },
      },
    };

    // 设置右侧新增配置
    newJson.value = JSON.stringify(newServerExample, null, 2);
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
    height: calc(100vh - 300px);
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

  .panel-description {
    font-size: 14px;
    color: #666;
    margin-top: 8px;
    margin-bottom: 0;
  }

  /* 对话框样式 */
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 100;
    animation: fade-in 0.2s ease;
  }

  .modal-container {
    background-color: white;
    border-radius: 8px;
    width: 80%;
    max-width: 800px;
    max-height: 90vh;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
    display: flex;
    flex-direction: column;
    animation: slide-up 0.3s ease;
  }

  .modal-header {
    padding: 16px 20px;
    border-bottom: 1px solid #eaeaea;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .modal-header h3 {
    margin: 0;
    font-size: 20px;
    color: #333;
  }

  .close-modal-btn {
    background: none;
    border: none;
    font-size: 24px;
    color: #666;
    cursor: pointer;
    padding: 0;
    line-height: 1;
  }

  .close-modal-btn:hover {
    color: #333;
  }

  .modal-content {
    padding: 20px;
    overflow-y: auto;
    flex: 1;
  }

  .modal-description {
    margin-top: 0;
    margin-bottom: 16px;
    color: #555;
  }

  .merged-json-container {
    background-color: #282c34;
    border-radius: 6px;
    padding: 16px;
    overflow: auto;
    max-height: 50vh;
    margin-bottom: 20px;
  }

  .merged-json {
    font-family: 'Fira Code', 'Consolas', monospace;
    font-size: 14px;
    line-height: 1.5;
    color: #abb2bf;
    margin: 0;
    white-space: pre-wrap;
    tab-size: 2;
  }

  .highlight-added {
    background-color: rgba(80, 250, 123, 0.2);
    color: #50fa7b;
    display: inline-block;
    width: 100%;
  }

  .modal-actions {
    display: flex;
    justify-content: flex-end;
    gap: 12px;
  }

  @keyframes fade-in {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  @keyframes slide-up {
    from {
      transform: translateY(20px);
      opacity: 0.8;
    }
    to {
      transform: translateY(0);
      opacity: 1;
    }
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
    background-color: #282c34;
    color: #636d83;
  }

  :deep(.cm-activeLineGutter) {
    background-color: #2c313a;
    color: #adb7c9;
  }

  /* 移除编辑器获得焦点时顶部的虚线 */
  :deep(.cm-editor.cm-focused) {
    outline: none !important;
  }

  /* 按钮容器样式 */
  .button-container {
    display: flex;
    justify-content: center;
    margin-bottom: 24px;
  }

  .get-config-btn {
    background: linear-gradient(135deg, #4b6cb7 0%, #182848 100%);
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 6px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .get-config-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
  }

  .get-config-btn:active {
    transform: translateY(1px);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  /* 操作按钮共享样式 */
  .copy-btn,
  .apply-btn {
    padding: 10px 16px;
    border-radius: 4px;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .copy-btn {
    background-color: #4b6cb7;
    color: white;
    border: none;
  }

  .copy-btn:hover {
    background-color: #3b5998;
  }

  .apply-btn {
    background-color: #10b981;
    color: white;
    border: none;
  }

  .apply-btn:hover {
    background-color: #059669;
  }

  /* 消息提示样式 */
  .message-container {
    position: fixed;
    top: 80px;
    left: 0;
    right: 0;
    display: flex;
    justify-content: center;
    z-index: 110;
  }

  .message-box {
    padding: 12px 20px;
    border-radius: 6px;
    font-size: 14px;
    font-weight: 500;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    animation: slide-down 0.3s ease;
  }

  .success {
    background-color: #10b981;
    color: white;
  }

  .warning {
    background-color: #f59e0b;
    color: white;
  }

  .error {
    background-color: #ef4444;
    color: white;
  }

  @keyframes slide-down {
    from {
      transform: translateY(-20px);
      opacity: 0;
    }
    to {
      transform: translateY(0);
      opacity: 1;
    }
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
