<template>
  <div class="body">
    <el-container class="page">
      <!-- 使用新的 MovieHeader 组件 -->
      <MovieHeader
          :disabled="!isPlaying"
          @emoji-action="handleEmojiAction"
      />

      <el-container class="container">
        <el-aside class="aside_left" order=-1></el-aside>
        <el-container class="main_content">
          <!--电影日历-->
          <el-main class="movies_calendar">
            <el-header class="movies_name">{{movie_name}}</el-header>
            <!-- 电影播放区域 -->
            <el-container style="display: flex; flex-direction: row; justify-content: space-between; align-items: center; overflow: visible;">
              <!-- 使用新的 MoviePoster 组件 -->
              <MoviePoster
                  :poster-image="posterimg_src"
                  :dialog="movies_dialog"
                  :date="current_date"
                  :loading="false"
                  :disabled="false"
                  @click="showPlayer"
                  @color-extracted="handleColorExtracted"
              />

              <!-- 使用重构后的播放器组件 -->
              <div v-if="playerVisible" class="player-section">
                <!-- 简单使用 - 直接使用 MoviesPlayer -->
                <MoviesPlayer
                    v-if="useSimplePlayer"
                    :title="movie_name"
                    :source="videoSource"
                    :emoji-list="emojiList"
                    @fullscreen-change="handlePlayerFullscreenChange"
                    @time-update="handleVideoTimeUpdate"
                    @play-state-change="handleVideoPlayStateChange"
                    @emoji-selected="handleEmojiSelected"
                    @danmu-sent="handleDanmuSent"
                    @danmu-toggle="handleDanmuToggle"
                    ref="videoPlayer"
                />

                <!-- 复杂使用 - 使用 VideoPlayerSection 包装器 -->
                <VideoPlayerSection
                    v-else
                    :visible="playerVisible"
                    :loading="playerLoading"
                    :player-title="movie_name"
                    :video-src="videoSource"
                    :danmu-enabled="danmuEnabled"
                    :current-user-id="user_id"
                    :show-usernames="true"
                    :show-video-info="true"
                    :show-comments="false"
                    :show-recommendations="false"
                    :view-count="viewCount"
                    :upload-time="uploadTime"
                    :like-count="likeCount"
                    :comment-count="commentCount"
                    :duration="videoDuration"
                    :emoji-list="emojiList"
                    @fullscreen-change="handlePlayerFullscreenChange"
                    @time-update="handleVideoTimeUpdate"
                    @play-state-change="handleVideoPlayStateChange"
                    @emoji-selected="handleEmojiSelected"
                    @danmu-sent="handleDanmuSent"
                    @danmu-toggle="handleDanmuToggle"
                    @load-danmu="loadDanmuAtTime"
                    @like-video="handleLikeVideo"
                    @collect-video="handleCollectVideo"
                    @share-video="handleShareVideo"
                    ref="videoPlayerSection"
                >
                  <!-- 自定义视频操作按钮 -->
                  <template #video-actions>
                    <el-button size="small" type="primary" @click="handleLikeVideo">
                      <el-icon><ThumbsUp /></el-icon>
                      点赞 {{ likeCount }}
                    </el-button>
                    <el-button size="small" @click="handleCollectVideo">
                      <el-icon><Star /></el-icon>
                      收藏
                    </el-button>
                    <el-button size="small" @click="handleShareVideo">
                      <el-icon><Share /></el-icon>
                      分享
                    </el-button>
                    <el-button size="small" @click="handleFollowUploader">
                      <el-icon><Plus /></el-icon>
                      关注UP主
                    </el-button>
                  </template>
                </VideoPlayerSection>
              </div>
            </el-container>
          </el-main>

          <!-- 每周电影 - 使用新的 MovieCarousel 组件 -->
          <MovieCarousel
              title="每周电影"
              subtitle="发现更多精彩内容"
              :auto-play-interval="3000"
              @upload-success="handleCarouselUploadSuccess"
              @upload-error="handleCarouselUploadError"
              @item-click="handleCarouselItemClick"
          />

          <!-- 弹幕输入区 - 使用新的 DanmuInputSection 组件 -->
          <!-- 注意：重构后的播放器已经包含弹幕输入功能，这个组件可能不再需要 -->
          <DanmuInputSection
              v-if="showExternalDanmuInput && !useSimplePlayer"
              :visible="playerVisible && isPlaying && !isPlayerFullscreen"
              :disabled="!danmuEnabled"
              placeholder="发送弹幕评论..."
              :cooldown-time="2000"
              @send-text="handleDanmuText"
              @send-emoji="handleDanmuEmoji"
              @color-change="handleDanmuColorChange"
          />
        </el-container>

        <el-aside class="aside_right"></el-aside>
      </el-container>
      <el-footer class="footer"></el-footer>
    </el-container>
  </div>
</template>

<script>
import DanmuService from "@/utils/danmuService.js";
import eventBus from '@/utils/eventBus.js';
// 导入组件
import MovieHeader from '@/components/MovieHeader.vue';
import MoviePoster from '@/components/MoviePoster.vue';
import DanmuInputSection from '@/components/DanmuInputSection.vue';
import MovieCarousel from '@/components/MovieCarousel.vue';
import VideoPlayerSection from '@/components/VideoPlayerSection.vue';
import MoviesPlayer from '@/components/MoviesPlayer.vue';
// 导入图标
import { ThumbsUp, Star, Share, Plus } from '@element-plus/icons-vue';

export default {
  name: 'MoviesPage',
  components: {
    MovieHeader,
    MoviePoster,
    DanmuInputSection,
    MovieCarousel,
    VideoPlayerSection,
    MoviesPlayer,
    // 注册图标
    ThumbsUp,
    Star,
    Share,
    Plus
  },

  data() {
    return {
      user_id: "bear",
      movie_name: "我的天才女友",
      movies_dialog: '"' + '要事先规划好自己的行动，知道自己在做什么，这样就能预测后果。' + '"',
      current_date: '',
      posterimg_src: require("@/assets/stagephotos/我的天才女友01.jpg"),

      // 播放器配置
      useSimplePlayer: true, // 是否使用简单播放器（true: MoviesPlayer, false: VideoPlayerSection）
      showExternalDanmuInput: false, // 是否显示外部弹幕输入组件
      videoSource: 'https://media.w3.org/2010/05/sintel/trailer.mp4', // 视频源

      // 播放器状态
      playerVisible: true,
      playerLoading: false,
      isPlayerFullscreen: false,

      // 视频信息
      viewCount: '1.2万',
      uploadTime: '2天前',
      likeCount: '8.8万',
      commentCount: '666',
      videoDuration: 0,

      // 弹幕相关数据
      danmuEnabled: true,
      lastSentEmoji: null,
      lastSentTime: 0,
      emojiList: [
        { name: 'laugh', image: '/emojis/laugh.png' },
        { name: 'angry', image: '/emojis/angry.png' },
        { name: 'shocked', image: '/emojis/shocked.png' },
        { name: 'think', image: '/emojis/think.png' },
        { name: 'touching', image: '/emojis/touching.png' },
        { name: 'relief', image: '/emojis/relief.png' },
        { name: 'boring', image: '/emojis/boring.png' }
      ],

      // 播放状态
      isPlaying: false,
      currentPlayTime: 0,

      // 弹幕服务
      danmuService: null
    }
  },

  mounted() {
    this.current_date = this.getFormattedDate();
    this.initDanmuService();
    this.handleShareParameters();
    this.checkResumePlayback();
  },

  beforeUnmount() {
    this.savePlaybackProgress();
    if (this.danmuService) {
      this.danmuService.disconnect();
    }
  },

  methods: {
    // 获取当前播放器引用
    getCurrentPlayer() {
      return this.useSimplePlayer ? this.$refs.videoPlayer : this.$refs.videoPlayerSection;
    },

    // 【新增】处理来自 MovieCarousel 的事件
    handleCarouselUploadSuccess({response, file, index, imageUrl}) {
      console.log('轮播图上传成功：', {response, file, index, imageUrl});
    },

    handleCarouselUploadError({error, file, index}) {
      console.error('轮播图上传失败：', {error, file, index});
    },

    handleCarouselItemClick({item, index}) {
      console.log('轮播图项目点击：', {item, index});
      // 可以在这里切换到对应的电影
      // this.switchToMovie(item.movieName, item.videoSource);
    },

    // 切换电影
    switchToMovie(movieName, videoSource) {
      this.movie_name = movieName;
      this.videoSource = videoSource;
      this.showPlayer();
    },

    // 【新增】处理来自 DanmuInputSection 的弹幕发送事件
    handleDanmuText(danmuData) {
      const completeDanmu = {
        type: 'text',
        content: danmuData.content,
        userId: this.user_id,
        username: this.user_id,
        videoTime: this.currentPlayTime,
        color: danmuData.color
      };

      this.sendDanmuToServer(completeDanmu);
    },

    handleDanmuEmoji(danmuData) {
      const completeDanmu = {
        type: 'emoji',
        content: danmuData.content,
        userId: this.user_id,
        username: this.user_id,
        videoTime: this.currentPlayTime,
        color: danmuData.color
      };

      this.sendDanmuToServer(completeDanmu);
    },

    handleDanmuColorChange(color) {
      console.log('弹幕颜色改变为：', color);
    },

    // 【新增】处理来自 MoviePoster 的颜色提取事件
    handleColorExtracted(color) {
      console.log('海报主色调已提取：', color);
    },

    // 【新增】处理来自 MovieHeader 的表情操作事件
    handleEmojiAction({emoji, action}) {
      console.log(`表情操作: ${emoji} - ${action}`);

      if (action === '发送表情') {
        this.sendEmojiDanmu(emoji);
      } else if (action === '收藏这一幕') {
        this.bookmarkCurrentScene(emoji);
      } else if (action === '分享这一幕') {
        this.shareCurrentScene(emoji);
      }
    },

    // 【新增】处理视频操作事件
    handleLikeVideo() {
      // 更新点赞数
      const currentLikes = parseFloat(this.likeCount);
      this.likeCount = (currentLikes + 0.1).toFixed(1) + '万';

      this.$message.success('点赞成功！');
    },

    handleCollectVideo() {
      // 收藏视频逻辑
      const bookmark = {
        movieName: this.movie_name,
        time: this.currentPlayTime,
        timestamp: new Date().toISOString(),
        poster: this.posterimg_src
      };

      const bookmarks = JSON.parse(localStorage.getItem('videoBookmarks') || '[]');
      bookmarks.push(bookmark);
      localStorage.setItem('videoBookmarks', JSON.stringify(bookmarks));

      this.$message.success('已添加到收藏夹');
    },

    handleShareVideo() {
      const shareUrl = `${window.location.origin}${window.location.pathname}?movie=${encodeURIComponent(this.movie_name)}&time=${this.currentPlayTime}`;

      navigator.clipboard.writeText(shareUrl).then(() => {
        this.$message.success('分享链接已复制到剪贴板');
      }).catch(() => {
        this.$message.error('无法复制分享链接');
      });
    },

    handleFollowUploader() {
      this.$message.success('已关注UP主');
    },

    // 获取格式化日期
    getFormattedDate() {
      const today = new Date();
      const year = today.getFullYear();
      const month = String(today.getMonth() + 1).padStart(2, '0');
      const day = String(today.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    },

    // 初始化弹幕服务
    initDanmuService() {
      const apiUrl = process.env.VUE_APP_DANMU_API_URL || '/api/danmu';
      this.danmuService = new DanmuService({
        apiUrl,
        movieId: this.getMovieId(),
        userId: this.user_id,
        onReceiveDanmu: (danmu) => {
          eventBus.emit('new-danmu', danmu);
        }
      });

      this.danmuService.connect().catch(err => {
        console.error('连接弹幕服务失败:', err);
        this.$message.error('连接弹幕服务失败，部分功能可能无法使用');
      });
    },

    getMovieId() {
      return encodeURIComponent(this.movie_name);
    },

    // 显示播放器
    showPlayer() {
      this.playerVisible = true;
      this.playerLoading = true;

      setTimeout(() => {
        this.playerLoading = false;
        // 播放器显示后可以触发自动播放
        const player = this.getCurrentPlayer();
        if (player && player.play) {
          player.play();
        }
      }, 300);
    },

    // 隐藏播放器
    hidePlayer() {
      this.playerVisible = false;
      this.isPlaying = false;
      this.isPlayerFullscreen = false;
    },

    // 处理播放器事件
    handlePlayerFullscreenChange(isFullscreen) {
      this.isPlayerFullscreen = isFullscreen;
      console.log('播放器全屏状态：', isFullscreen);
    },

    handleEmojiSelected(emojiName) {
      this.sendEmojiDanmu(emojiName);
    },

    handleDanmuSent(danmu) {
      const completeDanmu = {
        type: 'text',
        content: danmu.content,
        userId: this.user_id,
        username: this.user_id,
        videoTime: this.currentPlayTime,
        color: danmu.color
      };
      this.sendDanmuToServer(completeDanmu);
    },

    handleDanmuToggle(enabled) {
      this.danmuEnabled = enabled;
    },

    handleVideoTimeUpdate(currentTime) {
      this.currentPlayTime = currentTime;
      if (this.danmuService && this.isPlaying) {
        this.danmuService.onTimeUpdate(currentTime);
      }
    },

    handleVideoPlayStateChange(isPlaying) {
      this.isPlaying = isPlaying;
    },

    // 发送表情弹幕
    sendEmojiDanmu(emojiName) {
      const now = Date.now();
      if (this.lastSentEmoji === emojiName && now - this.lastSentTime < 2000) {
        return;
      }

      this.lastSentEmoji = emojiName;
      this.lastSentTime = now;

      const emojiUnicode = this.getEmojiUnicode(emojiName);

      const danmu = {
        type: 'emoji',
        content: emojiUnicode,
        userId: this.user_id,
        username: this.user_id,
        videoTime: this.currentPlayTime,
        color: '#FFFFFF'
      };

      this.sendDanmuToServer(danmu);
    },

    // 发送弹幕到服务器
    sendDanmuToServer(danmu) {
      if (this.danmuService) {
        this.danmuService.sendDanmu(danmu)
            .then(() => {
              this.$message({
                message: '弹幕发送成功',
                type: 'success',
                duration: 1000
              });
            })
            .catch(err => {
              console.error('发送弹幕失败:', err);
              this.$message.error('发送弹幕失败，请重试');
              eventBus.emit('new-danmu', danmu);
            });
      } else {
        eventBus.emit('new-danmu', danmu);
        this.$message({
          message: '弹幕发送成功(本地模式)',
          type: 'success',
          duration: 1000
        });
      }
    },

    // 在指定时间点加载弹幕
    loadDanmuAtTime(time) {
      if (!this.danmuService) {
        return;
      }
      this.danmuService.onTimeUpdate(time);
    },

    // 收藏当前场景
    bookmarkCurrentScene(emojiName) {
      const bookmark = {
        movieName: this.movie_name,
        time: this.currentPlayTime,
        timestamp: new Date().toISOString(),
        emoji: emojiName,
        screenshot: null
      };

      const bookmarks = JSON.parse(localStorage.getItem('movieBookmarks') || '[]');
      bookmarks.push(bookmark);
      localStorage.setItem('movieBookmarks', JSON.stringify(bookmarks));

      this.$message({
        message: `已收藏电影 ${this.movie_name} 在 ${this.formatTime(this.currentPlayTime)} 的场景`,
        type: 'success'
      });
    },

    // 分享当前场景
    shareCurrentScene(emojiName) {
      const shareUrl = `${window.location.origin}${window.location.pathname}?movie=${encodeURIComponent(this.movie_name)}&time=${this.currentPlayTime}&emoji=${emojiName}`;

      navigator.clipboard.writeText(shareUrl).then(() => {
        this.$message({
          message: '分享链接已复制到剪贴板',
          type: 'success'
        });
      }).catch(err => {
        console.error('复制失败:', err);
        this.$message.error('无法复制分享链接');
      });
    },

    // 格式化时间显示
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins}:${secs < 10 ? '0' : ''}${secs}`;
    },

    // 根据表情名称获取对应的Unicode
    getEmojiUnicode(name) {
      const emojiMap = {
        'laugh': '😂',
        'angry': '😡',
        'shocked': '😱',
        'think': '🤔',
        'touching': '🥹',
        'relief': '😌',
        'boring': '😒'
      };
      return emojiMap[name] || '👍';
    },

    // 处理分享参数
    handleShareParameters() {
      const urlParams = new URLSearchParams(window.location.search);
      const movieParam = urlParams.get('movie');
      const timeParam = urlParams.get('time');
      const emojiParam = urlParams.get('emoji');

      if (movieParam) {
        this.movie_name = decodeURIComponent(movieParam);
      }

      if (timeParam) {
        setTimeout(() => {
          const player = this.getCurrentPlayer();
          if (player) {
            player.seek && player.seek(parseFloat(timeParam));
            this.currentPlayTime = parseFloat(timeParam);
            this.showPlayer();
          }
        }, 1000);
      }

      if (emojiParam) {
        setTimeout(() => {
          this.sendEmojiDanmu(emojiParam);
        }, 1500);
      }
    },

    // 保存播放进度
    savePlaybackProgress() {
      const currentTime = this.currentPlayTime;

      if (currentTime < 5) {
        return;
      }

      const progressData = {
        movie: this.movie_name,
        time: currentTime,
        timestamp: new Date().toISOString()
      };

      sessionStorage.setItem('movieProgress', JSON.stringify(progressData));
    },

    // 检查是否有断点续播
    checkResumePlayback() {
      const savedProgress = sessionStorage.getItem('movieProgress');
      if (!savedProgress) {
        return;
      }

      try {
        const progressData = JSON.parse(savedProgress);
        const savedDate = new Date(progressData.timestamp);
        const now = new Date();
        const hoursDiff = (now - savedDate) / (1000 * 60 * 60);

        if (progressData.movie === this.movie_name && hoursDiff < 24) {
          this.$confirm(
              `是否从上次观看的位置（${this.formatTime(progressData.time)}）继续播放？`,
              '断点续播',
              {
                confirmButtonText: '继续播放',
                cancelButtonText: '从头开始',
                type: 'info'
              }
          ).then(() => {
            this.showPlayer();
            setTimeout(() => {
              const player = this.getCurrentPlayer();
              if (player && player.seek) {
                player.seek(progressData.time);
                this.currentPlayTime = progressData.time;
              }
            }, 500);
          }).catch(() => {
            this.showPlayer();
          });
        }
      } catch (error) {
        console.error('解析保存的进度数据出错:', error);
        sessionStorage.removeItem('movieProgress');
      }
    },

    // 播放器控制方法
    togglePlay() {
      const player = this.getCurrentPlayer();
      if (player) {
        if (this.isPlaying) {
          player.pause && player.pause();
        } else {
          player.play && player.play();
        }
      }
    },

    seekTo(time) {
      const player = this.getCurrentPlayer();
      if (player && player.seek) {
        player.seek(time);
      }
    }
  }
};
</script>

<style scoped>
.body {
  cursor: url("@/assets/film/bg_rmv/resized/clapperboard_on.png"), auto;
}

.body:active {
  cursor: url("@/assets/film/bg_rmv/resized/clapperboard_off.png"), auto;
}

.page {
  height: 150vh;
}

.container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  height: 100%;
  width: 100%;
}

.main_content {
  display: flex;
  flex-direction: column;
  width: 80%;
  height: 100%;
  overflow: visible !important;
  min-width: 0;
}

.movies_calendar {
  display: flex;
  flex-direction: column;
  height: 50vh;
  width: 100%;
  position: relative;
  overflow: visible !important;
}

.movies_name {
  font-size: 5vh;
  margin-bottom: 2vh;
}

/* 播放器区域样式 */
.player-section {
  width: 580px;
  max-width: 100%;
  position: relative;
}

.aside_left, .aside_right {
  background-image: url("@/assets/page_animation/clip_film_rolling_adventure time_long.png");
  background-size: 40vw 110%;
  background-repeat: repeat-y;
  background-position: 50% 50%;
  color: #333;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  width: 15%;
}

.footer {
  display: flex;
  background-color: #ffffff;
  color: #ffffff;
  justify-content: center;
  align-items: center;
}

/* 字体定义 */
@font-face {
  font-family: 楷体;
  src: local(楷体);
  unicode-range: U+4E00-9FFF;
}

@font-face {
  font-family: 'Brush Script MT';
  src: local('Brush Script MT');
  unicode-range: U+0020-007F;
}

.aside_left {
  animation: scrollBackground_left 10s linear infinite;
  animation-play-state: running;
}

.aside_left:hover {
  animation-play-state: running;
  cursor: pointer;
}

.aside_right {
  animation: scrollBackground_right 10s linear infinite;
  animation-play-state: running;
}

.aside_right:hover {
  animation-play-state: running;
  cursor: pointer;
}

@keyframes scrollBackground_left {
  0% {
    background-position: 50% 50%;
  }
  100% {
    background-position: 50% -1050%;
  }
}

@keyframes scrollBackground_right {
  0% {
    background-position: 50% 50%;
  }
  100% {
    background-position: 50% 1150%;
  }
}

.movies_calendar:not(.fullscreen) {
  display: flex !important;
  flex-direction: column !important;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .player-section {
    width: 100%;
    max-width: none;
  }

  .main_content {
    width: 95%;
  }

  .aside_left, .aside_right {
    width: 10%;
  }
}
</style>