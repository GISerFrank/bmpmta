<template>
  <div class="music-page-container">
    <!-- 左侧歌词区域 -->
    <div class="lyrics-container">
      <div class="lyrics-header">
        <h2>{{ currentSong.title || '未播放歌曲' }}</h2>
        <p class="artist">
          {{ currentSong.artist || '未知歌手' }}
        </p>

        <!-- 歌词功能控制栏 -->
        <div class="lyrics-controls">
          <button
            class="highlight-mode-btn"
            :class="{ active: isHighlightMode }"
            title="点击进入标记模式"
            @click="toggleHighlightMode"
          >
            <span class="highlight-icon">✨</span>
            <span>{{ isHighlightMode ? '退出标记' : '标记歌词' }}</span>
          </button>

          <button
            class="show-favorites-btn"
            :class="{ active: showOnlyFavorites }"
            title="只显示收藏的歌词"
            @click="toggleShowFavorites"
          >
            <span class="heart-icon">❤️</span>
            <span class="favorite-count">({{ favoriteLyricsCount }})</span>
          </button>
        </div>
      </div>

      <div
        ref="lyricsContainer"
        class="lyrics-content"
      >
        <p
          v-if="!filteredLyrics.length"
          class="empty-lyrics"
        >
          {{ showOnlyFavorites ? '还没有收藏的歌词' : '暂无歌词' }}
        </p>

        <div
          v-for="(line, _index) in filteredLyrics"
          :key="line.originalIndex"
          :ref="el => lyricRefs[line.originalIndex] = el"
          :class="[
            'lyric-line',
            {
              'active-lyric': currentLyricIndex === line.originalIndex,
              'highlighted-lyric': line.isHighlighted,
              'clickable': isHighlightMode,
              'favorite-lyric': line.isFavorite
            }
          ]"
          @click="handleLyricClick(line, line.originalIndex)"
          @mouseenter="showLyricTooltip(line, $event)"
          @mouseleave="hideLyricTooltip"
        >
          <!-- 歌词文本 -->
          <span class="lyric-text">{{ line.text }}</span>

          <!-- 收藏状态指示器 -->
          <div
            v-if="line.isFavorite"
            class="favorite-indicator"
          >
            <span class="favorite-star">⭐</span>
          </div>

          <!-- 点亮效果覆盖层 -->
          <div
            v-if="line.isHighlighted"
            class="highlight-overlay"
          />

          <!-- 悬浮工具提示 -->
          <div
            v-if="hoveredLyric === line.originalIndex && isHighlightMode"
            class="lyric-tooltip"
          >
            点击{{ line.isFavorite ? '取消收藏' : '收藏此句' }}
          </div>
        </div>
      </div>

      <!-- 收藏歌词统计 -->
      <div
        v-if="favoriteLyricsCount > 0"
        class="lyrics-stats"
      >
        <span class="stats-text">
          已收藏 {{ favoriteLyricsCount }} 句歌词
        </span>
        <button
          class="export-favorites-btn"
          title="导出收藏的歌词"
          @click="exportFavorites"
        >
          📤 导出
        </button>
      </div>
    </div>

    <!-- 右侧播放器区域 -->
    <div class="player-container">
      <!-- 专辑和CD区域 - 更居中的设计 -->
      <div class="player-visual-wrapper">
        <div class="player-visual">
          <!-- 专辑封面区域 -->
          <div class="album-cover">
            <img
              :src="albumCoverSrc"
              alt="专辑封面"
            >
            <div class="album-name">
              <h3>{{ currentSong.title }}</h3>
              <p>{{ currentSong.artist }}</p>
            </div>
          </div>

          <!-- 使用原来的CD组件并调整大小 -->
          <!-- 使用新的CD动画处理方式 -->
          <div
            ref="cdDisc"
            class="cd-disc"
            :class="{ readyPlay: isReadyPlay, playing: isReadyPlay && isPlaying }"
            :style="cdRotationStyle"
            @click="chooseCD(true)"
          >
            <!-- 添加CD刻痕效果 -->
            <div class="cd-grooves" />

            <!-- 添加CD反光效果 -->
            <div class="cd-reflection" />

            <!-- 可选：添加可视化标记点 -->
            <div
              v-if="showMarker"
              class="cd-marker"
            />
          </div>
        </div>
      </div>

      <!-- 播放控制区域 - 居中设计 -->
      <div class="player-controls">
        <div class="time-display">
          <span>{{ formatTime(currentTime) }}</span>
        </div>

        <div class="progress-bar">
          <div class="progress-bar-bg" />
          <div
            class="progress-bar-fill"
            :style="{ width: progressPercentage + '%' }"
          />
          <div
            class="progress-handle"
            :style="{ left: progressPercentage + '%' }"
          />
        </div>

        <div class="control-buttons">
          <button
            class="prev-btn"
            @click="prevSong"
          >
            <div class="prev-icon" />
          </button>
          <button
            class="play-btn"
            @click="togglePlay"
          >
            <div :class="isPlaying ? 'pause-icon' : 'play-icon'" />
          </button>
          <button
            class="next-btn"
            @click="nextSong"
          >
            <div class="next-icon" />
          </button>
        </div>
      </div>
    </div>
    <!-- 收藏歌词弹窗 -->
    <div
      v-if="showFavoritesModal"
      class="favorites-modal-overlay"
      @click="closeFavoritesModal"
    >
      <div
        class="favorites-modal"
        @click.stop
      >
        <div class="modal-header">
          <h3>收藏的歌词</h3>
          <button
            class="close-btn"
            @click="closeFavoritesModal"
          >
            ×
          </button>
        </div>
        <div class="modal-content">
          <div
            v-for="lyric in allFavoriteLyrics"
            :key="`${lyric.songId}-${lyric.index}`"
            class="favorite-lyric-item"
          >
            <div class="lyric-info">
              <span class="lyric-text">{{ lyric.text }}</span>
              <span class="song-info">来自: {{ lyric.songTitle }} - {{ lyric.artist }}</span>
            </div>
            <button
              class="remove-favorite-btn"
              title="取消收藏"
              @click="removeFavorite(lyric)"
            >
              🗑️
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "MusicPage",

  data() {
    return {
      isPlaying: false,
      isReadyPlay: true, // 将其设置为true，以便可以立即显示和使用CD
      isOpen: true, // 假设CD盒已经打开
      currentTime: 66, // 当前播放时间（秒）
      totalTime: 240, // 总时长（秒）
      progressPercentage: 27.5, // 进度百分比
      currentLyricIndex: 3,
      albumCoverSrc: require('@/assets/album_cover/Jay.png'), // 专辑封面路径

      // 歌曲信息
      currentSong: {
        title: "Jay",
        artist: "周杰伦",
        album: "Jay",
      },
      // 新增歌词功能相关数据
      isHighlightMode: false,
      showOnlyFavorites: false,
      showFavoritesModal: false,
      hoveredLyric: null,
      lyricRefs: {},

      // 歌词数据增强 - 为每句歌词添加状态
      enhancedLyrics: [
        {time: 0, text: "全世界不知道多少人在唱我的歌", isHighlighted: false, isFavorite: false},
        {time: 5, text: "在我背后的舞台有多大", isHighlighted: false, isFavorite: true},
        {time: 10, text: "我看不到", isHighlighted: true, isFavorite: false},
        {time: 15, text: "我听见你说我们会更好的生活", isHighlighted: false, isFavorite: false},
        {time: 20, text: "我们的手紧紧的相握", isHighlighted: true, isFavorite: true},
        {time: 25, text: "你的温柔", isHighlighted: false, isFavorite: false},
        {time: 30, text: "有我就足够", isHighlighted: false, isFavorite: true},
        // ... 更多歌词
      ],
      // 所有收藏的歌词（跨歌曲）
      allFavoriteLyrics: [
        {
          songId: 'jay-001',
          songTitle: 'Jay',
          artist: '周杰伦',
          index: 1,
          text: '在我背后的舞台有多大',
          timestamp: Date.now() - 86400000
        }
      ],
      // 播放器定时器
      timer: null,
      // CD旋转相关数据
      cdRotationAngle: 0,
      cdRotationTimer: null,
    };
  },

  computed: {
      // 动态计算CD的旋转样式
      cdRotationStyle() {
        if (this.isReadyPlay) {
          return {
            transform: `translateX(30px) rotateZ(${this.cdRotationAngle}deg)`,
            // 添加transition以确保立即停止
            transition: this.isPlaying ? 'none' : 'transform 0.1s'
          };
        }
        return {};
      },
      lyrics() {
        return this.enhancedLyrics;
      },

      // 根据筛选条件显示歌词
      filteredLyrics() {
        if (this.showOnlyFavorites) {
          return this.enhancedLyrics
              .map((lyric, index) => ({ ...lyric, originalIndex: index }))
              .filter(lyric => lyric.isFavorite);
        }
        return this.enhancedLyrics.map((lyric, index) => ({ ...lyric, originalIndex: index }));
      },

      // 收藏歌词数量
      favoriteLyricsCount() {
        return this.enhancedLyrics.filter(lyric => lyric.isFavorite).length;
      },
    },

  watch: {
    // 监听播放状态变化，确保CD旋转状态同步
    isPlaying(newValue) {
      if (newValue) {
        if (!this.cdRotationTimer) {
          this.startCdRotation();
        }
      } else {
        this.pauseCdRotation();
      }
    },
    // 监听歌曲变化，保存当前状态
    currentSong(newSong, oldSong) {
      if (oldSong) {
        this.saveLyricsState();
      }
      // 切换歌曲时加载新歌曲的状态
      this.$nextTick(() => {
        this.loadLyricsState();
      });
    }
  },

  mounted() {
    // 添加键盘控制
    window.addEventListener('keydown', (e) => {
      if (e.code === 'Space') {
        this.togglePlay();
        e.preventDefault();
      }
    });

    // 设置初始进度
    this.updateProgress();

    // 加载歌词状态
    this.loadLyricsState();

    // 键盘快捷键扩展
    window.addEventListener('keydown', (e) => {
      if (e.code === 'Space') {
        this.togglePlay();
        e.preventDefault();
      } else if (e.code === 'KeyH' && e.ctrlKey) {
        // Ctrl+H 切换标记模式
        this.toggleHighlightMode();
        e.preventDefault();
      } else if (e.code === 'KeyF' && e.ctrlKey) {
        // Ctrl+F 打开收藏列表
        this.showFavoritesModal = true;
        e.preventDefault();
      }
    });
  },

  beforeUnmount() {
    this.pausePlayback();
    this.pauseCdRotation();
    window.removeEventListener('keydown', null);
  },

  methods: {
    // 新增歌词功能方法
    toggleHighlightMode() {
      this.isHighlightMode = !this.isHighlightMode;
      if (!this.isHighlightMode) {
        this.hoveredLyric = null;
      }
    },

    toggleShowFavorites() {
      this.showOnlyFavorites = !this.showOnlyFavorites;
    },

    handleLyricClick(line, index) {
      if (!this.isHighlightMode) return;

      // 切换收藏状态
      this.enhancedLyrics[index].isFavorite = !this.enhancedLyrics[index].isFavorite;

      // 如果取消收藏，也取消点亮
      if (!this.enhancedLyrics[index].isFavorite) {
        this.enhancedLyrics[index].isHighlighted = false;
      } else {
        // 收藏时自动点亮
        this.enhancedLyrics[index].isHighlighted = true;

        // 添加到全局收藏列表
        this.addToGlobalFavorites(line, index);
      }

      // 触觉反馈（如果支持）
      if (navigator.vibrate) {
        navigator.vibrate(50);
      }

      // 保存到本地存储
      this.saveLyricsState();
    },

    addToGlobalFavorites(lyric, index) {
      const favoriteItem = {
        songId: this.currentSong.id || 'current-song',
        songTitle: this.currentSong.title,
        artist: this.currentSong.artist,
        index: index,
        text: lyric.text,
        timestamp: Date.now()
      };

      // 避免重复添加
      const exists = this.allFavoriteLyrics.find(
          item => item.songId === favoriteItem.songId && item.index === favoriteItem.index
      );

      if (!exists) {
        this.allFavoriteLyrics.unshift(favoriteItem);
      }
    },

    showLyricTooltip(line, _event) {
      console.log('showLyricTooltip called with:', line);
      if (this.isHighlightMode) {
        this.hoveredLyric = line.originalIndex;
      }
    },

    hideLyricTooltip() {
      this.hoveredLyric = null;
    },

    exportFavorites() {
      const favoriteLyrics = this.enhancedLyrics
          .filter(lyric => lyric.isFavorite)
          .map(lyric => lyric.text);

      const exportText = `${this.currentSong.title} - ${this.currentSong.artist}\n收藏的歌词:\n\n${favoriteLyrics.join('\n')}`;

      // 创建下载链接
      const blob = new Blob([exportText], { type: 'text/plain;charset=utf-8' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${this.currentSong.title}-收藏歌词.txt`;
      link.click();
      URL.revokeObjectURL(url);
    },

    removeFavorite(favoriteItem) {
      // 从全局收藏列表移除
      const globalIndex = this.allFavoriteLyrics.findIndex(
          item => item.songId === favoriteItem.songId && item.index === favoriteItem.index
      );
      if (globalIndex > -1) {
        this.allFavoriteLyrics.splice(globalIndex, 1);
      }

      // 如果是当前歌曲，也要更新本地状态
      if (favoriteItem.songId === (this.currentSong.id || 'current-song')) {
        this.enhancedLyrics[favoriteItem.index].isFavorite = false;
        this.enhancedLyrics[favoriteItem.index].isHighlighted = false;
      }

      this.saveLyricsState();
    },

    closeFavoritesModal() {
      this.showFavoritesModal = false;
    },

    // 保存歌词状态到本地存储
    saveLyricsState() {
      const state = {
        songId: this.currentSong.id || 'current-song',
        lyrics: this.enhancedLyrics.map(lyric => ({
          text: lyric.text,
          isHighlighted: lyric.isHighlighted,
          isFavorite: lyric.isFavorite
        })),
        allFavorites: this.allFavoriteLyrics
      };

      localStorage.setItem('lyricsState', JSON.stringify(state));
    },

    // 加载歌词状态
    loadLyricsState() {
      try {
        const saved = localStorage.getItem('lyricsState');
        if (saved) {
          const state = JSON.parse(saved);
          this.allFavoriteLyrics = state.allFavorites || [];

          // 如果是同一首歌，恢复状态
          if (state.songId === (this.currentSong.id || 'current-song')) {
            state.lyrics.forEach((savedLyric, index) => {
              if (this.enhancedLyrics[index]) {
                this.enhancedLyrics[index].isHighlighted = savedLyric.isHighlighted;
                this.enhancedLyrics[index].isFavorite = savedLyric.isFavorite;
              }
            });
          }
        }
      } catch (error) {
        console.error('加载歌词状态失败:', error);
      }
    },

    togglePlay() {
      // 先处理CD旋转，避免延迟
      if (!this.isPlaying) {
        // 将要开始播放
        this.startCdRotation();
      } else {
        // 将要暂停播放
        this.pauseCdRotation();
      }

      // 然后更新播放状态
      this.isPlaying = !this.isPlaying;

      if (this.isPlaying) {
        this.startPlayback();
      } else {
        this.pausePlayback();
      }
    },

    // CD相关方法
    chooseCD(openState) {
      if (openState) {
        this.isReadyPlay = true;
      }

      if (this.isReadyPlay) {
        this.togglePlay();
      }
    },

    // CD相关方法
    startCdRotation() {
      // 确保任何现有的旋转计时器都被清除
      this.pauseCdRotation();

      // 创建新的旋转计时器
      this.cdRotationTimer = setInterval(() => {
        if (this.isPlaying) {
          // 只有在播放状态才增加角度
          this.cdRotationAngle = (this.cdRotationAngle + 1) % 360;
        }
      }, 10);
    },

    pauseCdRotation() {
      // 立即清除计时器
      if (this.cdRotationTimer) {
        clearInterval(this.cdRotationTimer);
        this.cdRotationTimer = null;
      }
    },

    updateProgress() {
      this.progressPercentage = (this.currentTime / this.totalTime) * 100;
    },

    updateCurrentLyric() {
      for (let i = this.lyrics.length - 1; i >= 0; i--) {
        if (this.currentTime >= this.lyrics[i].time) {
          this.currentLyricIndex = i;
          this.scrollToCurrentLyric();
          break;
        }
      }
    },

    scrollToCurrentLyric() {
      this.$nextTick(() => {
        if (this.$refs.lyricLines && this.$refs.lyricLines[this.currentLyricIndex]) {
          const activeLyric = this.$refs.lyricLines[this.currentLyricIndex];
          const container = this.$refs.lyricsContainer;

          container.scrollTop = activeLyric.offsetTop - container.offsetHeight / 2;
        }
      });
    },

    prevSong() {
      // 这里可以实现切换上一首歌的逻辑
      this.resetPlayback();
    },

    nextSong() {
      // 这里可以实现切换下一首歌的逻辑
      this.resetPlayback();
    },

    startPlayback() {
      if (this.timer) clearInterval(this.timer);

      this.timer = setInterval(() => {
        this.currentTime += 1;
        this.updateProgress();
        this.updateCurrentLyric();

        if (this.currentTime >= this.totalTime) {
          this.nextSong();
        }
      }, 1000);
    },

    pausePlayback() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
    },

    resetPlayback() {
      this.currentTime = 0;
      this.progressPercentage = 0;
      this.currentLyricIndex = 0;

      if (this.isPlaying) {
        this.pausePlayback();
        this.startPlayback();
      }
    },

    formatTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins}:${secs < 10 ? '0' : ''}${secs}`;
    }
  }
};
</script>

<style scoped>
.music-page-container {
  display: flex;
  width: 100%;
  height: 100vh;
  background: #e8ecf3;
  overflow: hidden;
}

/* 左侧歌词部分 */
.lyrics-container {
  flex: 1;
  padding: 30px;
  background-color: rgba(255, 255, 255, 0.9);
  display: flex;
  flex-direction: column;
  max-width: 40%;
  overflow-y: auto;
}

.lyrics-header {
  margin-bottom: 30px;
}

.lyrics-header h2 {
  font-size: 28px;
  margin-bottom: 8px;
  color: #333;
}

.artist {
  font-size: 18px;
  color: #666;
}

.lyrics-content {
  flex: 1;
  overflow-y: auto;
  padding-right: 15px;
}

.lyrics-content p {
  margin: 18px 0;
  font-size: 16px;
  color: #666;
  transition: all 0.3s ease;
  line-height: 1.5;
}

.lyrics-content .active-lyric {
  font-size: 18px;
  color: #990055;
  font-weight: bold;
}

/* 右侧播放器部分 */
.player-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  padding: 40px;
  position: relative;
}

/* 增加一个包装容器来控制整体位置 */
.player-visual-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 60px;
  margin-top: 150px;
}

/* 专辑封面和唱片区域 */
.player-visual {
  position: relative;
  width: 550px; /* 增加宽度以容纳更大的元素 */
  height: 350px;
}

.album-cover {
  width: 350px; /* 略微放大专辑封面 */
  height: 350px;
  position: absolute;
  left: 0;
  top: 0;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  z-index: 2;
  overflow: hidden;
}

.album-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.album-name {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  padding: 15px;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.7));
  color: white;
}

.album-name h3 {
  font-size: 20px;
  margin: 0 0 5px 0;
}

.album-name p {
  font-size: 14px;
  margin: 0;
  opacity: 0.9;
}

/* CD增强旋转效果的CSS部分 */
.cd-disc {
  position: absolute;
  width: 320px;
  height: 320px;
  top: 15px;
  right: 0;
  background-image: url("@/assets/album_cover/CD_shadow_darker.png");
  background-size: 90%;
  background-repeat: no-repeat;
  background-position: center;
  backface-visibility: hidden;
  z-index: 0;
  transition: 1s ease-in-out;

  /* 添加可视化增强元素 */
  overflow: visible;
}

/* 增加CD上的视觉标记，让旋转更明显 */
.cd-disc::before {
  content: '';
  position: absolute;
  top: 13%;
  left: 13%;
  width: 74%;
  height: 74%;
  border-radius: 50%;
  background: linear-gradient(45deg, transparent, transparent 49%, rgba(255, 255, 255, 0.5) 50%, transparent 51%);
  z-index: 1;
}

/* CD中心红色区域增强 */
.cd-disc::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 30%;
  height: 30%;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(179, 198, 230, 0.6) 0%, rgba(140, 166, 217, 0.6) 70%, rgba(102, 134, 204, 0.6) 100%);
  box-shadow: 0 0 10px rgba(255, 0, 0, 0.3);
  z-index: 2;
}

/* CD上的刻痕效果 */
.cd-grooves {
  position: absolute;
  top: 20%;
  left: 20%;
  width: 60%;
  height: 60%;
  border-radius: 50%;
  background: repeating-radial-gradient(
      circle,
      rgba(0, 0, 0, 0.1) 0px,
      rgba(0, 0, 0, 0.1) 1px,
      transparent 1px,
      transparent 3px
  );
  z-index: 1;
  pointer-events: none;
}

/* CD反光效果 */
.cd-reflection {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: linear-gradient(120deg, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0.3) 50%, rgba(255, 255, 255, 0) 51%);
  transform: rotate(45deg);
  z-index: 3;
  pointer-events: none;
}

.cd-disc.readyPlay {
  transform: translateX(30px);
  /* 不再包含rotateZ变换，由JavaScript动态控制 */
}

/* 播放控制区域 */
.player-controls {
  width: 80%;
  max-width: 500px;
  position: relative;
}

.time-display {
  text-align: left;
  margin-bottom: 5px;
  font-size: 16px;
  color: #666;
}

.progress-bar {
  position: relative;
  height: 6px;
  background-color: #ddd;
  border-radius: 3px;
  cursor: pointer;
  margin-bottom: 25px;
}

.progress-bar-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border-radius: 3px;
}

.progress-bar-fill {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: #990055;
  border-radius: 3px;
  transition: width 0.1s linear;
}

.progress-handle {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 14px;
  height: 14px;
  background-color: #fff;
  border: 2px solid #990055;
  border-radius: 50%;
  cursor: pointer;
}

.control-buttons {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.control-buttons button {
  background: none;
  border: none;
  cursor: pointer;
  transition: transform 0.2s;
  padding: 10px;
}

.control-buttons button:hover {
  transform: scale(1.1);
}

.prev-btn, .next-btn {
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.play-btn {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: #990055;
  margin: 0 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.play-icon {
  width: 0;
  height: 0;
  border-top: 15px solid transparent;
  border-bottom: 15px solid transparent;
  border-left: 20px solid white;
  margin-left: 5px;
}

.pause-icon {
  width: 20px;
  height: 20px;
  position: relative;
}

.pause-icon::before, .pause-icon::after {
  content: '';
  position: absolute;
  width: 6px;
  height: 20px;
  background-color: white;
  top: 0;
}

.pause-icon::before {
  left: 4px;
}

.pause-icon::after {
  right: 4px;
}

.prev-icon {
  width: 0;
  height: 0;
  border-top: 12px solid transparent;
  border-bottom: 12px solid transparent;
  border-right: 18px solid #990055;
}

.next-icon {
  width: 0;
  height: 0;
  border-top: 12px solid transparent;
  border-bottom: 12px solid transparent;
  border-left: 18px solid #990055;
}
/* 歌词控制栏 */
.lyrics-controls {
  display: flex;
  gap: 12px;
  margin-top: 15px;
  padding-bottom: 15px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

.highlight-mode-btn,
.show-favorites-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 14px;
  color: #666;
}

.highlight-mode-btn:hover,
.show-favorites-btn:hover {
  background: #f5f5f5;
  transform: translateY(-1px);
}

.highlight-mode-btn.active {
  background: linear-gradient(135deg, #ff9a56, #ffad56);
  color: white;
  border-color: #ff9a56;
  box-shadow: 0 4px 15px rgba(255, 154, 86, 0.3);
}

.show-favorites-btn.active {
  background: linear-gradient(135deg, #ff6b6b, #ff8787);
  color: white;
  border-color: #ff6b6b;
}

.highlight-icon,
.heart-icon {
  font-size: 16px;
}

.favorite-count {
  font-size: 12px;
  opacity: 0.8;
}

/* 歌词行样式增强 */
.lyric-line {
  position: relative;
  margin: 18px 0;
  padding: 12px 16px;
  border-radius: 8px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: default;
}

.lyric-line.clickable {
  cursor: pointer;
}

.lyric-line.clickable:hover {
  background: rgba(255, 154, 86, 0.1);
  transform: translateX(8px);
}

/* 当前播放歌词样式 */
.lyric-line.active-lyric {
  background: linear-gradient(135deg, rgba(153, 0, 85, 0.1), rgba(153, 0, 85, 0.05));
  border-left: 4px solid #990055;
  font-weight: bold;
  color: #990055;
  transform: scale(1.02);
}

/* 点亮歌词样式 */
.lyric-line.highlighted-lyric {
  background: linear-gradient(135deg, rgba(255, 154, 86, 0.2), rgba(255, 173, 86, 0.1));
  border: 2px solid rgba(255, 154, 86, 0.5);
  box-shadow: 0 4px 20px rgba(255, 154, 86, 0.2);
}

/* 收藏歌词样式 */
.lyric-line.favorite-lyric {
  background: linear-gradient(135deg, rgba(255, 107, 107, 0.1), rgba(255, 135, 135, 0.05));
  border-left: 4px solid #ff6b6b;
}

/* 同时是当前播放和收藏的歌词 */
.lyric-line.active-lyric.favorite-lyric {
  background: linear-gradient(135deg, rgba(153, 0, 85, 0.15), rgba(255, 107, 107, 0.1));
  border-left: 4px solid;
  border-image: linear-gradient(45deg, #990055, #ff6b6b) 1;
}

.lyric-text {
  position: relative;
  z-index: 2;
  line-height: 1.6;
}

/* 收藏指示器 */
.favorite-indicator {
  position: absolute;
  top: 8px;
  right: 12px;
  z-index: 3;
}

.favorite-star {
  font-size: 14px;
  animation: twinkle 2s infinite;
}

@keyframes twinkle {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.7; transform: scale(1.1); }
}

/* 点亮效果覆盖层 */
.highlight-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg,
  rgba(255, 215, 0, 0.1),
  rgba(255, 154, 86, 0.1),
  rgba(255, 215, 0, 0.1)
  );
  border-radius: 8px;
  animation: shimmer 3s infinite;
  pointer-events: none;
}

@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

/* 悬浮提示 */
.lyric-tooltip {
  position: absolute;
  top: -35px;
  left: 50%;
  transform: translateX(-50%);
  padding: 6px 12px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  border-radius: 4px;
  font-size: 12px;
  white-space: nowrap;
  z-index: 10;
  animation: fadeInUp 0.3s ease;
}

.lyric-tooltip::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border: 5px solid transparent;
  border-top-color: rgba(0, 0, 0, 0.8);
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

/* 歌词统计 */
.lyrics-stats {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 0;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
  margin-top: 20px;
}

.stats-text {
  font-size: 14px;
  color: #666;
}

.export-favorites-btn {
  padding: 6px 12px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 12px;
}

.export-favorites-btn:hover {
  background: #f5f5f5;
  transform: translateY(-1px);
}

/* 收藏弹窗 */
.favorites-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

.favorites-modal {
  width: 90%;
  max-width: 600px;
  max-height: 80vh;
  background: white;
  border-radius: 16px;
  overflow: hidden;
  animation: slideUp 0.3s ease;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #eee;
}

.modal-header h3 {
  margin: 0;
  color: #333;
}

.close-btn {
  width: 32px;
  height: 32px;
  border: none;
  background: #f5f5f5;
  border-radius: 50%;
  cursor: pointer;
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  max-height: 60vh;
  overflow-y: auto;
  padding: 0;
}

.favorite-lyric-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #f5f5f5;
  transition: background 0.3s ease;
}

.favorite-lyric-item:hover {
  background: #f9f9f9;
}

.lyric-info {
  flex: 1;
}

.lyric-info .lyric-text {
  display: block;
  font-size: 16px;
  color: #333;
  margin-bottom: 4px;
}

.lyric-info .song-info {
  font-size: 12px;
  color: #999;
}

.remove-favorite-btn {
  width: 36px;
  height: 36px;
  border: none;
  background: none;
  cursor: pointer;
  border-radius: 50%;
  transition: background 0.3s ease;
}

.remove-favorite-btn:hover {
  background: #ffebee;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 响应式调整 */
@media (max-width: 768px) {
  .lyrics-controls {
    flex-direction: column;
    gap: 8px;
  }

  .highlight-mode-btn,
  .show-favorites-btn {
    justify-content: center;
  }

  .favorites-modal {
    width: 95%;
    margin: 20px;
  }
}
</style>