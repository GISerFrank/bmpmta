<template>
  <div class="time-recorder">
    <div class="timeline-container">
      <div class="timeline" ref="timelineRef">
        <div class="timeline-progress" :style="{ width: `${timelineProgress}%` }"></div>
        <div class="timeline-handle"
             :style="{ left: `${timelineProgress}%` }"
             @mousedown="startDrag"
             ref="timelineHandle">
          <div class="time-display">{{ formatTime(currentTime) }}</div>
        </div>
        <div class="timeline-markers">
          <div class="timeline-marker" v-for="(marker, index) in timelineMarkers"
               :key="index"
               :style="{ left: `${marker.position}%` }"
               @click="jumpToMarker(marker)">
            <div class="marker-label">{{ marker.label }}</div>
          </div>
        </div>
      </div>
      <div class="timeline-controls">
        <el-button type="primary" size="small" @click="playPause">
          <i :class="isPlaying ? 'el-icon-video-pause' : 'el-icon-video-play'"></i>
          {{ isPlaying ? '暂停' : '播放' }}
        </el-button>
        <el-button type="primary" size="small" @click="addMarker">
          添加标记
        </el-button>
        <el-button type="default" size="small" @click="reset">
          重置
        </el-button>
        <div class="timeline-speed-control">
          <span>速度:</span>
          <el-slider
              v-model="playbackSpeed"
              :min="0.5"
              :max="3"
              :step="0.5"
              :format-tooltip="formatSpeed"
              style="width: 100px; margin-left: 10px;"
          ></el-slider>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, defineComponent, watch } from 'vue';

export default defineComponent({
  name: 'TimeRecorder',

  props: {
    // 时间轴跨度
    duration: {
      type: Number,
      default: 300 // 默认总时长为300秒（5分钟）
    },
    // 初始标记列表
    initialMarkers: {
      type: Array,
      default: () => []
    }
  },

  emits: ['time-update', 'marker-added', 'timeline-reset'],

  setup(props, { emit }) {
    // 时间轴状态
    const timelineProgress = ref(0); // 时间线位置
    const currentTime = ref(0); // 当前时间
    const isPlaying = ref(false); // 播放状态
    const timelineHandle = ref(null); //
    const timelineRef = ref(null);
    const isDragging = ref(false); // 拖动状态
    const playbackSpeed = ref(1);

    // 初始化标记或使用默认标记
    const timelineMarkers = ref(
        props.initialMarkers.length > 0
            ? [...props.initialMarkers]
            : [
              { position: 10, label: '开始', time: props.duration * 0.1 },
              { position: 50, label: '中间', time: props.duration * 0.5 },
              { position: 90, label: '结束', time: props.duration * 0.9 }
            ]
    );

    let playInterval = null;

    // 将秒格式化为 mm:ss 格式
    const formatTime = (seconds) => {
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    };

    // 将播放速度格式化为文本
    const formatSpeed = (val) => {
      return `${val}x`;
    };

    // 处理时间轴点击
    const handleTimelineClick = (event) => {
      if (isDragging.value) return;

      const timeline = event.currentTarget;
      const rect = timeline.getBoundingClientRect();
      const clickX = event.clientX - rect.left;
      const percentage = (clickX / rect.width) * 100;

      updateProgress(percentage);
    };

    // 更新进度和时间
    const updateProgress = (percentage) => {
      // 将百分比限制在0到100之间
      const clampedPercentage = Math.max(0, Math.min(100, percentage));
      timelineProgress.value = clampedPercentage;
      currentTime.value = (clampedPercentage / 100) * props.duration;

      // 触发时间更新事件
      emit('time-update', {
        time: currentTime.value,
        percentage: clampedPercentage
      });
    };

    // 开始拖动时间轴手柄
    const startDrag = (event) => {
      event.preventDefault();
      isDragging.value = true;
      document.addEventListener('mousemove', drag);
      document.addEventListener('mouseup', stopDrag);
    };

    // 拖动处理器
    const drag = (event) => {
      if (!isDragging.value) return;

      const timeline = timelineRef.value;
      const rect = timeline.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const percentage = (x / rect.width) * 100;

      updateProgress(percentage);
    };

    // 停止拖动
    const stopDrag = () => {
      isDragging.value = false;
      document.removeEventListener('mousemove', drag);
      document.removeEventListener('mouseup', stopDrag);
    };

    // 开始播放的辅助函数
    const startPlayback = (speed) => {
      // 开始自动进度
      playInterval = setInterval(() => {
        if (currentTime.value >= props.duration) {
          isPlaying.value = false;
          clearInterval(playInterval);
          return;
        }

        // 根据播放速度调整增量
        const increment = 0.1 * speed;
        currentTime.value += increment;
        timelineProgress.value = (currentTime.value / props.duration) * 100;

        emit('time-update', {
          time: currentTime.value,
          percentage: timelineProgress.value
        });
      }, 100); // 每100毫秒更新一次
    };

    // 播放/暂停时间轴进度
    const playPause = () => {
      isPlaying.value = !isPlaying.value;

      if (isPlaying.value) {
        startPlayback(playbackSpeed.value);
      } else {
        // 停止自动进度
        clearInterval(playInterval);
      }
    };

    // 跳转到标记位置
    const jumpToMarker = (marker) => {
      updateProgress(marker.position);
    };

    // 添加新标记在当前位置
    const addMarker = () => {
      const newMarker = {
        position: timelineProgress.value,
        label: `标记 ${timelineMarkers.value.length + 1}`,
        time: currentTime.value
      };

      timelineMarkers.value.push(newMarker);
      emit('marker-added', newMarker);
    };

    // 重置时间轴
    const reset = () => {
      // 停止播放
      if (isPlaying.value) {
        clearInterval(playInterval);
        isPlaying.value = false;
      }

      // 重置位置和时间
      currentTime.value = 0;
      timelineProgress.value = 0;

      emit('timeline-reset');
    };

    // 监听播放速度变化
    watch(playbackSpeed, (newSpeed) => {
      console.log(`播放速度已更改为 ${newSpeed}x`);

      if (isPlaying.value) {
        // 如果正在播放，重新启动计时器以应用新速度
        clearInterval(playInterval);

        // 直接重启播放，保持当前播放状态
        startPlayback(newSpeed);
      }
    });

    // 添加时间轴点击事件监听器
    onMounted(() => {
      if (timelineRef.value) {
        timelineRef.value.addEventListener('click', handleTimelineClick);
      }
    });

    // 清除事件监听器
    onBeforeUnmount(() => {
      clearInterval(playInterval);

      if (timelineRef.value) {
        timelineRef.value.removeEventListener('click', handleTimelineClick);
      }

      document.removeEventListener('mousemove', drag);
      document.removeEventListener('mouseup', stopDrag);
    });

    return {
      timelineProgress,
      currentTime,
      isPlaying,
      timelineHandle,
      timelineRef,
      timelineMarkers,
      playbackSpeed,
      formatTime,
      formatSpeed,
      handleTimelineClick,
      updateProgress,
      startDrag,
      drag,
      stopDrag,
      playPause,
      startPlayback,
      jumpToMarker,
      addMarker,
      reset
    };
  }
});
</script>

<style scoped>
/* ====== 增强版时间轴样式 - 适用于照片页面 ====== */

.time-recorder {
  width: 100%;
}

.timeline-container {
  position: relative;
  width: 100%;
  margin: 0 auto;
  padding: 30px 20px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 16px;
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

/* 主时间轴轨道 */
.timeline {
  position: relative;
  height: 12px;
  background: linear-gradient(145deg, rgba(255, 255, 255, 0.1), rgba(255, 255, 255, 0.05));
  border-radius: 20px;
  cursor: pointer;
  box-shadow:
      inset 0 2px 6px rgba(0, 0, 0, 0.3),
      0 2px 8px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.08);
  overflow: hidden;
  transition: all 0.3s ease;
}

.timeline:hover {
  height: 14px;
  box-shadow:
      inset 0 2px 8px rgba(0, 0, 0, 0.35),
      0 4px 16px rgba(255, 255, 255, 0.1);
}

/* 时间轴进度填充 */
.timeline-progress {
  position: absolute;
  height: 100%;
  background: linear-gradient(45deg,
  #667eea 0%,
  #764ba2 25%,
  #f093fb 50%,
  #f5576c 75%,
  #4facfe 100%
  );
  background-size: 400% 100%;
  border-radius: 20px;
  transition: width 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  animation: gradientFlow 6s ease-in-out infinite;
  box-shadow:
      0 0 20px rgba(102, 126, 234, 0.4),
      inset 0 1px 0 rgba(255, 255, 255, 0.3);
  position: relative;
  overflow: hidden;
}

/* 进度条上的光泽效果 */
.timeline-progress::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 50%;
  background: linear-gradient(to bottom,
  rgba(255, 255, 255, 0.4),
  rgba(255, 255, 255, 0.1)
  );
  border-radius: 20px 20px 0 0;
}

/* 进度条流光效果 */
.timeline-progress::after {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg,
  transparent,
  rgba(255, 255, 255, 0.6),
  transparent
  );
  animation: shine 3s ease-in-out infinite;
}

/* 渐变流动动画 */
@keyframes gradientFlow {
  0%, 100% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
}

/* 流光动画 */
@keyframes shine {
  0% { left: -100%; }
  50% { left: 100%; }
  100% { left: 100%; }
}

/* 时间轴手柄 */
.timeline-handle {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 24px;
  height: 24px;
  background: linear-gradient(135deg,
  rgba(255, 255, 255, 0.9) 0%,
  rgba(255, 255, 255, 0.7) 100%
  );
  border: 3px solid rgba(102, 126, 234, 0.8);
  border-radius: 50%;
  cursor: grab;
  z-index: 10;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow:
      0 4px 12px rgba(0, 0, 0, 0.3),
      0 2px 6px rgba(102, 126, 234, 0.4),
      inset 0 1px 0 rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
}

.timeline-handle:hover {
  transform: translate(-50%, -50%) scale(1.2);
  box-shadow:
      0 6px 20px rgba(0, 0, 0, 0.4),
      0 4px 12px rgba(102, 126, 234, 0.6),
      inset 0 1px 0 rgba(255, 255, 255, 0.9);
  border-color: rgba(102, 126, 234, 1);
}

.timeline-handle:active {
  cursor: grabbing;
  transform: translate(-50%, -50%) scale(1.1);
  box-shadow:
      0 2px 8px rgba(0, 0, 0, 0.4),
      inset 0 1px 2px rgba(0, 0, 0, 0.2);
}

/* 时间显示气泡 */
.time-display {
  position: absolute;
  top: -40px;
  left: 50%;
  transform: translateX(-50%);
  background: linear-gradient(135deg,
  rgba(0, 0, 0, 0.8) 0%,
  rgba(0, 0, 0, 0.9) 100%
  );
  color: white;
  padding: 6px 12px;
  border-radius: 12px;
  font-size: 13px;
  font-weight: 600;
  white-space: nowrap;
  box-shadow:
      0 4px 12px rgba(0, 0, 0, 0.3),
      inset 0 1px 0 rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  opacity: 0;
  transition: all 0.3s ease;
  pointer-events: none;
}

/* 气泡小箭头 */
.time-display::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
  border-top: 6px solid rgba(0, 0, 0, 0.8);
}

.timeline-handle:hover .time-display {
  opacity: 1;
  transform: translateX(-50%) translateY(-5px);
}

/* 标记点容器 */
.timeline-markers {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

/* 时间轴标记点 */
.timeline-marker {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 16px;
  height: 16px;
  background: linear-gradient(135deg,
  rgba(245, 108, 108, 0.9) 0%,
  rgba(255, 99, 132, 0.9) 100%
  );
  border: 2px solid rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  cursor: pointer;
  z-index: 8;
  transition: all 0.3s ease;
  box-shadow:
      0 3px 8px rgba(245, 108, 108, 0.4),
      inset 0 1px 0 rgba(255, 255, 255, 0.5);
  pointer-events: auto;
  animation: markerPulse 3s ease-in-out infinite;
}

.timeline-marker:hover {
  transform: translate(-50%, -50%) scale(1.3);
  box-shadow:
      0 6px 16px rgba(245, 108, 108, 0.6),
      inset 0 1px 0 rgba(255, 255, 255, 0.7);
  border-color: rgba(255, 255, 255, 1);
}

/* 标记点脉冲动画 */
@keyframes markerPulse {
  0%, 100% {
    box-shadow:
        0 3px 8px rgba(245, 108, 108, 0.4),
        inset 0 1px 0 rgba(255, 255, 255, 0.5);
  }
  50% {
    box-shadow:
        0 5px 15px rgba(245, 108, 108, 0.6),
        0 0 20px rgba(245, 108, 108, 0.3),
        inset 0 1px 0 rgba(255, 255, 255, 0.6);
  }
}

/* 标记点标签 */
.marker-label {
  position: absolute;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  background: linear-gradient(135deg,
  rgba(245, 108, 108, 0.9) 0%,
  rgba(255, 99, 132, 0.9) 100%
  );
  color: white;
  padding: 4px 10px;
  border-radius: 8px;
  font-size: 12px;
  font-weight: 600;
  white-space: nowrap;
  box-shadow:
      0 3px 10px rgba(0, 0, 0, 0.3),
      inset 0 1px 0 rgba(255, 255, 255, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(5px);
  opacity: 0;
  transition: all 0.3s ease;
}

/* 标签小箭头 */
.marker-label::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-top: 5px solid rgba(245, 108, 108, 0.9);
}

.timeline-marker:hover .marker-label {
  opacity: 1;
  transform: translateX(-50%) translateY(-3px);
}

/* 控制按钮区域 */
.timeline-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 25px;
  gap: 15px;
  flex-wrap: wrap;
}

/* 增强控制按钮样式 */
.timeline-controls .el-button {
  background: rgba(255, 255, 255, 0.1) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  color: white !important;
  backdrop-filter: blur(10px);
  transition: all 0.3s ease !important;
  font-weight: 500;
}

.timeline-controls .el-button:hover {
  background: rgba(255, 255, 255, 0.2) !important;
  border-color: rgba(255, 255, 255, 0.4) !important;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.timeline-controls .el-button--primary {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.8), rgba(118, 75, 162, 0.8)) !important;
  border-color: rgba(102, 126, 234, 0.6) !important;
}

.timeline-controls .el-button--primary:hover {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.9), rgba(118, 75, 162, 0.9)) !important;
  border-color: rgba(102, 126, 234, 0.8) !important;
}

/* 速度控制区域 */
.timeline-speed-control {
  display: flex;
  align-items: center;
  margin-left: 20px;
  color: white;
  font-weight: 500;
}

.timeline-speed-control span {
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.5);
}

/* 滑块样式增强 */
.timeline-speed-control .el-slider {
  margin-left: 12px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .timeline-container {
    padding: 20px 15px;
  }

  .timeline {
    height: 10px;
  }

  .timeline:hover {
    height: 12px;
  }

  .timeline-handle {
    width: 20px;
    height: 20px;
  }

  .timeline-marker {
    width: 14px;
    height: 14px;
  }

  .timeline-controls {
    gap: 10px;
    margin-top: 20px;
  }

  .timeline-speed-control {
    margin-left: 0;
    margin-top: 10px;
    flex-basis: 100%;
    justify-content: center;
  }

  .time-display {
    font-size: 11px;
    padding: 4px 8px;
    top: -35px;
  }

  .marker-label {
    font-size: 11px;
    padding: 3px 8px;
  }
}

/* 无障碍支持 */
.timeline:focus {
  outline: 2px solid rgba(102, 126, 234, 0.8);
  outline-offset: 2px;
}

.timeline-handle:focus {
  outline: 2px solid rgba(255, 255, 255, 0.8);
  outline-offset: 2px;
}

.timeline-marker:focus {
  outline: 2px solid rgba(245, 108, 108, 0.8);
  outline-offset: 2px;
}
</style>