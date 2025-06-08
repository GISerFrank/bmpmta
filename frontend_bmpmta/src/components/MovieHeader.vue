<template>
  <el-header class="header">
    <!-- 表情工具栏 - 移到中间 -->
    <div class="emoji-toolbar">
      <el-dropdown
          v-for="(emoji, index) in emojiList"
          :key="index"
          placement="bottom-start"
          @command="handleEmojiAction"
          :disabled="disabled"
      >
        <el-button
            class="emoji-button"
            :class="{ 'emoji-button--disabled': disabled }"
        >
          <el-image
              class="emoji-image"
              :src="emoji.image"
              fit="contain"
              :alt="emoji.name"
          />
        </el-button>

        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item
                v-for="(action, actionIndex) in emoji.actions"
                :key="actionIndex"
                :command="{emoji: emoji.name, action: action}"
            >
              {{ action }}
            </el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </el-header>
</template>

<script>
// 导入表情图片
import laughImage from '@/assets/film/comment_emoji/laugh.png'
import angryImage from '@/assets/film/comment_emoji/angry.png'
import shockedImage from '@/assets/film/comment_emoji/shocked.png'
import thinkImage from '@/assets/film/comment_emoji/think.png'
import touchingImage from '@/assets/film/comment_emoji/touching.png'
import reliefImage from '@/assets/film/comment_emoji/relief.png'
import boringImage from '@/assets/film/comment_emoji/boring.png'

export default {
  name: 'MovieHeader',

  props: {
    disabled: {
      type: Boolean,
      default: false
    },
    customEmojiList: {
      type: Array,
      default: undefined
    }
  },

  emits: ['emoji-action'],

  data() {
    return {
      // 默认表情列表
      defaultEmojiList: [
        {
          name: 'laugh',
          image: laughImage,
          actions: ['发送表情', '收藏这一幕', '分享这一幕']
        },
        {
          name: 'angry',
          image: angryImage,
          actions: ['发送表情', '收藏这一幕', '分享这一幕']
        },
        {
          name: 'boring',
          image: boringImage,
          actions: ['发送表情', '收藏这一幕', '分享这一幕']
        },
        {
          name: 'relief',
          image: reliefImage,
          actions: ['发送表情', '收藏这一幕', '分享这一幕']
        },
        {
          name: 'shocked',
          image: shockedImage,
          actions: ['发送表情', '收藏这一幕', '分享这一幕']
        },
        {
          name: 'think',
          image: thinkImage,
          actions: ['发送表情', '收藏这一幕', '分享这一幕']
        },
        {
          name: 'touching',
          image: touchingImage,
          actions: ['发送表情', '收藏这一幕', '分享这一幕']
        }
      ]
    }
  },

  computed: {
    // 计算属性：使用自定义表情列表或默认列表
    emojiList() {
      return this.customEmojiList || this.defaultEmojiList
    }
  },

  methods: {
    // 处理表情操作
    handleEmojiAction(command) {
      if (this.disabled) {
        return
      }

      // 向父组件发射事件
      this.$emit('emoji-action', command)
    }
  }
}
</script>

<style scoped>
.header {
  display: flex;
  justify-content: center;  /* 表情居中 */
  align-items: center;
  line-height: 10vh;
  position: relative;
  background: transparent;  /* 移除背景色 */
  border: none;  /* 移除边框 */
}

/* 表情工具栏样式 */
.emoji-toolbar {
  display: flex;
  gap: 12px;  /* 增加间距让表情更好看 */
}

.emoji-button {
  border-radius: 50% !important;
  height: 48px !important;  /* 稍微增大按钮 */
  width: 48px !important;
  background: rgba(255, 255, 255, 0.95) !important;  /* 半透明白色背景 */
  border: 2px solid rgba(0, 0, 0, 0.1) !important;  /* 淡边框 */
  padding: 6px !important;
  transition: all 0.3s ease;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);  /* 添加阴影 */
}

.emoji-button:hover {
  background: rgba(255, 255, 255, 1) !important;
  border-color: #409EFF !important;
  transform: scale(1.15);  /* 稍微增大缩放效果 */
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
}

.emoji-button--disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.emoji-button--disabled:hover {
  transform: none !important;
  border-color: rgba(0, 0, 0, 0.1) !important;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1) !important;
}

.emoji-image {
  width: 32px;
  height: 32px;
  border-radius: 50%;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .emoji-toolbar {
    gap: 8px;
  }

  .emoji-button {
    height: 40px !important;
    width: 40px !important;
  }

  .emoji-image {
    width: 28px;
    height: 28px;
  }
}

/* 深色主题支持 */
@media (prefers-color-scheme: dark) {
  .emoji-button {
    background: rgba(44, 62, 80, 0.9) !important;
    border-color: rgba(255, 255, 255, 0.1) !important;
  }

  .emoji-button:hover {
    background: rgba(44, 62, 80, 1) !important;
    border-color: #409EFF !important;
  }
}
</style>