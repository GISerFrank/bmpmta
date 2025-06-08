<template>
  <div
      v-if="visible"
      class="danmu-input-container"
      :class="{ 'danmu-input--disabled': disabled }"
  >
    <el-input
        v-model="inputText"
        :placeholder="placeholder"
        @keyup.enter="handleSendText"
        @input="handleInput"
        class="danmu-input"
        :disabled="disabled"
        :maxlength="maxLength"
        show-word-limit
    >
      <!-- 前置表情按钮 -->
      <template #prepend>
        <el-dropdown
            trigger="click"
            placement="top-start"
            :disabled="disabled"
        >
          <el-button
              icon="emoji-smile"
              class="emoji-button"
              :disabled="disabled"
          />
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item
                  v-for="(emoji, index) in quickEmojis"
                  :key="index"
                  @click="handleSendEmoji(emoji)"
                  class="emoji-dropdown-item"
              >
                <span class="emoji-text">{{ emoji }}</span>
                <span class="emoji-shortcut">{{ getEmojiName(emoji) }}</span>
              </el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </template>

      <!-- 后置颜色选择和发送按钮 -->
      <template #append>
        <el-select
            v-model="selectedColor"
            style="width: 80px"
            :disabled="disabled"
            @change="handleColorChange"
        >
          <el-option
              v-for="item in colorOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              :style="{ color: item.value }"
          >
            <span :style="{ color: item.value }">{{ item.label }}</span>
          </el-option>
        </el-select>

        <el-button
            @click="handleSendText"
            type="primary"
            :disabled="disabled || !canSend"
            :loading="sending"
        >
          发送
        </el-button>
      </template>
    </el-input>

    <!-- 发送状态提示 -->
    <div v-if="lastSentTime" class="send-status">
      <span class="send-cooldown">{{ cooldownText }}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DanmuInputSection',

  props: {
    // 显示控制
    visible: {
      type: Boolean,
      default: true
    },
    disabled: {
      type: Boolean,
      default: false
    },

    // 输入配置
    placeholder: {
      type: String,
      default: '发送弹幕评论...'
    },
    maxLength: {
      type: Number,
      default: 100
    },

    // 默认值
    defaultColor: {
      type: String,
      default: '#FFFFFF'
    },

    // 自定义选项
    customColorOptions: {
      type: Array,
      default: undefined
    },
    customQuickEmojis: {
      type: Array,
      default: undefined
    },

    // 冷却时间（毫秒）
    cooldownTime: {
      type: Number,
      default: 2000
    }
  },

  emits: [
    'send-text',     // 发送文本弹幕
    'send-emoji',    // 发送表情弹幕
    'color-change',  // 颜色改变
    'input-change'   // 输入内容改变
  ],

  data() {
    return {
      inputText: '',
      selectedColor: this.defaultColor,
      sending: false,
      lastSentTime: 0,
      cooldownTimer: null,

      // 默认颜色选项
      defaultColorOptions: [
        { value: '#FFFFFF', label: '白色' },
        { value: '#FF0000', label: '红色' },
        { value: '#00FF00', label: '绿色' },
        { value: '#0000FF', label: '蓝色' },
        { value: '#FFFF00', label: '黄色' },
        { value: '#FF00FF', label: '粉色' },
        { value: '#00FFFF', label: '青色' },
        { value: '#FFA500', label: '橙色' }
      ],

      // 默认快速表情
      defaultQuickEmojis: ['😂', '❤️', '👍', '🎬', '🤔', '😱', '🔥', '👏'],

      // 表情名称映射
      emojiNames: {
        '😂': '笑哭',
        '❤️': '爱心',
        '👍': '点赞',
        '🎬': '电影',
        '🤔': '思考',
        '😱': '震惊',
        '🔥': '火爆',
        '👏': '鼓掌'
      }
    }
  },

  computed: {
    // 颜色选项
    colorOptions() {
      return this.customColorOptions || this.defaultColorOptions
    },

    // 快速表情选项
    quickEmojis() {
      return this.customQuickEmojis || this.defaultQuickEmojis
    },

    // 是否可以发送
    canSend() {
      return this.inputText.trim().length > 0 && !this.isInCooldown
    },

    // 是否在冷却时间内
    isInCooldown() {
      if (!this.lastSentTime) return false
      return Date.now() - this.lastSentTime < this.cooldownTime
    },

    // 冷却时间文本
    cooldownText() {
      if (!this.isInCooldown) return ''
      const remaining = Math.ceil((this.cooldownTime - (Date.now() - this.lastSentTime)) / 1000)
      return `${remaining}秒后可再次发送`
    }
  },

  watch: {
    defaultColor: {
      immediate: true,
      handler(newColor) {
        this.selectedColor = newColor
      }
    }
  },

  methods: {
    // 处理文本输入
    handleInput(value) {
      this.$emit('input-change', value)
    },

    // 处理颜色变化
    handleColorChange(color) {
      this.$emit('color-change', color)
    },

    // 发送文本弹幕
    handleSendText() {
      const text = this.inputText.trim()

      if (!text || this.disabled || this.sending || this.isInCooldown) {
        return
      }

      this.sending = true

      const danmuData = {
        content: text,
        color: this.selectedColor,
        type: 'text'
      }

      // 发射事件到父组件
      this.$emit('send-text', danmuData)

      // 清空输入框
      this.inputText = ''

      // 设置冷却时间
      this.startCooldown()

      // 模拟发送延迟
      setTimeout(() => {
        this.sending = false
      }, 300)
    },

    // 发送表情弹幕
    handleSendEmoji(emoji) {
      if (this.disabled || this.isInCooldown) {
        return
      }

      const danmuData = {
        content: emoji,
        color: this.selectedColor,
        type: 'emoji'
      }

      // 发射事件到父组件
      this.$emit('send-emoji', danmuData)

      // 设置冷却时间
      this.startCooldown()
    },

    // 开始冷却计时
    startCooldown() {
      this.lastSentTime = Date.now()

      // 清除之前的计时器
      if (this.cooldownTimer) {
        clearInterval(this.cooldownTimer)
      }

      // 启动新的计时器来更新冷却状态
      this.cooldownTimer = setInterval(() => {
        if (!this.isInCooldown) {
          this.lastSentTime = 0
          clearInterval(this.cooldownTimer)
          this.cooldownTimer = null
        }
      }, 100)
    },

    // 获取表情名称
    getEmojiName(emoji) {
      return this.emojiNames[emoji] || ''
    },

    // 清空输入
    clearInput() {
      this.inputText = ''
    },

    // 设置输入内容
    setInputText(text) {
      this.inputText = text
    },

    // 重置冷却时间
    resetCooldown() {
      this.lastSentTime = 0
      if (this.cooldownTimer) {
        clearInterval(this.cooldownTimer)
        this.cooldownTimer = null
      }
    }
  },

  beforeUnmount() {
    // 清理计时器
    if (this.cooldownTimer) {
      clearInterval(this.cooldownTimer)
    }
  }
}
</script>

<style scoped>
.danmu-input-container {
  margin: 15px 0;
  width: 100%;
  max-width: 800px;
  padding: 12px;
  background-color: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.danmu-input-container:hover {
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.danmu-input-container.danmu-input--disabled {
  opacity: 0.6;
  background-color: rgba(0, 0, 0, 0.05);
}

.danmu-input :deep(.el-input__wrapper) {
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.danmu-input :deep(.el-input__wrapper):hover {
  border-color: #409EFF;
}

.danmu-input :deep(.el-input__wrapper.is-focus) {
  border-color: #409EFF;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
}

.emoji-button {
  padding: 0 12px;
  border: none;
  background: transparent;
  color: #606266;
  transition: all 0.3s ease;
}

.emoji-button:hover {
  color: #409EFF;
  background-color: rgba(64, 158, 255, 0.1);
}

.emoji-dropdown-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 16px;
}

.emoji-text {
  font-size: 18px;
  margin-right: 8px;
}

.emoji-shortcut {
  font-size: 12px;
  color: #909399;
}

.send-status {
  margin-top: 8px;
  text-align: center;
}

.send-cooldown {
  font-size: 12px;
  color: #E6A23C;
  background-color: rgba(230, 162, 60, 0.1);
  padding: 4px 8px;
  border-radius: 4px;
  display: inline-block;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .danmu-input-container {
    margin: 10px;
    padding: 8px;
    max-width: calc(100% - 20px);
  }

  .emoji-text {
    font-size: 16px;
  }

  .emoji-shortcut {
    display: none;
  }
}

/* 深色主题支持 */
@media (prefers-color-scheme: dark) {
  .danmu-input-container {
    background-color: rgba(0, 0, 0, 0.8);
    border-color: rgba(255, 255, 255, 0.1);
  }

  .danmu-input :deep(.el-input__wrapper) {
    background-color: rgba(255, 255, 255, 0.1);
    border-color: rgba(255, 255, 255, 0.2);
  }

  .emoji-button {
    color: #C0C4CC;
  }

  .emoji-button:hover {
    color: #409EFF;
  }
}

/* 动画效果 */
.danmu-input-container {
  animation: slideInUp 0.3s ease-out;
}

@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 发送按钮加载状态 */
.danmu-input :deep(.el-button.is-loading) {
  position: relative;
}

.danmu-input :deep(.el-button.is-loading::after) {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 16px;
  height: 16px;
  border: 2px solid transparent;
  border-top: 2px solid currentColor;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  transform: translate(-50%, -50%);
}

@keyframes spin {
  0% { transform: translate(-50%, -50%) rotate(0deg); }
  100% { transform: translate(-50%, -50%) rotate(360deg); }
}
</style>