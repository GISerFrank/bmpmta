<template>
  <div class="movies_oftheweek" :class="{ 'carousel--loading': loading }">
    <div class="carousel-header">
      <h3 class="carousel-title">{{ title }}</h3>
      <p class="carousel-subtitle">{{ subtitle }}</p>
    </div>

    <el-carousel
        :interval="autoPlayInterval"
        :type="carouselType"
        :height="carouselHeight"
        :style="{ width: carouselWidth }"
        @change="handleSlideChange"
        ref="carousel"
    >
      <el-carousel-item
          v-for="(item, index) in items"
          :key="item.id || index"
          :style="{ width: itemWidth }"
          class="carousel-item"
      >
        <!-- 如果是上传项 -->
        <div v-if="item.type === 'upload'" class="upload-item">
          <el-upload
              class="avatar-uploader"
              :action="uploadAction"
              :show-file-list="false"
              :on-success="(response, file) => handleUploadSuccess(response, file, index)"
              :on-error="(error, file) => handleUploadError(error, file, index)"
              :before-upload="(file) => handleBeforeUpload(file, index)"
              :on-progress="(event, file) => handleUploadProgress(event, file, index)"
              :accept="acceptedTypes"
              :disabled="uploadDisabled"
          >
            <!-- 上传成功显示图片 -->
            <img
                v-if="item.imageUrl"
                :src="item.imageUrl"
                class="avatar"
                :alt="item.title || '上传的图片'"
            />

            <!-- 上传中显示进度 -->
            <div v-else-if="item.uploading" class="upload-progress">
              <el-progress
                  :percentage="item.uploadProgress"
                  type="circle"
                  :width="60"
              />
              <p class="progress-text">上传中...</p>
            </div>

            <!-- 默认上传区域 -->
            <div v-else class="upload-placeholder">
              <el-icon class="avatar-uploader-icon">
                <Plus />
              </el-icon>
              <p class="upload-text">{{ item.uploadText || '点击上传' }}</p>
            </div>

            <template #tip>
              <div class="el-upload__tip">
                {{ item.tipText || '支持 JPG、PNG 格式，大小不超过 2MB' }}
              </div>
            </template>
          </el-upload>
        </div>

        <!-- 如果是展示项 -->
        <div v-else-if="item.type === 'display'" class="display-item">
          <div class="movie-card" @click="handleItemClick(item, index)">
            <div class="movie-poster">
              <img
                  :src="item.poster"
                  :alt="item.title"
                  @load="handleImageLoad(item, index)"
                  @error="handleImageError(item, index)"
              />
              <div class="movie-overlay">
                <h4 class="movie-title">{{ item.title }}</h4>
                <p class="movie-info">{{ item.year }} · {{ item.genre }}</p>
                <div class="movie-rating">
                  <span class="rating-score">{{ item.rating }}</span>
                  <span class="rating-text">分</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 默认空项 -->
        <div v-else class="empty-item">
          <div class="empty-placeholder">
            <el-icon><Picture /></el-icon>
            <p>暂无内容</p>
          </div>
        </div>
      </el-carousel-item>
    </el-carousel>

    <!-- 轮播控制 -->
    <div v-if="showControls" class="carousel-controls">
      <el-button
          @click="previousSlide"
          :disabled="loading"
          size="small"
          circle
      >
        <el-icon><ArrowLeft /></el-icon>
      </el-button>

      <span class="slide-indicator">
        {{ currentSlide + 1 }} / {{ items.length }}
      </span>

      <el-button
          @click="nextSlide"
          :disabled="loading"
          size="small"
          circle
      >
        <el-icon><ArrowRight /></el-icon>
      </el-button>
    </div>
  </div>
</template>

<script>
import { Plus, Picture, ArrowLeft, ArrowRight } from '@element-plus/icons-vue'

export default {
  name: 'MovieCarousel',

  components: {
    Plus,
    Picture,
    ArrowLeft,
    ArrowRight
  },

  props: {
    // 基础配置
    title: {
      type: String,
      default: '每周电影'
    },
    subtitle: {
      type: String,
      default: '发现更多精彩内容'
    },

    // 轮播配置
    autoPlayInterval: {
      type: Number,
      default: 3000
    },
    carouselType: {
      type: String,
      default: 'card'
    },
    carouselHeight: {
      type: String,
      default: '30vh'
    },
    carouselWidth: {
      type: String,
      default: '100vh'
    },
    itemWidth: {
      type: String,
      default: '50vh'
    },

    // 数据
    items: {
      type: Array,
      default: () => [
        { id: 1, type: 'upload', uploadText: '上传电影海报' },
        { id: 2, type: 'upload', uploadText: '上传电影海报' },
        { id: 3, type: 'upload', uploadText: '上传电影海报' },
        { id: 4, type: 'upload', uploadText: '上传电影海报' },
        { id: 5, type: 'upload', uploadText: '上传电影海报' },
        { id: 6, type: 'upload', uploadText: '上传电影海报' }
      ]
    },

    // 上传配置
    uploadAction: {
      type: String,
      default: 'https://run.mocky.io/v3/9d059bf9-4660-45f2-925d-ce80ad6c4d15'
    },
    acceptedTypes: {
      type: String,
      default: 'image/jpeg,image/png,image/jpg'
    },
    maxFileSize: {
      type: Number,
      default: 2 // MB
    },
    uploadDisabled: {
      type: Boolean,
      default: false
    },

    // 控制选项
    showControls: {
      type: Boolean,
      default: true
    },
    loading: {
      type: Boolean,
      default: false
    }
  },

  emits: [
    'item-click',        // 点击项目
    'upload-success',    // 上传成功
    'upload-error',      // 上传失败
    'slide-change',      // 轮播切换
    'image-load',        // 图片加载完成
    'image-error'        // 图片加载失败
  ],

  data() {
    return {
      currentSlide: 0,
      internalItems: []
    }
  },

  computed: {
    // 处理后的项目数组
    processedItems() {
      return this.internalItems.length > 0 ? this.internalItems : this.items
    }
  },

  watch: {
    items: {
      immediate: true,
      deep: true,
      handler(newItems) {
        this.internalItems = newItems.map(item => ({
          ...item,
          uploading: false,
          uploadProgress: 0,
          imageLoaded: false,
          imageError: false
        }))
      }
    }
  },

  methods: {
    // 轮播控制
    nextSlide() {
      if (this.$refs.carousel) {
        this.$refs.carousel.next()
      }
    },

    previousSlide() {
      if (this.$refs.carousel) {
        this.$refs.carousel.prev()
      }
    },

    goToSlide(index) {
      if (this.$refs.carousel) {
        this.$refs.carousel.setActiveItem(index)
      }
    },

    handleSlideChange(current, prev) {
      this.currentSlide = current
      this.$emit('slide-change', { current, prev })
    },

    // 项目点击
    handleItemClick(item, index) {
      this.$emit('item-click', { item, index })
    },

    // 上传处理
    handleBeforeUpload(file, index) {
      // 文件类型检查
      const isValidType = this.acceptedTypes.split(',').some(type =>
          file.type === type.trim()
      )

      if (!isValidType) {
        this.$message.error('只支持 JPG、PNG 格式的图片!')
        return false
      }

      // 文件大小检查
      const isValidSize = file.size / 1024 / 1024 < this.maxFileSize
      if (!isValidSize) {
        this.$message.error(`图片大小不能超过 ${this.maxFileSize}MB!`)
        return false
      }

      // 设置上传状态
      this.setItemState(index, {
        uploading: true,
        uploadProgress: 0
      })

      return true
    },

    handleUploadProgress(event, file, index) {
      const progress = Math.round((event.loaded / event.total) * 100)
      this.setItemState(index, {
        uploadProgress: progress
      })
    },

    handleUploadSuccess(response, file, index) {
      const imageUrl = URL.createObjectURL(file.raw)

      this.setItemState(index, {
        uploading: false,
        uploadProgress: 100,
        imageUrl: imageUrl,
        fileName: file.name,
        response: response
      })

      this.$emit('upload-success', { response, file, index, imageUrl })

      this.$message({
        message: '上传成功!',
        type: 'success',
        duration: 2000
      })
    },

    handleUploadError(error, file, index) {
      this.setItemState(index, {
        uploading: false,
        uploadProgress: 0
      })

      this.$emit('upload-error', { error, file, index })

      this.$message.error('上传失败，请重试!')
      console.error('Upload error:', error)
    },

    // 图片加载处理
    handleImageLoad(item, index) {
      this.setItemState(index, {
        imageLoaded: true,
        imageError: false
      })

      this.$emit('image-load', { item, index })
    },

    handleImageError(item, index) {
      this.setItemState(index, {
        imageLoaded: false,
        imageError: true
      })

      this.$emit('image-error', { item, index })
    },

    // 设置项目状态
    setItemState(index, state) {
      if (index >= 0 && index < this.internalItems.length) {
        Object.assign(this.internalItems[index], state)
      }
    },

    // 重置项目
    resetItem(index) {
      this.setItemState(index, {
        uploading: false,
        uploadProgress: 0,
        imageUrl: '',
        fileName: '',
        response: null,
        imageLoaded: false,
        imageError: false
      })
    },

    // 获取项目数据
    getItemData(index) {
      return this.internalItems[index] || null
    },

    // 批量重置
    resetAllItems() {
      this.internalItems.forEach((item, index) => {
        this.resetItem(index)
      })
    }
  }
}
</script>

<style scoped>
.movies_oftheweek {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 80vh;
  padding: 20px;
  position: relative;
  background: #ffffff;
}

.carousel--loading {
  opacity: 0.7;
  pointer-events: none;
}

/* 轮播标题 */
.carousel-header {
  text-align: center;
  margin-bottom: 30px;
}

.carousel-title {
  font-size: 2.5rem;
  font-weight: bold;
  color: #333;
  margin: 0 0 10px 0;
}

.carousel-subtitle {
  font-size: 1.1rem;
  color: #666;
  margin: 0;
}

/* 轮播项目样式 */
.carousel-item {
  border-radius: 1vh;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

.el-carousel__item:nth-child(2n) {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  opacity: 0.9;
}

.el-carousel__item:nth-child(2n + 1) {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  opacity: 0.9;
}

/* 上传项目样式 */
.upload-item {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.avatar-uploader {
  width: 200px;
  height: 200px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border: 2px dashed rgba(255, 255, 255, 0.5);
  border-radius: 12px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
}

.avatar-uploader:hover {
  border-color: rgba(255, 255, 255, 0.8);
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
}

.avatar {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  border-radius: 10px;
}

.upload-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: white;
}

.avatar-uploader-icon {
  font-size: 48px;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 10px;
}

.upload-text {
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
  margin: 0;
  text-align: center;
}

.upload-progress {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
}

.progress-text {
  color: white;
  margin-top: 10px;
  font-size: 14px;
}

/* 展示项目样式 */
.display-item {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.movie-card {
  width: 200px;
  height: 200px;
  position: relative;
  cursor: pointer;
  border-radius: 12px;
  overflow: hidden;
  transition: all 0.3s ease;
}

.movie-card:hover {
  transform: scale(1.05);
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.3);
}

.movie-poster {
  width: 100%;
  height: 100%;
  position: relative;
}

.movie-poster img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.movie-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.8));
  color: white;
  padding: 15px;
  transform: translateY(100%);
  transition: transform 0.3s ease;
}

.movie-card:hover .movie-overlay {
  transform: translateY(0);
}

.movie-title {
  font-size: 16px;
  margin: 0 0 5px 0;
  font-weight: bold;
}

.movie-info {
  font-size: 12px;
  margin: 0 0 5px 0;
  opacity: 0.8;
}

.movie-rating {
  display: flex;
  align-items: center;
}

.rating-score {
  font-size: 18px;
  font-weight: bold;
  color: #f39c12;
}

.rating-text {
  font-size: 12px;
  margin-left: 2px;
  opacity: 0.8;
}

/* 空项目样式 */
.empty-item {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.empty-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: rgba(255, 255, 255, 0.6);
}

.empty-placeholder .el-icon {
  font-size: 48px;
  margin-bottom: 10px;
}

.empty-placeholder p {
  margin: 0;
  font-size: 14px;
}

/* 轮播控制 */
.carousel-controls {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-top: 20px;
  padding: 10px 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 25px;
  backdrop-filter: blur(10px);
}

.slide-indicator {
  color: #333;
  font-weight: 500;
  font-size: 14px;
  min-width: 60px;
  text-align: center;
}

/* 上传提示 */
.el-upload__tip {
  color: rgba(255, 255, 255, 0.8);
  font-size: 12px;
  text-align: center;
  margin-top: 8px;
  line-height: 1.4;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .movies_oftheweek {
    height: auto;
    min-height: 60vh;
    padding: 15px;
  }

  .carousel-title {
    font-size: 2rem;
  }

  .carousel-subtitle {
    font-size: 1rem;
  }

  .avatar-uploader,
  .movie-card {
    width: 150px;
    height: 150px;
  }

  .carousel-controls {
    gap: 10px;
    padding: 8px 15px;
  }
}

/* 深色主题支持 */
@media (prefers-color-scheme: dark) {
  .carousel-title {
    color: #f0f0f0;
  }

  .carousel-subtitle {
    color: #c0c0c0;
  }

  .slide-indicator {
    color: #f0f0f0;
  }
}
</style>