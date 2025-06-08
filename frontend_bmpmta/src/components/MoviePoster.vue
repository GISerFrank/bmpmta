<template>
  <div
      v-if="dominantColor"
      class="movies_poster"
      @click="handleClick"
      :style="{
      boxShadow: `0 4px 8px 2px ${dominantColor}`,
      backgroundImage: `url(${posterImage})`
    }"
  >
    <p class="movies_dialog">{{ dialog }}</p>
    <p class="movies_date">{{ date }}</p>

    <!-- 加载状态 -->
    <div v-if="loading" class="loading-overlay">
      <div class="loading-spinner"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MoviePoster',

  props: {
    // 电影信息
    posterImage: {
      type: String,
      required: true
    },
    dialog: {
      type: String,
      default: ''
    },
    date: {
      type: String,
      default: ''
    },
    // 控制属性
    loading: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },

  emits: ['click', 'color-extracted'],

  data() {
    return {
      dominantColor: '',
      isHovered: false,
      imageLoaded: false
    }
  },

  computed: {
    posterStyle() {
      return {
        backgroundImage: `url(${this.posterImage})`,
        boxShadow: this.dominantColor
            ? `0 4px 8px 2px ${this.dominantColor}`
            : 'none'
      }
    }
  },

  mounted() {
    this.extractDominantColor()
  },

  watch: {
    posterImage() {
      this.extractDominantColor()
    }
  },

  methods: {
    // 处理点击事件
    handleClick() {
      if (this.disabled || this.loading) {
        return
      }
      this.$emit('click')
    },

    // 提取图片主要颜色
    extractDominantColor() {
      if (!this.posterImage) {
        return
      }

      this.getDominantColor(this.posterImage, (color) => {
        const adjustedColor = this.adjustShadowColor(color)
        this.dominantColor = adjustedColor

        // 通知父组件颜色已提取
        this.$emit('color-extracted', adjustedColor)
      })
    },

    // 获取图片平均颜色
    getDominantColor(imageSrc, callback) {
      const img = new Image()
      img.crossOrigin = "Anonymous"
      img.src = imageSrc

      img.onload = () => {
        try {
          const canvas = document.createElement('canvas')
          const ctx = canvas.getContext('2d')
          canvas.width = img.width
          canvas.height = img.height
          ctx.drawImage(img, 0, 0)

          const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
          const data = imageData.data

          let r = 0, g = 0, b = 0
          let pixelCount = data.length / 4

          for (let i = 0; i < data.length; i += 4) {
            r += data[i]
            g += data[i + 1]
            b += data[i + 2]
          }

          r = Math.floor(r / pixelCount)
          g = Math.floor(g / pixelCount)
          b = Math.floor(b / pixelCount)

          callback(`rgb(${r}, ${g}, ${b})`)
          this.imageLoaded = true
        } catch (error) {
          console.warn('无法提取图片颜色:', error)
          callback('rgba(0, 0, 0, 0.1)') // 默认颜色
        }
      }

      img.onerror = () => {
        console.warn('图片加载失败:', imageSrc)
        callback('rgba(0, 0, 0, 0.1)') // 默认颜色
      }
    },

    // RGB转HSL
    rgbToHsl(r, g, b) {
      r /= 255
      g /= 255
      b /= 255

      const max = Math.max(r, g, b)
      const min = Math.min(r, g, b)
      let h, s, l = (max + min) / 2

      if (max === min) {
        h = s = 0 // achromatic
      } else {
        const d = max - min
        s = l > 0.5 ? d / (2 - max - min) : d / (max + min)
        switch (max) {
          case r:
            h = (g - b) / d + (g < b ? 6 : 0)
            break
          case g:
            h = (b - r) / d + 2
            break
          case b:
            h = (r - g) / d + 4
            break
        }
        h /= 6
      }
      return [h * 360, s * 100, l * 100]
    },

    // 调整阴影颜色
    adjustShadowColor(dominantColor) {
      try {
        const rgb = dominantColor.match(/\d+/g)?.map(Number)
        if (!rgb || rgb.length < 3) {
          return dominantColor
        }

        const [h, s, l] = this.rgbToHsl(rgb[0], rgb[1], rgb[2])
        const newL = Math.min(l + 20, 100)
        return `hsl(${h}, ${s}%, ${newL}%)`
      } catch (error) {
        console.warn('颜色调整失败:', error)
        return dominantColor
      }
    }
  }
}
</script>

<style scoped>
.movies_poster {
  position: relative;
  width: 50%;
  height: 100%;
  cursor: pointer;
  padding: 0;
  overflow: hidden;
  border-radius: 1vh;
  transition: 0.5s ease;
  background-size: cover;
  background-position: left;
  background-repeat: no-repeat;
}

.movies_poster:hover {
  box-shadow: 0 8px 8px 6px v-bind(dominantColor) !important;
}

.movies_poster::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: inherit;
  background-size: cover;
  background-position: left;
  background-repeat: no-repeat;
  opacity: 0.5;
  z-index: -1;
  transition: opacity 0.5s ease, transform 1.0s ease;
}

.movies_poster:hover::before {
  opacity: 1.0;
  transform: translateY(-25%);
}

.movies_dialog {
  position: absolute;
  left: 5%;
  bottom: 10%;
  opacity: 0;
  transition: opacity 1.0s ease-in-out;
  font-family: 楷体, "Brush Script MT", cursive;
  font-size: 2vh;
  color: white;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
  max-width: 90%;
  line-height: 1.4;
}

.movies_poster:hover .movies_dialog {
  opacity: 1.0;
}

.movies_date {
  position: absolute;
  left: 5%;
  top: 5%;
  font-size: 2vh;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.8);
  background: rgba(0, 0, 0, 0.3);
  padding: 0.5vh 1vh;
  border-radius: 0.5vh;
}

/* 加载状态 */
.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #409EFF;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .movies_poster {
    width: 100%;
    height: 300px;
    margin-bottom: 20px;
  }

  .movies_dialog {
    font-size: 1.6vh;
    bottom: 15%;
  }

  .movies_date {
    font-size: 1.6vh;
  }
}

/* 禁用状态 */
.movies_poster[disabled] {
  opacity: 0.6;
  cursor: not-allowed;
}

.movies_poster[disabled]:hover {
  transform: none;
  box-shadow: inherit;
}

/* 字体支持 */
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

/* 深色主题支持 */
@media (prefers-color-scheme: dark) {
  .movies_dialog,
  .movies_date {
    color: #f0f0f0;
  }

  .loading-overlay {
    background: rgba(0, 0, 0, 0.8);
  }
}
</style>