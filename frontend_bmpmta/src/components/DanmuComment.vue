<!-- DanmuComment.vue -->
<template>
  <div class="danmu-container" v-show="enabled">
    <div
        v-for="(comment, index) in visibleComments"
        :key="index"
        class="danmu-comment"
        :style="calculatePosition(comment)"
        :class="{'user-danmu': comment.userId === currentUserId}"
    >
      <span class="emoji" v-if="comment.type === 'emoji'">{{ comment.content }}</span>
      <span
          class="text"
          v-else
          :style="{ color: comment.color || '#fff' }"
      >
        <span class="user-name" v-if="showUsernames">{{ comment.username || 'user' }}:</span>
        {{ comment.content }}
      </span>
    </div>
  </div>
</template>

<script>
import eventBus from '@/utils/eventBus.js';

export default {
  name: 'DanmuComment',
  props: {
    isPlaying: {
      type: Boolean,
      default: false
    },
    currentTime: {
      type: Number,
      default: 0
    },
    enabled: {
      type: Boolean,
      default: true
    },
    showUsernames: {
      type: Boolean,
      default: true
    },
    currentUserId: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      visibleComments: [],
      lanes: 10, // 弹幕轨道数量
      laneUsage: [], // 跟踪每个轨道的使用情况
      historicalComments: [] // 保存已经显示过的弹幕，用于断点续播时恢复
    };
  },
  watch: {
    // 当视频播放状态变化时，处理弹幕显示
    isPlaying(newVal) {
      if (newVal) {
        // 视频播放时，启动弹幕动画
        this.startDanmuAnimation();
      } else {
        // 视频暂停时，暂停弹幕动画
        this.pauseDanmuAnimation();
      }
    },
    // 当视频时间变化较大时（如用户拖动进度条），重新加载对应时间的弹幕
    currentTime(newVal, oldVal) {
      // 如果时间变化超过3秒，认为是用户跳转了视频
      if (Math.abs(newVal - oldVal) > 3) {
        this.handleVideoSeek(newVal);
      }
    },
    // 当弹幕开关状态变化时
    enabled(newVal) {
      if (!newVal) {
        // 如果关闭弹幕，清空当前显示的弹幕
        this.visibleComments = [];
      } else if (this.isPlaying) {
        // 如果开启弹幕且视频正在播放，重新加载弹幕
        this.loadDanmuAtCurrentTime();
      }
    }
  },
// Replace this.$root.$on with eventBus.on
  mounted() {
    // Initialize track usage status
    this.laneUsage = Array(this.lanes).fill(0);

    // Listen for new danmu events
    eventBus.on('new-danmu', this.addDanmu);

    // If danmu is enabled and video is playing, start danmu animation
    if (this.enabled && this.isPlaying) {
      this.startDanmuAnimation();
    }
  },

  beforeUnmount() {
    // Clear event listeners and timers
    eventBus.off('new-danmu', this.addDanmu);
    this.pauseDanmuAnimation();
  },

  methods: {
    // 添加新弹幕
    addDanmu(danmu) {
      // 如果弹幕未启用，不处理
      if (!this.enabled) return;

      // 如果是历史弹幕（有videoTime属性）且当前视频时间已经过了该弹幕时间，不显示
      if (danmu.videoTime && this.currentTime > danmu.videoTime + 0.5) {
        return;
      }

      // 添加时间戳和初始位置
      const newDanmu = {
        ...danmu,
        timestamp: Date.now(),
        lane: this.findAvailableLane(danmu),
        position: 100, // 从右侧开始 (100%)
        speed: this.calculateSpeed(danmu) // 根据弹幕类型和长度计算速度
      };

      // 添加到显示弹幕列表
      this.visibleComments.push(newDanmu);

      // 更新轨道使用状态
      this.laneUsage[newDanmu.lane] = Date.now();

      // 添加到历史弹幕列表
      this.historicalComments.push(newDanmu);

      // 限制历史弹幕数量
      if (this.historicalComments.length > 1000) {
        this.historicalComments = this.historicalComments.slice(-1000);
      }
    },

    // 找到一个可用的弹幕轨道
    findAvailableLane(danmu) {
      // 为不同类型的弹幕选择不同区域的轨道
      let startLane = 0;
      let endLane = this.lanes - 1;

      if (danmu.type === 'emoji') {
        // 表情弹幕放在上半部分
        endLane = Math.floor(this.lanes / 2) - 1;
      } else if (danmu.userId === this.currentUserId) {
        // 当前用户的弹幕放在中间部分
        startLane = Math.floor(this.lanes / 4);
        endLane = Math.floor(this.lanes * 3 / 4) - 1;
      } else {
        // 其他用户的文本弹幕放在下半部分
        startLane = Math.floor(this.lanes / 2);
      }

      // 找出指定范围内最久没使用的轨道
      let leastRecentLane = startLane;
      let leastRecentTime = Date.now();

      for (let i = startLane; i <= endLane; i++) {
        if (this.laneUsage[i] < leastRecentTime) {
          leastRecentLane = i;
          leastRecentTime = this.laneUsage[i];
        }
      }

      return leastRecentLane;
    },

    // 计算弹幕速度
    calculateSpeed(danmu) {
      // 基础速度
      let baseSpeed = 0.05;

      // 根据弹幕类型调整速度
      if (danmu.type === 'emoji') {
        // 表情弹幕慢一些
        baseSpeed *= 0.8;
      } else {
        // 文本弹幕根据长度调整速度，避免太长的弹幕速度过快
        const contentLength = danmu.content.length;
        if (contentLength > 10) {
          // 长弹幕速度稍快
          baseSpeed *= (1 + contentLength / 100);
        }
      }

      // 添加一点随机性
      baseSpeed += (Math.random() * 0.01 - 0.005);

      return baseSpeed;
    },

    // 计算弹幕的位置样式
    calculatePosition(danmu) {
      const topPercentage = (danmu.lane / this.lanes) * 80 + 10; // 让弹幕分布在10%~90%的高度范围内

      const styles = {
        top: `${topPercentage}%`,
        right: `${danmu.position}%`,
      };

      // 根据弹幕类型设置不同样式
      if (danmu.type === 'emoji') {
        styles.fontSize = '24px';
      } else if (danmu.fontSize) {
        styles.fontSize = `${danmu.fontSize}px`;
      }

      // 特殊类型弹幕的样式
      if (danmu.mode === 'top') {
        // 顶部固定弹幕
        styles.top = '10%';
        styles.right = '50%';
        styles.transform = 'translateX(50%)';
      } else if (danmu.mode === 'bottom') {
        // 底部固定弹幕
        styles.top = '85%';
        styles.right = '50%';
        styles.transform = 'translateX(50%)';
      }

      return styles;
    },

    // 启动弹幕动画
    startDanmuAnimation() {
      // 清除之前的定时器
      if (this.animationTimer) {
        clearInterval(this.animationTimer);
      }

      // 设置定时器更新弹幕位置
      this.animationTimer = setInterval(() => {
        this.updateDanmuPositions();
      }, 16); // 约60fps

      // 加载当前时间点的弹幕
      this.loadDanmuAtCurrentTime();
    },

    // 暂停弹幕动画
    pauseDanmuAnimation() {
      if (this.animationTimer) {
        clearInterval(this.animationTimer);
        this.animationTimer = null;
      }
    },

    // 更新弹幕位置
    updateDanmuPositions() {
      // 移动每条可见弹幕
      this.visibleComments.forEach(danmu => {
        // 不移动固定位置的弹幕
        if (danmu.mode === 'top' || danmu.mode === 'bottom') {
          return;
        }

        danmu.position -= danmu.speed;
      });

      // 移除已经移出屏幕的弹幕
      this.visibleComments = this.visibleComments.filter(danmu => {
        // 固定位置的弹幕保留时间长一些
        if (danmu.mode === 'top' || danmu.mode === 'bottom') {
          return (Date.now() - danmu.timestamp) < 5000; // 固定弹幕显示5秒
        }

        return danmu.position > -20; // 普通弹幕移出屏幕后移除
      });
    },

    // 处理视频跳转
    handleVideoSeek(newTime) {
      // 清空当前显示的弹幕
      this.visibleComments = [];

      // 加载新时间点的弹幕（使用传入的newTime）
      this.$emit('load-danmu',newTime);

      // 重置轨道使用状态
      this.laneUsage = Array(this.lanes).fill(0);
    },

    // 加载当前时间点的弹幕
    loadDanmuAtCurrentTime() {
      // 发出事件通知父组件需要加载当前时间点的弹幕
      this.$emit('load-danmu', this.currentTime);

      // 从历史弹幕中找出当前时间点附近的弹幕重新显示
      const nearbyDanmus = this.historicalComments.filter(danmu => {
        // 只显示当前时间前后2秒内的弹幕
        return danmu.videoTime &&
            Math.abs(danmu.videoTime - this.currentTime) < 2 &&
            !this.visibleComments.some(d => d.id === danmu.id);
      });

      // 将这些弹幕重新添加到显示列表
      nearbyDanmus.forEach(danmu => {
        this.addDanmu(danmu);
      });
    }
  }
};
</script>

<style scoped>
.danmu-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none; /* 让弹幕不影响视频的点击事件 */
  overflow: hidden;
  z-index: 5; /* 确保在视频上方，但在控制器下方 */
}

.danmu-comment {
  position: absolute;
  white-space: nowrap;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.8);
  font-weight: bold;
  transition: right 0.016s linear; /* 平滑移动 */
}

.emoji {
  font-size: 24px;
  filter: drop-shadow(0 0 2px rgba(0, 0, 0, 0.7));
}

.text {
  padding: 2px 6px;
  background-color: rgba(0, 0, 0, 0.3);
  border-radius: 4px;
}

.user-name {
  color: #FFD700; /* 金色用户名 */
  margin-right: 3px;
  font-size: 0.9em;
}

.user-danmu {
  z-index: 6; /* 当前用户的弹幕层级稍高 */
}

.user-danmu .text {
  background-color: rgba(56, 103, 214, 0.6); /* 当前用户的弹幕背景色 */
  border: 1px solid rgba(255, 255, 255, 0.3);
}
</style>