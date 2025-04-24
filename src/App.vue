<!--
 * @Version    : v1.00
 * @Author     : Wang Chao
 * @Date       : 2025-04-24 00:28
 * @LastAuthor : Wang Chao
 * @LastTime   : 2025-04-24 19:19
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

            <!-- 添加服务器选择器和删除功能 -->
            <div
              class="server-manager-container"
              v-if="serverList.length > 0"
            >
              <div class="server-selector-container">
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
                  >
                    <div class="server-option">
                      <el-checkbox
                        v-model="selectedServers"
                        :label="server"
                        @click.stop
                      ></el-checkbox>
                    </div>
                  </el-option>
                </el-select>
              </div>

              <el-button
                type="danger"
                class="delete-btn"
                :disabled="selectedServers.length === 0"
                @click="deleteSelectedServers"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  viewBox="0 0 24 24"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                >
                  <path d="M3 6h18"></path>
                  <path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"></path>
                  <path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"></path>
                </svg>
                删除已选
              </el-button>
            </div>
          </div>
          <div class="editor-container">
            <div
              class="copy-icon"
              @click="copyOriginalToClipboard"
              title="复制到剪贴板"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="16"
                height="16"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <rect
                  x="9"
                  y="9"
                  width="13"
                  height="13"
                  rx="2"
                  ry="2"
                ></rect>
                <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
              </svg>
            </div>
            <codemirror
              v-model="originalJson"
              :extensions="extensions"
              :indent-with-tab="true"
              :tab-size="2"
              placeholder="请输入MCP配置JSON"
              class="editor"
              ref="jsonEditor"
            />
          </div>
        </div>

        <div class="right-side">
          <div class="json-panel right-panel">
            <div class="panel-header">
              <h3>新增配置</h3>
              <p class="panel-description">请在此输入需要新增的服务器配置</p>
            </div>
            <div class="editor-container">
              <div
                class="copy-icon"
                @click="copyNewToClipboard"
                title="复制到剪贴板"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  viewBox="0 0 24 24"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                >
                  <rect
                    x="9"
                    y="9"
                    width="13"
                    height="13"
                    rx="2"
                    ry="2"
                  ></rect>
                  <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
                </svg>
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
  import { ElSelect, ElOption, ElCheckbox, ElButton } from 'element-plus';

  const originalJson = ref('');
  const newJson = ref('');
  const message = ref('');
  const showMessage = ref(false);
  const messageType = ref('success');
  const addedServerNames = ref([]);
  const serverList = ref([]);
  const selectedServer = ref('');
  const selectedServers = ref([]);

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

  // 删除选中的服务器
  const deleteSelectedServers = () => {
    try {
      if (selectedServers.value.length === 0) {
        return;
      }

      let jsonObj = JSON.parse(originalJson.value);

      // 确保有mcpServers对象
      if (!jsonObj.mcpServers) {
        showMessageTip('未找到mcpServers配置', 'error');
        return;
      }

      // 删除选中的服务器
      selectedServers.value.forEach((server) => {
        if (jsonObj.mcpServers[server]) {
          delete jsonObj.mcpServers[server];
        }
      });

      // 更新JSON
      originalJson.value = JSON.stringify(jsonObj, null, 2);

      // 清空选中状态
      selectedServers.value = [];

      // 提示成功
      showMessageTip(`已删除 ${selectedServers.value.length} 个服务器配置`);

      // 更新服务器列表
      parseServerList();
    } catch (error) {
      console.error('删除服务器配置出错:', error);
      showMessageTip(`删除失败: ${error.message}`, 'error');
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

  // 复制左侧编辑器内容到剪贴板
  const copyOriginalToClipboard = () => {
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

  // 复制右侧编辑器内容到剪贴板
  const copyNewToClipboard = () => {
    navigator.clipboard
      .writeText(newJson.value)
      .then(() => {
        showMessageTip('新增配置已复制到剪贴板！');
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
  }

  /* 左侧面板样式 */
  .left-panel {
    flex: 3; /* 左侧面板占更多空间 */
    max-height: calc(100vh - 240px);
    display: flex;
    flex-direction: column;
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
    display: flex;
    flex-direction: column;
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

  /* 服务器管理容器 */
  .server-manager-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin: 12px 0;
  }

  /* 服务器选择器样式 */
  .server-selector-container {
    display: flex;
    align-items: center;
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

  /* 删除按钮样式 */
  .delete-btn {
    display: flex;
    align-items: center;
    gap: 6px;
    margin-left: 10px;
    height: 36px;
  }

  .delete-btn svg {
    width: 16px;
    height: 16px;
  }

  /* 服务器选项样式 */
  .server-option {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  /* 编辑器容器样式 */
  .editor-container {
    position: relative;
    flex: 1;
    display: flex;
    flex-direction: column;
    height: 100%;
    overflow: hidden;
  }

  /* 复制图标样式 */
  .copy-icon {
    position: absolute;
    top: 8px;
    right: 8px;
    width: 28px;
    height: 28px;
    background-color: rgba(255, 255, 255, 0.9);
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    color: #4b6cb7;
    z-index: 20;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: all 0.2s ease;
  }

  .copy-icon:hover {
    background-color: #4b6cb7;
    color: white;
    transform: scale(1.1);
  }

  .copy-icon:active {
    transform: scale(0.95);
  }

  /* 编辑器样式 */
  .editor {
    flex: 1;
    overflow: auto;
    font-family: 'Fira Code', 'Consolas', monospace;
    display: flex;
    flex-direction: column;
    height: 100%;
    padding-bottom: 0;
    margin-bottom: 0;
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
    padding-bottom: 0;
  }

  :deep(.cm-content) {
    padding-bottom: 0;
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

  .codemirror {
    margin-bottom: 0 !important;
  }
</style>
