<!--
 * @Version    : v1.00
 * @Author     : Wang Chao
 * @Date       : 2025-04-24 00:28
 * @LastAuthor : Wang Chao
 * @LastTime   : 2025-04-24 16:09
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
        <div class="json-panel left-panel">
          <div class="panel-header">
            <h3>配置工作区</h3>
            <p class="panel-description">在此编辑和查看MCP完整配置</p>

            <!-- 添加服务器选择器，使用Element-plus -->
            <div
              class="server-selector-container"
              v-if="serverList.length > 0"
            >
              <span class="server-selector-label">服务器列表：</span>
              <el-select
                v-model="selectedServer"
                filterable
                placeholder="搜索服务器..."
                class="server-select"
                @change="selectServer"
              >
                <el-option
                  v-for="server in serverList"
                  :key="server"
                  :label="server"
                  :value="server"
                />
              </el-select>
            </div>
          </div>
          <codemirror
            v-model="originalJson"
            :extensions="extensions"
            :indent-with-tab="true"
            :tab-size="2"
            placeholder="请输入MCP配置JSON"
            class="editor"
            style="height: calc(100% - 10px); margin-bottom: 10px"
            ref="jsonEditor"
          />
        </div>

        <div class="right-side">
          <div class="json-panel right-panel">
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
              style="height: calc(100% - 10px); margin-bottom: 10px"
            />
          </div>

          <div class="button-container">
            <button
              class="get-config-btn"
              @click="getMergedConfig"
            >
              合并配置
            </button>
            <button
              v-if="addedServerNames.length > 0"
              class="copy-btn"
              @click="copyToClipboard"
            >
              复制到剪贴板
            </button>
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
  import { ref, onMounted, computed, watch, onUnmounted } from 'vue';
  import { Codemirror } from 'vue-codemirror';
  import { json } from '@codemirror/lang-json';
  import { oneDark } from '@codemirror/theme-one-dark';
  import { basicSetup } from 'codemirror';
  // 引入Element-plus组件
  import { ElSelect, ElOption } from 'element-plus';

  const originalJson = ref('');
  const newJson = ref('');
  const message = ref('');
  const showMessage = ref(false);
  const messageType = ref('success');
  const addedServerNames = ref([]);
  const serverList = ref([]);
  const selectedServer = ref('');

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

  // 解析JSON获取服务器列表
  const parseServerList = () => {
    try {
      const jsonObj = JSON.parse(originalJson.value);
      if (jsonObj && jsonObj.mcpServers) {
        serverList.value = Object.keys(jsonObj.mcpServers);
      } else {
        serverList.value = [];
      }
    } catch (error) {
      console.error('解析JSON出错:', error);
      serverList.value = [];
    }
  };

  // 监听JSON变化，更新服务器列表
  watch(originalJson, () => {
    parseServerList();
  });

  // 选择服务器，高亮显示
  const selectServer = (server) => {
    if (!server) return;

    // 尝试查找并滚动到选中服务器的位置
    try {
      const searchText = `"${server}"`;
      const content = originalJson.value;
      const index = content.indexOf(searchText);

      if (index !== -1) {
        // 计算行号（粗略估计）
        const beforeText = content.substring(0, index);
        const lineCount = beforeText.split('\n').length - 1;

        // 获取编辑器元素并滚动
        const editorEl = document.querySelector('.editor .cm-scroller');
        if (editorEl) {
          // 估计每行高度，滚动到大致位置
          const lineHeight = 20; // 估计行高
          const scrollPosition = lineCount * lineHeight;

          // 滚动到位置
          editorEl.scrollTop = scrollPosition;

          // 添加闪烁高亮效果（可选）
          setTimeout(() => {
            // 尝试找到包含服务器名称的行
            const lines = document.querySelectorAll('.editor .cm-line');
            for (let i = 0; i < lines.length; i++) {
              if (lines[i].textContent.includes(searchText)) {
                // 添加临时高亮类
                lines[i].classList.add('highlight-line');

                // 2秒后移除高亮
                setTimeout(() => {
                  lines[i].classList.remove('highlight-line');
                }, 2000);

                break;
              }
            }
          }, 100);
        }
      }
    } catch (error) {
      console.error('滚动到服务器位置出错:', error);
    }
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

  // 复制到剪贴板
  const copyToClipboard = () => {
    navigator.clipboard
      .writeText(originalJson.value)
      .then(() => {
        showMessageTip('配置已复制到剪贴板！');
      })
      .catch((err) => {
        console.error('复制到剪贴板失败:', err);
        showMessageTip('复制到剪贴板失败，请手动复制。', 'warning');
      });
  };

  // 合并JSON并直接更新到原始配置
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

      // 记录新增的服务器名称（为将来功能保留）
      addedServerNames.value = [];

      // 将新配置添加到原始配置中
      for (const [serverName, serverConfig] of newServerEntries) {
        original.mcpServers[serverName] = serverConfig;
        addedServerNames.value.push(serverName);
      }

      // 格式化结果并直接更新原始配置
      originalJson.value = JSON.stringify(original, null, 2);

      // 提示成功
      showMessageTip('配置已成功合并并更新！');
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

    // 清空已添加的服务器记录
    addedServerNames.value = [];

    // 初始化解析服务器列表
    parseServerList();
  });

  onUnmounted(() => {
    // 移除事件监听
    document.removeEventListener('click', closeDropdownOnClickOutside);
  });
</script>

<style scoped>
  /* 全局样式 */
  .app-container {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    height: 100vh;
    background-color: #f5f7fa;
    font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
    overflow: hidden;
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
    overflow: hidden;
  }

  .editors-container {
    display: flex;
    gap: 24px;
    height: calc(100vh - 240px);
    min-height: 450px;
    margin-bottom: 24px;
    overflow: hidden;
  }

  @media (max-width: 768px) {
    .editors-container {
      flex-direction: column;
      height: auto;
    }
  }

  /* JSON面板样式 */
  .json-panel {
    display: flex;
    flex-direction: column;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
    overflow: hidden;
    transition: all 0.3s ease;
    position: relative;
    padding-bottom: 10px;
  }

  /* 左侧面板样式 */
  .left-panel {
    flex: 3; /* 左侧面板占更多空间 */
    max-height: calc(100vh - 240px);
  }

  /* 右侧整体样式 */
  .right-side {
    flex: 2;
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  /* 右侧面板样式 */
  .right-panel {
    flex: 1;
    max-height: calc(100vh - 340px); /* 比左侧面板矮一些，给按钮留出空间 */
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

  /* 编辑器样式 */
  .editor {
    flex: 1;
    overflow: auto;
    font-family: 'Fira Code', 'Consolas', monospace;
    display: flex;
    flex-direction: column;
  }

  :deep(.cm-editor) {
    height: 100%;
    flex: 1;
    font-size: 14px;
    overflow: visible;
  }

  :deep(.cm-scroller) {
    overflow: auto !important;
    padding: 8px 0;
    min-height: 300px;
    padding-bottom: 40px;
  }

  :deep(.cm-content) {
    padding-bottom: 40px;
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
    gap: 16px;
    margin-top: 8px;
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

  /* 复制按钮样式 */
  .copy-btn {
    background-color: #10b981;
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 6px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    animation: appear 0.3s ease;
  }

  .copy-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
  }

  .copy-btn:active {
    transform: translateY(1px);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  @keyframes appear {
    from {
      opacity: 0;
      transform: scale(0.9);
    }
    to {
      opacity: 1;
      transform: scale(1);
    }
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

  /* 服务器选择器样式 - 使用Element-plus */
  .server-selector-container {
    display: flex;
    align-items: center;
    margin: 12px 0;
  }

  .server-selector-label {
    font-size: 14px;
    color: #555;
    margin-right: 8px;
    font-weight: 500;
  }

  .server-select {
    width: 240px;
  }

  /* 高亮样式 */
  .highlight-line {
    background-color: rgba(16, 185, 129, 0.2);
    animation: flash-highlight 2s ease;
  }

  @keyframes flash-highlight {
    0%,
    100% {
      background-color: rgba(16, 185, 129, 0.2);
    }
    50% {
      background-color: rgba(16, 185, 129, 0.4);
    }
  }
</style>
