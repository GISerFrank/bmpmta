<template>
  <div class="h-screen flex flex-col bg-gradient-to-br from-slate-50 to-blue-50">
    <!-- 顶部导航栏 -->
    <header class="relative overflow-hidden bg-white/80 backdrop-blur-xl border-b border-slate-200/60 shadow-sm">
      <div class="absolute inset-0 bg-gradient-to-r from-blue-500/5 to-purple-500/5"></div>
      <div class="relative flex items-center justify-between p-4">
        <div class="flex items-center space-x-3">
          <div class="w-10 h-10 bg-gradient-to-br from-blue-500 to-purple-600 rounded-xl flex items-center justify-center">
            <div class="sparkles-icon">✨</div>
          </div>
          <div>
            <h1 class="text-xl font-bold bg-gradient-to-r from-gray-900 to-gray-600 bg-clip-text text-transparent">
              AI Chat Assistant
            </h1>
            <p class="text-xs text-gray-500">智能对话，随时随地</p>
          </div>
        </div>

        <div class="flex items-center space-x-2">
          <select
              v-model="model"
              class="px-3 py-2 bg-white/80 border border-slate-200 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all"
          >
            <option v-for="option in availableModels" :key="option.value" :value="option.value">
              {{ option.label }}
            </option>
          </select>

          <button
              @click="showApiKeyModal = true"
              class="p-2 bg-white/80 hover:bg-white rounded-lg border border-slate-200 transition-colors"
          >
            <div class="settings-icon">⚙️</div>
          </button>

          <button
              @click="startNewChat"
              class="flex items-center space-x-2 px-4 py-2 bg-gradient-to-r from-blue-500 to-purple-600 text-white rounded-lg hover:shadow-lg transition-all duration-300 hover:scale-105"
          >
            <div class="plus-icon">➕</div>
            <span class="text-sm">新对话</span>
          </button>
        </div>
      </div>
    </header>

    <!-- 聊天窗口 -->
    <div
        ref="chatContainer"
        class="flex-1 overflow-y-auto p-4 space-y-6"
    >
      <div
          v-for="msg in messages"
          :key="msg.id"
          :class="[
          'flex items-start space-x-3 max-w-4xl mx-auto',
          msg.role === 'user' ? 'flex-row-reverse space-x-reverse' : ''
        ]"
      >
        <!-- 头像 -->
        <div :class="[
          'w-10 h-10 rounded-full flex items-center justify-center flex-shrink-0',
          msg.role === 'user'
            ? 'bg-gradient-to-br from-blue-500 to-blue-600'
            : 'bg-gradient-to-br from-emerald-500 to-emerald-600'
        ]">
          <div class="avatar-icon">{{ msg.role === 'user' ? '👤' : '🤖' }}</div>
        </div>

        <!-- 消息内容 -->
        <div :class="[
          'group relative max-w-2xl',
          msg.role === 'user' ? 'ml-auto' : 'mr-auto'
        ]">
          <div :class="[
            'relative px-4 py-3 rounded-2xl shadow-sm',
            msg.role === 'user'
              ? 'bg-gradient-to-br from-blue-500 to-blue-600 text-white'
              : 'bg-white border border-slate-200'
          ]">
            <div class="text-sm leading-relaxed whitespace-pre-wrap">
              {{ msg.content }}
            </div>

            <!-- 时间戳 -->
            <div :class="[
              'text-xs mt-2',
              msg.role === 'user' ? 'text-blue-100' : 'text-gray-400'
            ]">
              {{ formatTime(msg.timestamp) }}
            </div>
          </div>

          <!-- 操作按钮 -->
          <div v-if="msg.role === 'assistant'" class="absolute -bottom-8 left-0 flex items-center space-x-1 opacity-0 group-hover:opacity-100 transition-opacity">
            <button
                @click="copyMessage(msg.content)"
                class="p-1.5 bg-white border border-slate-200 rounded-lg hover:bg-gray-50 transition-colors"
                title="复制"
            >
              📋
            </button>
            <button
                class="p-1.5 bg-white border border-slate-200 rounded-lg hover:bg-gray-50 transition-colors"
                title="点赞"
            >
              👍
            </button>
            <button
                class="p-1.5 bg-white border border-slate-200 rounded-lg hover:bg-gray-50 transition-colors"
                title="点踩"
            >
              👎
            </button>
          </div>
        </div>
      </div>

      <!-- 打字指示器 -->
      <div v-if="isLoading || isTyping" class="flex items-start space-x-3 max-w-4xl mx-auto">
        <div class="w-10 h-10 rounded-full bg-gradient-to-br from-emerald-500 to-emerald-600 flex items-center justify-center">
          🤖
        </div>
        <div class="bg-white border border-slate-200 rounded-2xl px-4 py-3 shadow-sm">
          <div class="flex items-center space-x-1">
            <div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce"></div>
            <div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 0.2s"></div>
            <div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 0.4s"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- 输入区域 -->
    <footer class="p-4 bg-white/80 backdrop-blur-xl border-t border-slate-200/60">
      <div class="max-w-4xl mx-auto">
        <div class="relative bg-white rounded-2xl border border-slate-200 shadow-lg">
          <textarea
              ref="textareaRef"
              v-model="userInput"
              @keydown="handleKeyDown"
              @input="adjustTextareaHeight"
              placeholder="输入你的问题..."
              class="w-full px-4 py-3 pr-24 resize-none border-0 focus:outline-none bg-transparent rounded-2xl"
              rows="1"
              :disabled="isLoading"
          />

          <!-- 底部工具栏 -->
          <div class="flex items-center justify-between px-4 py-2 border-t border-slate-100">
            <div class="flex items-center space-x-2">
              <button class="p-1.5 hover:bg-gray-100 rounded-lg transition-colors" title="附件">
                📎
              </button>
              <button class="p-1.5 hover:bg-gray-100 rounded-lg transition-colors" title="语音">
                🎤
              </button>
              <span class="text-xs text-gray-400">
                {{ userInput.length }}/4000
              </span>
            </div>

            <button
                @click="sendMessage"
                :disabled="isLoading || !userInput.trim() || !hasSetApiKey"
                class="flex items-center space-x-2 px-4 py-2 bg-gradient-to-r from-blue-500 to-purple-600 text-white rounded-lg hover:shadow-lg transition-all duration-300 disabled:opacity-50 disabled:cursor-not-allowed hover:scale-105"
            >
              <div>📤</div>
              <span class="text-sm">发送</span>
            </button>
          </div>
        </div>

        <p class="mt-2 text-xs text-center text-gray-500">
          ChatGPT 可能会生成不准确的信息。按 Enter 发送，Shift+Enter 换行
        </p>
      </div>
    </footer>

    <!-- API Key 设置模态框 -->
    <div v-if="showApiKeyModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50">
      <div class="bg-white rounded-2xl p-6 w-full max-w-md mx-4 shadow-2xl">
        <h3 class="text-lg font-semibold mb-4">设置 OpenAI API Key</h3>
        <input
            type="password"
            v-model="apiKey"
            placeholder="sk-..."
            class="w-full px-4 py-3 border border-slate-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 mb-4"
        />
        <p class="text-sm text-gray-600 mb-4">
          API Key 将安全地存储在你的浏览器本地存储中，不会发送到任何服务器。
        </p>
        <div class="flex space-x-3">
          <button
              @click="showApiKeyModal = false"
              class="flex-1 px-4 py-2 border border-slate-200 rounded-lg hover:bg-gray-50 transition-colors"
          >
            取消
          </button>
          <button
              @click="handleSetApiKey"
              class="flex-1 px-4 py-2 bg-gradient-to-r from-blue-500 to-purple-600 text-white rounded-lg hover:shadow-lg transition-all"
          >
            保存
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref, nextTick, onMounted, watch} from 'vue';

// 响应式数据
const messages = ref([
  {
    id: 1,
    role: "assistant",
    content: "你好！我是基于 OpenAI 的 AI 助手，有什么我可以帮助你的？",
    timestamp: new Date()
  }
]);

const userInput = ref("");
const isLoading = ref(false);
const apiKey = ref("");
const hasSetApiKey = ref(false);
const showApiKeyModal = ref(false);
const model = ref("gpt-4o");
const isTyping = ref(false);

// DOM引用
const chatContainer = ref(null);
const textareaRef = ref(null);

const availableModels = [
  {value: "gpt-4o", label: "GPT-4o", description: "最新最强模型"},
  {value: "gpt-4-turbo", label: "GPT-4 Turbo", description: "平衡性能与速度"},
  {value: "gpt-3.5-turbo", label: "GPT-3.5 Turbo", description: "快速响应"}
];

// 自动调整输入框高度
const adjustTextareaHeight = () => {
  const textarea = textareaRef.value;
  if (textarea) {
    textarea.style.height = 'auto';
    textarea.style.height = Math.min(textarea.scrollHeight, 120) + 'px';
  }
};

// 滚动到底部
const scrollToBottom = () => {
  setTimeout(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
    }
  }, 100);
};

// 监听消息变化，自动滚动
watch([messages, isLoading], () => {
  scrollToBottom();
}, {deep: true});

// 监听输入变化，自动调整高度
watch(userInput, () => {
  nextTick(() => {
    adjustTextareaHeight();
  });
});

// 模拟API调用
const sendToOpenAI = async (_messagesData) => {
  // 模拟网络延迟
  await new Promise(resolve => setTimeout(resolve, 1000 + Math.random() * 2000));

  const responses = [
    "这是一个很好的问题！让我来为你详细解答...",
    "根据你的描述，我建议以下几个方案：\n\n1. 首先考虑...\n2. 其次可以...\n3. 最后建议...",
    "我理解你的需求。这个问题确实比较复杂，需要从多个角度来分析。",
    "很有趣的想法！我们可以从以下几个方面来探讨这个话题...",
    "让我整理一下思路来回答你的问题。首先，我们需要明确几个关键点..."
  ];

  return responses[Math.floor(Math.random() * responses.length)];
};

// 发送消息
const sendMessage = async () => {
  if (!userInput.value.trim() || isLoading.value) return;

  if (!hasSetApiKey.value) {
    showApiKeyModal.value = true;
    return;
  }

  const userMessage = {
    id: Date.now(),
    role: "user",
    content: userInput.value.trim(),
    timestamp: new Date()
  };

  messages.value.push(userMessage);
  userInput.value = "";
  isLoading.value = true;
  isTyping.value = true;

  try {
    const response = await sendToOpenAI([...messages.value, userMessage]);

    isTyping.value = false;

    // 模拟打字效果
    const assistantMessage = {
      id: Date.now() + 1,
      role: "assistant",
      content: "",
      timestamp: new Date()
    };

    messages.value.push(assistantMessage);

    // 逐字显示效果
    for (let i = 0; i <= response.length; i++) {
      await new Promise(resolve => setTimeout(resolve, 20));
      const msgIndex = messages.value.findIndex(msg => msg.id === assistantMessage.id);
      if (msgIndex !== -1) {
        messages.value[msgIndex].content = response.slice(0, i);
      }
    }

  } catch (error) {
    isTyping.value = false;
    messages.value.push({
      id: Date.now() + 1,
      role: "assistant",
      content: `抱歉，发生了错误：${error.message}`,
      timestamp: new Date()
    });
  } finally {
    isLoading.value = false;
  }
};

// 复制消息
const copyMessage = async (content) => {
  try {
    await navigator.clipboard.writeText(content);
    // 这里可以添加toast提示
    console.log('消息已复制到剪贴板');
  } catch (err) {
    console.error('复制失败:', err);
  }
};

// 开始新对话
const startNewChat = () => {
  if (confirm('确定要开始新的对话吗？当前对话将被清除。')) {
    messages.value = [{
      id: 1,
      role: "assistant",
      content: "你好！我是基于 OpenAI 的 AI 助手，有什么我可以帮助你的？",
      timestamp: new Date()
    }];
  }
};

// 格式化时间
const formatTime = (date) => {
  return date.toLocaleTimeString('zh-CN', {
    hour: '2-digit',
    minute: '2-digit'
  });
};

// 设置API Key
const handleSetApiKey = () => {
  if (apiKey.value.trim().startsWith("sk-")) {
    hasSetApiKey.value = true;
    showApiKeyModal.value = false;
    localStorage.setItem("openai_api_key", apiKey.value);
  } else {
    alert("请输入有效的 OpenAI API Key");
  }
};

// 处理键盘事件
const handleKeyDown = (e) => {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
};

// 检查本地存储的API Key
onMounted(() => {
  const savedApiKey = localStorage.getItem("openai_api_key");
  if (savedApiKey) {
    apiKey.value = savedApiKey;
    hasSetApiKey.value = true;
  }
});
</script>

<style scoped>
/* 基础布局 */
.h-screen {
  height: 100vh;
}

.flex {
  display: flex;
}

.flex-col {
  flex-direction: column;
}

.flex-1 {
  flex: 1;
}

.items-center {
  align-items: center;
}

.items-start {
  align-items: flex-start;
}

.justify-between {
  justify-content: space-between;
}

.justify-center {
  justify-content: center;
}

.relative {
  position: relative;
}

.absolute {
  position: absolute;
}

.fixed {
  position: fixed;
}

.inset-0 {
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

.z-50 {
  z-index: 50;
}

/* 间距 */
.p-4 {
  padding: 1rem;
}

.p-6 {
  padding: 1.5rem;
}

.px-3 {
  padding-left: 0.75rem;
  padding-right: 0.75rem;
}

.px-4 {
  padding-left: 1rem;
  padding-right: 1rem;
}

.py-2 {
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}

.py-3 {
  padding-top: 0.75rem;
  padding-bottom: 0.75rem;
}

.m-0 {
  margin: 0;
}

.mt-2 {
  margin-top: 0.5rem;
}

.mb-4 {
  margin-bottom: 1rem;
}

.ml-auto {
  margin-left: auto;
}

.mr-auto {
  margin-right: auto;
}

.mx-auto {
  margin-left: auto;
  margin-right: auto;
}

.space-x-2 > * + * {
  margin-left: 0.5rem;
}

.space-x-3 > * + * {
  margin-left: 0.75rem;
}

.space-y-6 > * + * {
  margin-top: 1.5rem;
}

/* 尺寸 */
.w-2 {
  width: 0.5rem;
}

.w-4 {
  width: 1rem;
}

.w-10 {
  width: 2.5rem;
}

.w-full {
  width: 100%;
}

.h-2 {
  height: 0.5rem;
}

.h-4 {
  height: 1rem;
}

.h-10 {
  height: 2.5rem;
}

.max-w-md {
  max-width: 28rem;
}

.max-w-2xl {
  max-width: 42rem;
}

.max-w-4xl {
  max-width: 56rem;
}

/* 背景和颜色 */
.bg-gradient-to-br {
  background: linear-gradient(135deg, #f8fafc 0%, #eff6ff 100%);
}

.bg-gradient-to-r {
  background: linear-gradient(90deg, var(--gradient-from), var(--gradient-to));
}

.from-blue-500 {
  --gradient-from: #3b82f6;
}

.to-purple-600 {
  --gradient-to: #9333ea;
}

.from-emerald-500 {
  --gradient-from: #10b981;
}

.to-emerald-600 {
  --gradient-to: #059669;
}

.from-blue-500.to-blue-600 {
  background: linear-gradient(135deg, #3b82f6, #2563eb);
}

.from-emerald-500.to-emerald-600 {
  background: linear-gradient(135deg, #10b981, #059669);
}

.from-blue-500.to-purple-600 {
  background: linear-gradient(90deg, #3b82f6, #9333ea);
}

.bg-white {
  background-color: white;
}

.bg-gray-50 {
  background-color: #f9fafb;
}

.text-white {
  color: white;
}

.text-gray-400 {
  color: #9ca3af;
}

.text-gray-500 {
  color: #6b7280;
}

.text-gray-600 {
  color: #4b5563;
}

.text-gray-900 {
  color: #111827;
}

.text-blue-100 {
  color: #dbeafe;
}

/* 透明度和模糊 */
.bg-white\/80 {
  background-color: rgba(255, 255, 255, 0.8);
}

.bg-black\/50 {
  background-color: rgba(0, 0, 0, 0.5);
}

.backdrop-blur-xl {
  backdrop-filter: blur(24px);
}

.backdrop-blur-sm {
  backdrop-filter: blur(4px);
}

.opacity-0 {
  opacity: 0;
}

.opacity-50 {
  opacity: 0.5;
}

/* 边框和圆角 */
.border {
  border-width: 1px;
  border-style: solid;
}

.border-0 {
  border-width: 0;
}

.border-t {
  border-top-width: 1px;
  border-top-style: solid;
}

.border-b {
  border-bottom-width: 1px;
  border-bottom-style: solid;
}

.border-slate-100 {
  border-color: #f1f5f9;
}

.border-slate-200 {
  border-color: #e2e8f0;
}

.border-slate-200\/60 {
  border-color: rgba(226, 232, 240, 0.6);
}

.rounded-lg {
  border-radius: 0.5rem;
}

.rounded-xl {
  border-radius: 0.75rem;
}

.rounded-2xl {
  border-radius: 1rem;
}

.rounded-full {
  border-radius: 9999px;
}

/* 阴影 */
.shadow-sm {
  box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
}

.shadow-lg {
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

.shadow-2xl {
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
}

/* 文字 */
.text-xs {
  font-size: 0.75rem;
  line-height: 1rem;
}

.text-sm {
  font-size: 0.875rem;
  line-height: 1.25rem;
}

.text-lg {
  font-size: 1.125rem;
  line-height: 1.75rem;
}

.text-xl {
  font-size: 1.25rem;
  line-height: 1.75rem;
}

.font-semibold {
  font-weight: 600;
}

.font-bold {
  font-weight: 700;
}

.leading-relaxed {
  line-height: 1.625;
}

.whitespace-pre-wrap {
  white-space: pre-wrap;
}

.text-center {
  text-align: center;
}

/* 渐变文字 */
.bg-clip-text {
  background-clip: text;
  -webkit-background-clip: text;
}

.text-transparent {
  color: transparent;
}

.from-gray-900.to-gray-600 {
  background: linear-gradient(90deg, #111827, #4b5563);
}

/* 溢出和滚动 */
.overflow-hidden {
  overflow: hidden;
}

.overflow-y-auto {
  overflow-y: auto;
}

/* 光标 */
.cursor-not-allowed {
  cursor: not-allowed;
}

.resize-none {
  resize: none;
}

/* 焦点 */
.focus\:outline-none:focus {
  outline: none;
}

.focus\:ring-2:focus {
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.5);
}

.focus\:ring-blue-500:focus {
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.5);
}

/* 过渡和变换 */
.transition-all {
  transition: all 0.3s ease;
}

.transition-colors {
  transition: color 0.3s ease, background-color 0.3s ease;
}

.transition-opacity {
  transition: opacity 0.3s ease;
}

.duration-300 {
  transition-duration: 300ms;
}

/* 悬停效果 */
.hover\:bg-white:hover {
  background-color: white;
}

.hover\:bg-gray-50:hover {
  background-color: #f9fafb;
}

.hover\:bg-gray-100:hover {
  background-color: #f3f4f6;
}

.hover\:shadow-lg:hover {
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

.hover\:scale-105:hover {
  transform: scale(1.05);
}

/* 禁用状态 */
.disabled\:opacity-50:disabled {
  opacity: 0.5;
}

.disabled\:cursor-not-allowed:disabled {
  cursor: not-allowed;
}

/* 动画 */
.animate-bounce {
  animation: bounce 1s infinite;
}

@keyframes bounce {
  0%, 20%, 53%, 80%, 100% {
    transform: translateY(0);
  }
  40%, 43% {
    transform: translateY(-10px);
  }
  70% {
    transform: translateY(-5px);
  }
}

/* Group hover效果 */
.group:hover .group-hover\:opacity-100 {
  opacity: 1;
}

/* 自定义样式 */
.sparkles-icon, .avatar-icon {
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 自定义滚动条 */
.overflow-y-auto::-webkit-scrollbar {
  width: 6px;
}

.overflow-y-auto::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}

/* 输入框样式 */
textarea {
  min-height: 48px;
  max-height: 120px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .max-w-4xl {
    max-width: 100%;
  }

  .px-4 {
    padding-left: 1rem;
    padding-right: 1rem;
  }

  .space-x-3 > * + * {
    margin-left: 0.75rem;
  }
}
</style>