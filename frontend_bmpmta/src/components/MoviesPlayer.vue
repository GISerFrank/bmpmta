<template>
  <div class="player-container">
    <div class="video-wrapper" :class="{ 'fullscreen': isFullscreen }">
      <video
          ref="videoPlayer"
          class="video-player"
          @click="togglePlay"
          @timeupdate="updateProgress"
          @ended="videoEnded"
          @error="handleVideoError"
      >
        <!-- 提供多个不同格式的视频源 -->
        <source :src="videoSource" type="video/mp4">
        <source :src="videoSource.replace('.mp4', '.webm')" type="video/webm">
        <source :src="videoSource.replace('.mp4', '.ogg')" type="video/ogg">
        <!-- 提示不支持视频格式的信息 -->
        <div class="video-error-message">您的浏览器不支持该视频格式</div>
      </video>

      <!-- 弹幕层 -->
      <div v-if="danmuEnabled" class="danmu-container">
        <slot name="danmu"></slot>
      </div>

      <!-- 视频错误提示 -->
      <div v-if="videoError" class="video-error-overlay">
        <div class="video-error-content">
          <el-icon :size="48"><VideoCamera /></el-icon>
          <h3>{{ videoErrorMessage }}</h3>
          <el-button type="primary" @click="retryLoadVideo">重试</el-button>
        </div>
      </div>

      <!-- 非全屏模式：简单的播放按钮覆盖层 -->
      <div v-if="!isFullscreen" class="simple-overlay">
        <div
            class="play-overlay"
            v-show="!isPlaying || showPlayOverlay"
            @click="togglePlay"
        >
          <div class="play-button-large">
            <el-icon :size="48" color="white">
              <component :is="isPlaying ? 'VideoPause' : 'VideoPlay'" />
            </el-icon>
          </div>
        </div>
      </div>

      <!-- 全屏模式：完整的控制栏 -->
      <div v-if="isFullscreen" class="player-controls" v-show="controlsVisible || isHovering">
        <!-- 弹幕表情发送器 (仅在展开控制栏时显示) -->
        <div v-if="emojiSelectorVisible" class="emoji-selector-container">
          <div class="emoji-grid">
            <div
                v-for="(emoji, index) in emojiList"
                :key="index"
                class="emoji-item"
                @click="handleEmojiSelect(emoji.name)"
            >
              <el-image :src="emoji.image" fit="contain" class="emoji-image" />
            </div>
          </div>
          <el-input
              v-model="danmuText"
              class="danmu-input"
              placeholder="发送弹幕..."
              @keyup.enter="sendDanmu"
          >
            <template #append>
              <el-select v-model="danmuColor" size="small" class="color-selector">
                <el-option
                    v-for="color in colorOptions"
                    :key="color.value"
                    :label="color.label"
                    :value="color.value"
                    :style="{ color: color.value }"
                >
                  <span :style="{ color: color.value }">{{ color.label }}</span>
                </el-option>
              </el-select>
              <el-button @click="sendDanmu">发送</el-button>
            </template>
          </el-input>
        </div>

        <div class="progress-container" @click="seek">
          <div class="progress-bar">
            <div class="progress-filled" :style="{ width: `${progress}%` }"></div>
          </div>
          <div class="time-display">
            <span>{{ formatTime(currentTime) }}</span>
            <span>/</span>
            <span>{{ formatTime(duration) }}</span>
          </div>
        </div>

        <!-- 播放器控制区域 -->
        <div class="button-controls">
          <!-- 左侧控制按钮组 -->
          <div class="left-controls">
            <button @click="togglePlay" class="control-button">
              <el-icon :size="20" :color="iconColor">
                <component :is="isPlaying ? 'VideoPause' : 'VideoPlay'" />
              </el-icon>
            </button>

            <div class="volume-control">
              <button @click="toggleMute" class="control-button">
                <el-icon :size="18" :color="iconColor">
                  <component :is="isMuted ? 'Mute' : 'Microphone'" />
                </el-icon>
              </button>
              <div class="volume-slider-container">
                <el-slider
                    v-model="volume"
                    :min="0"
                    :max="100"
                    @input="updateVolume"
                    class="volume-slider"
                />
              </div>
            </div>
          </div>

          <!-- 中间控制按钮组 -->
          <div class="middle-controls">
            <div class="playback-rate">
              <el-dropdown trigger="click" :teleported="false" placement="top">
                <button class="control-button">
                  <el-icon :size="18" :color="iconColor"><Timer /></el-icon>
                  <span class="rate-display">{{ playbackRate }}x</span>
                </button>
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item
                        v-for="rate in playbackRates"
                        :key="rate"
                        @click="setPlaybackRate(rate)"
                    >
                      {{ rate }}x
                    </el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </div>
          </div>

          <!-- 右侧控制按钮组 -->
          <div class="right-controls">
            <!-- 弹幕开关按钮 -->
            <button @click="toggleDanmu" class="control-button">
              <el-icon :size="18" :color="iconColor">
                <component :is="danmuEnabled ? 'ChatDotSquare' : 'ChatLineSquare'" />
              </el-icon>
              <span class="button-text">{{ danmuEnabled ? '关闭弹幕' : '开启弹幕' }}</span>
            </button>

            <!-- 弹幕输入框显示按钮 -->
            <button @click="toggleEmojiSelector" class="control-button">
              <el-icon :size="18" :color="iconColor">
                <component :is="emojiSelectorVisible ? 'Close' : 'ChatDotRound'" />
              </el-icon>
              <span class="button-text">{{ emojiSelectorVisible ? '关闭评论' : '发表评论' }}</span>
            </button>

            <!-- 全屏按钮 -->
            <button @click="toggleFullscreen" class="control-button">
              <el-icon :size="18" :color="iconColor">
                <component :is="'FullscreenExit'" />
              </el-icon>
              <span class="button-text">退出全屏</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 非全屏模式的外部控制栏 -->
    <div v-if="!isFullscreen" class="external-controls">
      <!-- 进度条 -->
      <div class="progress-container" @click="seek">
        <div class="progress-bar">
          <div class="progress-filled" :style="{ width: `${progress}%` }"></div>
        </div>
      </div>

      <!-- 控制按钮栏 -->
      <div class="control-section">
        <div class="left-controls">
          <button @click="togglePlay" class="control-button">
            <el-icon :size="18">
              <component :is="isPlaying ? 'VideoPause' : 'VideoPlay'" />
            </el-icon>
          </button>

          <div class="volume-control">
            <button @click="toggleMute" class="control-button">
              <el-icon :size="16">
                <component :is="isMuted ? 'Mute' : 'Microphone'" />
              </el-icon>
            </button>
            <div class="volume-slider-container">
              <el-slider
                  v-model="volume"
                  :min="0"
                  :max="100"
                  @input="updateVolume"
                  class="volume-slider"
                  size="small"
              />
            </div>
          </div>

          <div class="time-display">
            <span>{{ formatTime(currentTime) }} / {{ formatTime(duration) }}</span>
          </div>
        </div>

        <div class="right-controls">
          <!-- 播放速度 -->
          <div class="playback-rate">
            <el-dropdown trigger="click">
              <button class="control-button">
                <span class="rate-display">{{ playbackRate }}x</span>
                <el-icon><ArrowDown /></el-icon>
              </button>
              <template #dropdown>
                <el-dropdown-menu>
                  <el-dropdown-item
                      v-for="rate in playbackRates"
                      :key="rate"
                      @click="setPlaybackRate(rate)"
                  >
                    {{ rate }}x
                  </el-dropdown-item>
                </el-dropdown-menu>
              </template>
            </el-dropdown>
          </div>

          <!-- 弹幕开关 -->
          <button @click="toggleDanmu" class="control-button" :class="{ active: danmuEnabled }">
            <el-icon :size="16">
              <component :is="danmuEnabled ? 'ChatDotSquare' : 'ChatLineSquare'" />
            </el-icon>
            <span class="button-text">弹幕</span>
          </button>

          <!-- 弹幕发送 -->
          <button @click="toggleEmojiSelector" class="control-button" :class="{ active: emojiSelectorVisible }">
            <el-icon :size="16">
              <component :is="emojiSelectorVisible ? 'Close' : 'ChatDotRound'" />
            </el-icon>
            <span class="button-text">发弹幕</span>
          </button>

          <!-- 全屏按钮 -->
          <button @click="toggleFullscreen" class="control-button">
            <el-icon :size="16">
              <FullScreen />
            </el-icon>
            <span class="button-text">全屏</span>
          </button>
        </div>
      </div>

      <!-- 弹幕输入区域（非全屏模式） -->
      <div v-if="emojiSelectorVisible" class="external-danmu-input">
        <div class="emoji-grid">
          <div
              v-for="(emoji, index) in emojiList"
              :key="index"
              class="emoji-item"
              @click="handleEmojiSelect(emoji.name)"
          >
            <el-image :src="emoji.image" fit="contain" class="emoji-image" />
          </div>
        </div>
        <el-input
            v-model="danmuText"
            class="danmu-input"
            placeholder="发送弹幕..."
            @keyup.enter="sendDanmu"
        >
          <template #append>
            <el-select v-model="danmuColor" size="small" class="color-selector">
              <el-option
                  v-for="color in colorOptions"
                  :key="color.value"
                  :label="color.label"
                  :value="color.value"
                  :style="{ color: color.value }"
              >
                <span :style="{ color: color.value }">{{ color.label }}</span>
              </el-option>
            </el-select>
            <el-button @click="sendDanmu">发送</el-button>
          </template>
        </el-input>
      </div>
    </div>

<!--    <div class="movie-info" v-if="!isFullscreen">-->
<!--      <h3>{{ title || '我的天才女友' }}</h3>-->
<!--      <div class="episode-selector">-->
<!--        <el-dropdown>-->
<!--          <span class="episode-dropdown">-->
<!--            选集 <el-icon><ArrowDown /></el-icon>-->
<!--          </span>-->
<!--          <template #dropdown>-->
<!--            <el-dropdown-menu>-->
<!--              <el-dropdown-item v-for="ep in episodes" :key="ep.id" @click="changeEpisode(ep)">-->
<!--                第{{ ep.id }}集: {{ ep.title }}-->
<!--              </el-dropdown-item>-->
<!--            </el-dropdown-menu>-->
<!--          </template>-->
<!--        </el-dropdown>-->
<!--      </div>-->
<!--    </div>-->
  </div>
</template>

<script>
// MoviesPlayer.vue
export default {
  name: 'MoviesPlayer',
  props: {
    title: {
      type: String,
      default: ''
    },
    source: {
      type: String,
      default: null
    },
    emojiList: {
      type: Array,
      default: () => []
    }
  },

  // 明确定义组件发出的事件
  emits: [
    'fullscreen-change',
    'play-state-change',
    'time-update',
    'emoji-selected',
    'danmu-sent'
  ],

  data() {
    return {
      videoSource: this.source || 'https://media.w3.org/2010/05/sintel/trailer.mp4',
      isPlaying: false,
      isMuted: false,
      duration: 0,
      currentTime: 0,
      progress: 0,
      volume: 80,
      playbackRate: 1,
      controlsVisible: true,
      controlsTimeout: null,
      isHovering: false,
      isFullscreen: false,
      playbackRates: [0.5, 0.75, 1, 1.25, 1.5, 2],
      iconColor: '#ffffff',
      videoError: false,
      videoErrorMessage: '视频加载失败，请重试',
      currentEpisode: 1,
      episodes: [
        { id: 1, title: '两个女孩', source: 'https://media.w3.org/2010/05/sintel/trailer.mp4' },
        { id: 2, title: '金钱', source: 'https://media.w3.org/2010/05/sintel/trailer.mp4' },
        { id: 3, title: '男孩', source: 'https://media.w3.org/2010/05/sintel/trailer.mp4' },
        { id: 4, title: '边界的溶解', source: 'https://media.w3.org/2010/05/sintel/trailer.mp4' }
      ],
      // 弹幕相关
      danmuEnabled: true,
      emojiSelectorVisible: false,
      danmuText: '',
      danmuColor: '#FFFFFF',
      colorOptions: [
        { value: '#FFFFFF', label: '白色' },
        { value: '#FF0000', label: '红色' },
        { value: '#00FF00', label: '绿色' },
        { value: '#0000FF', label: '蓝色' },
        { value: '#FFFF00', label: '黄色' },
        { value: '#FF00FF', label: '粉色' },
        { value: '#00FFFF', label: '青色' },
        { value: '#FFA500', label: '橙色' }
      ],
      // 非全屏播放按钮显示
      showPlayOverlay: false,
      playOverlayTimeout: null
    };
  },

  // 使用计算属性获取正确的视频地址
  computed: {
    currentVideoSource() {
      return this.videoSource || 'https://media.w3.org/2010/05/sintel/trailer.mp4';
    }
  },

  mounted() {
    const player = this.$refs.videoPlayer;
    if (!player) return; // 安全检查

    // 视频元数据加载事件
    player.addEventListener('loadedmetadata', () => {
      this.duration = player.duration;
      this.updateVolume();
      this.videoError = false;
    });

    // 其他事件监听
    player.addEventListener('canplay', () => {
      this.videoError = false;
    });

    player.addEventListener('loadstart', () => {
      console.log('Video loading started');
    });

    // 全屏模式控制显示/隐藏的逻辑
    this.hideControlsAfterDelay();

    // 确保元素存在后再添加事件监听
    if (this.$el) {
      this.$el.addEventListener('mousemove', this.showControls);
      this.$el.addEventListener('mouseenter', () => { this.isHovering = true; });
      this.$el.addEventListener('mouseleave', () => { this.isHovering = false; });
    }

    // 全局事件监听
    document.addEventListener('keydown', this.handleKeydown);

    // 使用window的全屏事件监听而不是document
    window.addEventListener('fullscreenchange', this.handleFullscreenChange);
  },

  beforeUnmount() {
    // 清理事件监听
    document.removeEventListener('keydown', this.handleKeydown);
    window.removeEventListener('fullscreenchange', this.handleFullscreenChange);

    if (this.$el) {
      this.$el.removeEventListener('mousemove', this.showControls);
      this.$el.removeEventListener('mouseenter', () => {});
      this.$el.removeEventListener('mouseleave', () => {});
    }

    if (this.controlsTimeout) {
      clearTimeout(this.controlsTimeout);
    }

    if (this.playOverlayTimeout) {
      clearTimeout(this.playOverlayTimeout);
    }
  },

  methods: {
    // 播放控制
    togglePlay() {
      const player = this.$refs.videoPlayer;
      if (!player) return;

      if (this.isPlaying) {
        player.pause();
      } else {
        player.play().catch(err => {
          console.error('Play error:', err);
          // 处理自动播放失败的情况
        });
      }
      this.isPlaying = !this.isPlaying;

      // 非全屏模式下显示播放状态提示
      if (!this.isFullscreen) {
        this.showPlayOverlayBriefly();
      } else {
        this.showControls();
      }

      // 发射播放状态改变事件
      this.$emit('play-state-change', this.isPlaying);
    },

    // 非全屏模式下短暂显示播放按钮
    showPlayOverlayBriefly() {
      this.showPlayOverlay = true;

      if (this.playOverlayTimeout) {
        clearTimeout(this.playOverlayTimeout);
      }

      this.playOverlayTimeout = setTimeout(() => {
        this.showPlayOverlay = false;
      }, 1000);
    },

    // 更新播放进度
    updateProgress() {
      const player = this.$refs.videoPlayer;
      if (!player) return;

      this.currentTime = player.currentTime;
      this.progress = (player.currentTime / player.duration) * 100 || 0;

      // 发送时间更新事件
      this.$emit('time-update', this.currentTime);
    },

    // 拖动进度条
    seek(event) {
      const player = this.$refs.videoPlayer;
      if (!player) return;

      const progressBar = event.currentTarget.querySelector('.progress-bar');
      const rect = progressBar.getBoundingClientRect();
      const seekPosition = (event.clientX - rect.left) / rect.width;

      player.currentTime = seekPosition * player.duration;
      this.updateProgress();
    },

    // 更新音量
    updateVolume() {
      const player = this.$refs.videoPlayer;
      if (!player) return;

      player.volume = this.volume / 100;
      this.isMuted = this.volume === 0;
    },

    // 切换静音
    toggleMute() {
      if (this.isMuted) {
        this.volume = this.volume > 0 ? this.volume : 80;
        this.isMuted = false;
      } else {
        this.volume = 0;
        this.isMuted = true;
      }
      this.updateVolume();
    },

    // 设置播放速度
    setPlaybackRate(rate) {
      const player = this.$refs.videoPlayer;
      if (!player) return;

      player.playbackRate = rate;
      this.playbackRate = rate;
    },

    // 全屏状态变化监听
    handleFullscreenChange() {
      // 获取之前的状态用于比较
      const wasFullscreen = this.isFullscreen;
      // 更新当前状态
      this.isFullscreen = !!document.fullscreenElement;

      // 如果状态发生了变化，通知父组件
      if (wasFullscreen !== this.isFullscreen) {
        this.$emit('fullscreen-change', this.isFullscreen);
      }
    },

    // 切换全屏
    toggleFullscreen() {
      try {
        if (!document.fullscreenElement) {
          // 请求进入全屏
          const container = this.$el;
          if (container && container.requestFullscreen) {
            container.requestFullscreen().catch(err => {
              console.error('Fullscreen error:', err);
            });
          }
        } else {
          // 退出全屏
          if (document.exitFullscreen) {
            document.exitFullscreen().catch(err => {
              console.error('Exit fullscreen error:', err);
            });
          }
        }
      } catch (err) {
        console.error('Fullscreen toggle error:', err);
      }
    },

    // 格式化时间显示
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins}:${secs < 10 ? '0' : ''}${secs}`;
    },

    // 显示控件（仅全屏模式）
    showControls() {
      if (this.isFullscreen) {
        this.controlsVisible = true;
        this.hideControlsAfterDelay();
      }
    },

    // 延迟隐藏控件（仅全屏模式）
    hideControlsAfterDelay() {
      if (!this.isFullscreen) return;

      if (this.controlsTimeout) {
        clearTimeout(this.controlsTimeout);
      }

      this.controlsTimeout = setTimeout(() => {
        if (!this.isHovering && this.isPlaying && !this.emojiSelectorVisible) {
          this.controlsVisible = false;
        }
      }, 3000);
    },

    // 键盘快捷键
    handleKeydown(event) {
      // 只有在组件处于活动状态时才处理快捷键
      if (this.$el && !this.$el.contains(document.activeElement) && document.activeElement !== document.body) {
        return;
      }

      switch (event.key) {
        case ' ':
          this.togglePlay();
          event.preventDefault();
          break;
        case 'ArrowRight':
          if (this.$refs.videoPlayer) {
            this.$refs.videoPlayer.currentTime += 10;
            this.showControls();
          }
          event.preventDefault();
          break;
        case 'ArrowLeft':
          if (this.$refs.videoPlayer) {
            this.$refs.videoPlayer.currentTime -= 10;
            this.showControls();
          }
          event.preventDefault();
          break;
        case 'f':
          this.toggleFullscreen();
          event.preventDefault();
          break;
        case 'm':
          this.toggleMute();
          event.preventDefault();
          break;
        case 'd':
          this.toggleDanmu();
          event.preventDefault();
          break;
        case 'c':
          this.toggleEmojiSelector();
          event.preventDefault();
          break;
      }
    },

    // 视频播放结束
    videoEnded() {
      this.isPlaying = false;
      if (this.isFullscreen) {
        this.controlsVisible = true;
      }

      // 发送播放状态变化事件
      this.$emit('play-state-change', false);

      // 自动播放下一集
      if (this.currentEpisode < this.episodes.length) {
        this.changeEpisode(this.episodes[this.currentEpisode]);
      }
    },

    // 切换剧集
    changeEpisode(episode) {
      this.videoSource = episode.source;
      this.currentEpisode = episode.id;
      this.videoError = false;

      // 重置播放器状态
      this.$nextTick(() => {
        const player = this.$refs.videoPlayer;
        if (!player) return;

        player.load();
        this.progress = 0;
        this.currentTime = 0;

        // 自动播放新剧集
        this.isPlaying = true;
        player.play().catch(err => {
          console.error('Error playing video:', err);
          this.videoError = true;
          this.videoErrorMessage = '播放视频时出错，可能是浏览器阻止了自动播放';
        });

        // 发送播放状态变化事件
        this.$emit('play-state-change', true);
      });
    },

    // 处理视频错误
    handleVideoError(e) {
      console.error('Video error:', e);
      this.videoError = true;

      // 根据错误代码设置不同的错误消息
      const player = this.$refs.videoPlayer;
      if (player && player.error) {
        switch(player.error.code) {
          case 1:
            this.videoErrorMessage = '视频加载被中止';
            break;
          case 2:
            this.videoErrorMessage = '网络错误，无法加载视频';
            break;
          case 3:
            this.videoErrorMessage = '视频解码错误';
            break;
          case 4:
            this.videoErrorMessage = '视频格式不支持';
            break;
          default:
            this.videoErrorMessage = '视频加载失败，请重试';
        }
      }
    },

    // 重试加载视频
    retryLoadVideo() {
      this.videoError = false;
      const player = this.$refs.videoPlayer;
      if (!player) return;

      player.load();

      if (this.isPlaying) {
        player.play().catch(err => {
          console.error('Error playing video on retry:', err);
          this.videoError = true;
        });
      }
    },

    // 弹幕相关方法
    // 切换弹幕显示
    toggleDanmu() {
      this.danmuEnabled = !this.danmuEnabled;

      // 通知父组件弹幕状态变化
      this.$emit('danmu-toggle', this.danmuEnabled);
    },

    // 切换表情选择器显示
    toggleEmojiSelector() {
      this.emojiSelectorVisible = !this.emojiSelectorVisible;

      // 如果显示表情选择器，确保控制栏保持显示
      if (this.emojiSelectorVisible && this.isFullscreen) {
        this.controlsVisible = true;
      }
    },

    // 处理表情选择
    handleEmojiSelect(emojiName) {
      this.$emit('emoji-selected', emojiName);
    },

    // 发送弹幕
    sendDanmu() {
      if (!this.danmuText.trim()) return;

      this.$emit('danmu-sent', {
        content: this.danmuText,
        color: this.danmuColor,
        time: this.currentTime
      });

      // 清空输入框
      this.danmuText = '';
    }
  }
};
</script>

<style scoped>
.player-container {
  width: 100%;
  max-width: 100%;
  margin: 0;
  border-radius: 8px;
  background-color: #000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  position: relative;
  box-sizing: border-box;
  z-index: 5;
  overflow: hidden;
}

.video-wrapper {
  position: relative;
  width: 100%;
  padding-top: 56.25%; /* 16:9 Aspect Ratio */
  background-color: #000;
  overflow: hidden;
  max-height: calc(100vh - 120px);
  box-sizing: border-box;
}

/* 全屏特有样式 */
.video-wrapper.fullscreen {
  padding-top: 0;
  height: 100vh;
  width: 100vw;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 9999;
}

.video-player {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: contain;
  cursor: pointer;
  z-index: 1;
}

/* 弹幕容器 */
.danmu-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  overflow: hidden;
  z-index: 5;
}

.video-error-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 20;
}

.video-error-content {
  text-align: center;
  color: white;
  padding: 20px;
  border-radius: 8px;
  background-color: rgba(60, 60, 60, 0.8);
}

.video-error-content h3 {
  margin: 16px 0;
}

.video-error-message {
  color: white;
  text-align: center;
  padding: 20px;
}

/* 非全屏模式：简单播放覆盖层 */
.simple-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 10;
  pointer-events: none;
}

.play-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.3);
  pointer-events: auto;
  cursor: pointer;
  transition: opacity 0.3s ease;
}

.play-button-large {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.play-button-large:hover {
  background: rgba(0, 0, 0, 0.8);
  transform: scale(1.1);
}

/* 全屏模式控制栏样式 */
.player-controls {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  padding: 8px;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.9));
  transition: opacity 0.3s ease;
  display: flex;
  flex-direction: column;
  gap: 6px;
  z-index: 10;
  box-sizing: border-box;
}

/* 表情选择器 */
.emoji-selector-container {
  width: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  padding: 8px;
  border-radius: 4px;
  margin-bottom: 10px;
}

.emoji-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 8px;
  justify-content: center;
}

.emoji-item {
  width: 32px;
  height: 32px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.1);
  transition: all 0.2s;
}

.emoji-item:hover {
  background-color: rgba(255, 255, 255, 0.3);
  transform: scale(1.1);
}

.emoji-image {
  width: 24px;
  height: 24px;
}

.danmu-input {
  display: flex;
  width: 100%;
}

.danmu-input :deep(.el-input__wrapper) {
  background-color: rgba(255, 255, 255, 0.8);
}

.color-selector {
  width: 70px;
}

/* 修改进度条容器以水平居中显示 */
.progress-container {
  display: flex;
  flex-direction: column;
  width: 100%;
  gap: 2px;
  align-items: center;
}

/* 修改进度条为90%宽度并居中 */
.progress-bar {
  width: 90%;
  height: 4px;
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 2px;
  cursor: pointer;
  position: relative;
}

.progress-filled {
  background-color: #2374FF;
  height: 100%;
  border-radius: 2px;
  position: absolute;
  top: 0;
  left: 0;
}

/* 调整时间显示 */
.time-display {
  color: white;
  font-size: 12px;
  display: flex;
  gap: 3px;
  margin-bottom: 4px;
  align-self: flex-start;
  margin-left: 5%;
}

/* 按钮控制区样式 */
.button-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 90%;
  margin: 0 auto;
  box-sizing: border-box;
  padding: 0 10px;
}

/* 将控制区分为左中右三部分 */
.left-controls, .middle-controls, .right-controls {
  display: flex;
  align-items: center;
}

.left-controls {
  gap: 12px;
}

.right-controls {
  gap: 15px;
}

/* 控制按钮样式 */
.control-button {
  background: none;
  border: none;
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 6px 8px;
  border-radius: 4px;
  transition: background-color 0.2s;
}

.control-button:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

/* 按钮文字样式 */
.button-text {
  font-size: 12px;
  margin-left: 4px;
  opacity: 0;
  max-width: 0;
  overflow: hidden;
  white-space: nowrap;
  transition: all 0.2s ease;
}

/* 鼠标悬停时显示文字 */
.control-button:hover .button-text {
  opacity: 1;
  max-width: 80px;
}

/* 全屏模式下始终显示退出全屏文字 */
.video-wrapper.fullscreen .right-controls .control-button:last-child .button-text {
  opacity: 1;
  max-width: 80px;
}

/* 音量控制样式优化 */
.volume-control {
  display: flex;
  align-items: center;
  position: relative;
}

.volume-slider-container {
  width: 70px;
  margin-left: 8px;
}

.volume-slider {
  --el-slider-main-bg-color: #2374FF;
}

.playback-rate {
  display: flex;
  align-items: center;
  position: relative;
  z-index: 10;
}

.playback-rate .el-dropdown {
  display: flex;
}

.playback-rate .control-button {
  padding: 6px 8px;
  background-color: rgba(255, 255, 255, 0.05);
  border-radius: 4px;
  min-width: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.playback-rate .control-button:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.rate-display {
  margin-left: 2px;
  color: white;
  font-size: 12px;
}

/* 非全屏模式：外部控制栏样式 */
.external-controls {
  background: #fafafa;
  border-top: 1px solid #e1e8ed;
}

.external-controls .progress-container {
  padding: 12px 16px 8px;
}

.external-controls .progress-bar {
  width: 100%;
  height: 4px;
  background-color: #e1e8ed;
}

.external-controls .progress-filled {
  background-color: #2374FF;
}

.control-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 16px 12px;
}

.external-controls .left-controls,
.external-controls .right-controls {
  display: flex;
  align-items: center;
  gap: 15px;
}

.external-controls .control-button {
  color: #333;
  padding: 6px 8px;
  font-size: 13px;
}

.external-controls .control-button:hover {
  background-color: #f0f0f0;
}

.external-controls .control-button.active {
  color: #2374FF;
  background-color: #e6f3ff;
}

.external-controls .volume-slider-container {
  width: 60px;
}

.external-controls .time-display {
  color: #666;
  font-size: 13px;
}

.external-controls .rate-display {
  color: #333;
  font-size: 13px;
}

.external-controls .button-text {
  font-size: 13px;
  margin-left: 4px;
}

/* 外部弹幕输入区域 */
.external-danmu-input {
  padding: 12px 16px;
  border-top: 1px solid #e1e8ed;
  background: #f9f9f9;
}

.external-danmu-input .emoji-grid {
  display: flex;
  gap: 8px;
  margin-bottom: 8px;
  justify-content: flex-start;
  flex-wrap: wrap;
}

.external-danmu-input .emoji-item {
  width: 28px;
  height: 28px;
  background-color: rgba(0, 0, 0, 0.05);
}

.external-danmu-input .emoji-item:hover {
  background-color: rgba(0, 0, 0, 0.1);
}

.external-danmu-input .emoji-image {
  width: 20px;
  height: 20px;
}

/* 电影信息部分 */
.movie-info {
  padding: 12px 16px;
  background-color: #1A1A1A;
  color: white;
}

.movie-info h3 {
  margin: 0 0 5px 0;
  font-size: 16px;
  font-weight: normal;
}

.episode-selector {
  margin-top: 10px;
}

.episode-dropdown {
  color: #2374FF;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 2px;
  font-size: 14px;
}

/* 小屏幕适配 */
@media (max-width: 768px) {
  .player-container {
    max-width: 100%;
    border-radius: 4px;
  }

  .video-wrapper {
    max-height: 50vh;
  }

  .button-controls {
    flex-wrap: wrap;
    gap: 8px;
    justify-content: center;
  }

  .control-button {
    padding: 4px;
  }

  .volume-control {
    order: -1;
    width: 100%;
    justify-content: center;
    margin-bottom: 8px;
  }

  .external-controls .left-controls,
  .external-controls .right-controls {
    gap: 8px;
  }

  .external-controls .control-button {
    padding: 4px 6px;
    font-size: 12px;
  }

  .volume-slider-container {
    width: 50px !important;
  }

  .emoji-grid {
    justify-content: center;
  }
}

/* 确保全屏模式中的元素正确响应 */
@media (orientation: landscape) and (max-height: 450px) {
  .video-wrapper.fullscreen .player-controls {
    padding: 4px 8px;
  }

  .video-wrapper.fullscreen .emoji-selector-container {
    padding: 4px;
  }

  .video-wrapper.fullscreen .emoji-grid {
    margin-bottom: 4px;
  }
}
</style>