<template>
  <div class="body">
    <el-container>
      <el-main class="main-content">
        <!-- Photo Gallery Section -->
        <el-container id="photo_player" class="photo_player">
          <transition-group name="photo-transition">
            <div
                v-for="(bgphoto, bgindex) in filteredPhotos"
                :key="bgindex"
                class="photo_holder"
                :style="{backgroundImage: bgphoto.backgroundImage}"
                @click="expandPhoto(bgindex)"
            >
              <div class="camera-badge">
                <el-image class="dressing_cam" :src="cameras[bgindex % cameras.length].camera" :fit="'contain'"/>
                <span class="year-tag">{{ getYearForPhoto(bgindex) }}</span>
              </div>

              <div class="photo_holder_main"
                   :style="{backgroundImage: `url(${filteredPhotos[bgindex].photo})`}">
                <div class="photo-info">
                  <h3>{{ filteredPhotos[bgindex].title }}</h3>
                  <p>{{ filteredPhotos[bgindex].description }}</p>
                </div>
              </div>

              <!-- 修改取色器部分，阻止事件冒泡 -->
              <el-color-picker
                  v-model="colorpickers[bgindex % colorpickers.length].color"
                  show-alpha
                  style="position: absolute; bottom: 0; left: 0; z-index: 5;"
                  @click.stop="handleColorPickerClick"
                  @change="updatePhotoFilter(bgindex)"
              />
            </div>
          </transition-group>
        </el-container>

        <!-- Empty State Message -->
        <div v-if="filteredPhotos.length === 0" class="empty-state">
          <i class="el-icon-picture"></i>
          <p>No photos found for this time period</p>
          <el-button type="primary" @click="resetFilters">Reset Filters</el-button>
        </div>

        <!-- Selected Photo Modal -->
        <el-dialog
            v-model="showExpandedPhoto"
            custom-class="photo-dialog"
            :show-close="true"
            :fullscreen="false"
        >
          <div v-if="selectedPhotoIndex !== null" class="expanded-photo">
            <el-image
                :src="filteredPhotos[selectedPhotoIndex].photo"
                fit="contain"
                class="expanded-image"
            />
            <div class="photo-details">
              <h2>{{ filteredPhotos[selectedPhotoIndex].title }}</h2>
              <p class="date">{{ getYearForPhoto(selectedPhotoIndex) }}</p>
              <p class="description">{{ filteredPhotos[selectedPhotoIndex].description }}</p>

              <!-- Photo Controls -->
              <div class="photo-controls">
                <el-button
                    v-if="selectedPhotoIndex > 0"
                    @click="selectedPhotoIndex--"
                    icon="el-icon-arrow-left"
                    circle
                />
                <el-button
                    type="primary"
                    @click="applyFilter"
                    icon="el-icon-magic-stick"
                >Apply Filter</el-button>
                <el-button
                    v-if="selectedPhotoIndex < filteredPhotos.length - 1"
                    @click="selectedPhotoIndex++"
                    icon="el-icon-arrow-right"
                    circle
                />
              </div>
            </div>
          </div>
        </el-dialog>
      </el-main>
    </el-container>

    <!-- Time Recorder Component -->
    <div class="time-recorder-wrapper">
      <TimeRecorder
          :duration="recordDuration"
          :initial-markers="timeMarkers"
          @time-update="handleTimeUpdate"
          @marker-added="handleMarkerAdded"
          @timeline-reset="handleTimelineReset"
      />

      <!-- Year Filter Buttons -->
      <div class="year-filters">
        <el-button
            v-for="year in availableYears"
            :key="year"
            :type="selectedYear === year ? 'primary' : 'default'"
            @click="filterByYear(year)"
            size="small"
            class="year-button"
        >
          {{ year }}
        </el-button>
        <el-button
            v-if="selectedYear !== null"
            @click="clearYearFilter"
            size="small"
            icon="el-icon-close"
        >
          Clear
        </el-button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.body {
  min-height: 100vh;
  width: 100%;
  cursor: url("@/assets/photograph/bg_rmv/resized/opacity/光圈_大.png"), auto;
  background-color: #121212;
  background-image: url("@/assets/photograph/forever_memory.png");
  background-size: cover;
  background-repeat: no-repeat;
  position: relative;
  overflow-x: hidden;
}

.body:active {
  cursor: url("@/assets/photograph/bg_rmv/resized/opacity/光圈_小.png"), auto;
}

.app-header {
  background-color: rgba(179, 192, 209, 0.8);
  color: #333;
  text-align: center;
  line-height: 60px;
  font-size: 24px;
  font-weight: 600;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  position: sticky;
  top: 0;
  z-index: 10;
}

.main-content {
  display: flex;
  flex-direction: column;
  background: none;
  color: #f5f5f5;
  text-align: center;
  align-items: center;
  justify-content: center;
  padding: 40px 20px;
  min-height: 60vh;
}

.photo_player {
  display: flex;
  flex-wrap: wrap;
  gap: 30px;
  justify-content: center;
  width: 100%;
  margin-bottom: 40px;
}

.photo-transition-enter-active,
.photo-transition-leave-active {
  transition: all 0.5s ease;
}

.photo-transition-enter-from {
  opacity: 0;
  transform: translateY(30px);
}

.photo-transition-leave-to {
  opacity: 0;
  transform: scale(0.8);
}

.photo_holder {
  position: relative;
  width: 250px;
  height: 350px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
  transition: all 0.3s ease;
  cursor: pointer;
}

.photo_holder:hover {
  transform: translateY(-10px);
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.4);
}

.camera-badge {
  position: absolute;
  top: 15px;
  left: 15px;
  z-index: 2;
  display: flex;
  align-items: center;
  gap: 10px;
}

.dressing_cam {
  width: 36px;
  height: 36px;
  filter: drop-shadow(2px 2px 3px rgba(0, 0, 0, 0.3));
  transition: transform 0.3s ease;
}

.photo_holder:hover .dressing_cam {
  transform: rotate(-10deg) scale(1.1);
}

.year-tag {
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: bold;
}

.photo_holder_main {
  position: relative;
  background-size: cover;
  background-position: center;
  margin: 10px;
  width: calc(100% - 20px);
  height: calc(100% - 40px);
  border-radius: 8px;
  box-shadow: inset 2px 2px 5px rgba(0, 0, 0, 0.2);
  transition: all 0.5s ease;
  overflow: hidden;
}

/* 修改photo-info样式以确保文本正确显示 */
.photo-info {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.8)); /* 更深的渐变背景确保文字可读 */
  padding: 15px;
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.3s ease;
  text-align: left;
  min-height: 80px; /* 确保有足够高度容纳文本 */
  max-height: 40%; /* 限制最大高度 */
  overflow: auto; /* 允许内容滚动 */
}

.photo-info h3 {
  margin: 0 0 5px;
  font-size: 16px;
  color: white;
  white-space: normal; /* 允许标题换行 */
  line-height: 1.2;
  max-height: 2.4em; /* 限制为2行 */
  overflow: hidden;
  text-overflow: ellipsis; /* 超出部分显示省略号 */
  display: -webkit-box;
  -webkit-line-clamp: 2; /* 限制最多显示2行 */
  -webkit-box-orient: vertical;
}

.photo-info p {
  margin: 0;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.9); /* 增加对比度 */
  line-height: 1.4;
  white-space: normal; /* 允许描述文本换行 */
  max-height: 4.2em; /* 限制为3行 */
  overflow: hidden;
  text-overflow: ellipsis; /* 超出部分显示省略号 */
  display: -webkit-box;
  -webkit-line-clamp: 3; /* 限制最多显示3行 */
  -webkit-box-orient: vertical;
}

.photo_holder:hover .photo-info {
  opacity: 1;
  transform: translateY(0);
  max-height: 50%; /* 悬停时增加最大高度，显示更多内容 */
}

.color-picker {
  position: absolute;
  bottom: 15px;
  right: 15px;
  z-index: 2;
  opacity: 0.6;
  transition: opacity 0.3s ease;
}

.color-picker:hover {
  opacity: 1;
}

.time-recorder-wrapper {
  position: relative;
  width: 100%;
  margin-top: 30px;
  padding: 20px;
  background-color: rgba(0, 0, 0, 0.3);
  border-radius: 12px;
  backdrop-filter: blur(8px);
}

.year-filters {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 20px;
}

.year-button {
  transition: all 0.3s ease;
}

.year-button:hover {
  transform: translateY(-3px);
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  margin: 30px 0;
}

.empty-state i {
  font-size: 48px;
  margin-bottom: 20px;
  opacity: 0.6;
}

.empty-state p {
  font-size: 18px;
  margin-bottom: 20px;
}

.photo-dialog {
  background-color: rgba(30, 30, 30, 0.9);
  border-radius: 16px;
  overflow: hidden;
  width: 80%;
  max-width: 1200px;
}

.expanded-photo {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

@media (min-width: 768px) {
  .expanded-photo {
    flex-direction: row;
  }
}

.expanded-image {
  width: 100%;
  max-height: 70vh;
  border-radius: 8px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.photo-details {
  padding: 0 20px;
  flex: 1;
  display: flex;
  flex-direction: column;
}

.photo-details h2 {
  margin-top: 0;
  color: white;
}

.photo-details .date {
  color: #aaa;
  font-size: 14px;
  margin-bottom: 20px;
}

.photo-details .description {
  flex: 1;
  line-height: 1.6;
}

.photo-controls {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
  gap: 10px;
}
</style>


<script setup>
import { ref, computed, onMounted } from "vue";
import TimeRecorder from './TimeRecorder.vue';

// Time recorder configuration
const recordDuration = ref(1460); // Total duration in seconds
const timeMarkers = ref([
  { position: 15, label: '2023', time: 90 },
  { position: 40, label: '2024', time: 240 },
  { position: 75, label: '2025', time: 450 }
]);

// Available years from the time markers
const availableYears = computed(() => {
  return timeMarkers.value.map(marker => marker.label);
});

// Current time position
const currentTime = ref(0);
const currentPercentage = ref(0);

// Selected year filter (null means show all)
const selectedYear = ref(null);

// Photo expansion
const showExpandedPhoto = ref(false);
const selectedPhotoIndex = ref(null);

// Define background gradients
const backgroundGradients = ref([
  { backgroundImage: "linear-gradient(to top, #30cfd0 0%, #330867 100%)" },
  { backgroundImage: "linear-gradient(to top, #a8edea 0%, #fed6e3 100%)" },
  { backgroundImage: "linear-gradient(to top, #fddb92 0%, #d1fdff 100%)" },
  { backgroundImage: "linear-gradient(-20deg, #e9defa 0%, #fbfcdb 100%)" },
  { backgroundImage: "linear-gradient(to right, #fa709a 0%, #fee140 100%)" },
  { backgroundImage: "linear-gradient(to top, #c471f5 0%, #fa71cd 100%)" },
  { backgroundImage: "linear-gradient(to top, #48c6ef 0%, #6f86d6 100%)" },
  { backgroundImage: "linear-gradient(to right, #434343 0%, black 100%)" }
]);

// Define camera icons
const cameras = ref([
  { camera: new URL("@/assets/photograph/vectorizer/红相机.svg", import.meta.url).href },
  { camera: new URL("@/assets/photograph/vectorizer/黄相机.svg", import.meta.url).href },
  { camera: new URL("@/assets/photograph/vectorizer/蓝相机.svg", import.meta.url).href },
  { camera: new URL("@/assets/photograph/vectorizer/绿相机.svg", import.meta.url).href },
]);

// Define color pickers with default values
const colorpickers = ref([
  {color: 'rgba(19, 206, 102, 0.8)'},
  {color: 'rgba(255, 102, 0, 0.7)'},
  {color: 'rgba(0, 102, 255, 0.7)'},
  {color: 'rgba(255, 0, 102, 0.7)'},
]);

// Enhanced photo data with more metadata
const allPhotos = ref([
  // 2023 photos
  {
    photo: new URL("@/assets/photograph/红相机.png", import.meta.url).href,
    backgroundImage: backgroundGradients.value[0].backgroundImage,
    title: "Summer Memories",
    description: "Captured during our summer trip to the beach",
    year: "2023",
    filter: null
  },
  {
    photo: new URL("@/assets/photograph/黄相机.png", import.meta.url).href,
    backgroundImage: backgroundGradients.value[1].backgroundImage,
    title: "Autumn Leaves",
    description: "The colors of fall in the park",
    year: "2023",
    filter: null
  },
  // 2024 photos
  {
    photo: new URL("@/assets/photograph/绿相机.png", import.meta.url).href,
    backgroundImage: backgroundGradients.value[2].backgroundImage,
    title: "Spring Garden",
    description: "First flowers of the season",
    year: "2024",
    filter: null
  },
  {
    photo: new URL("@/assets/photograph/蓝相机.png", import.meta.url).href,
    backgroundImage: backgroundGradients.value[3].backgroundImage,
    title: "Winter Wonderland",
    description: "Snow-covered mountains during our ski trip",
    year: "2024",
    filter: null
  },
  // 2025 photos
  {
    photo: new URL("@/assets/photograph/红相机.png", import.meta.url).href,
    backgroundImage: backgroundGradients.value[4].backgroundImage,
    title: "City Lights",
    description: "Downtown at night with beautiful illumination",
    year: "2025",
    filter: null
  },
  {
    photo: new URL("@/assets/photograph/黄相机.png", import.meta.url).href,
    backgroundImage: backgroundGradients.value[5].backgroundImage,
    title: "Sunset Boulevard",
    description: "Golden hour by the ocean",
    year: "2025",
    filter: null
  },
]);

const handleColorPickerClick = (event) => {
  // 阻止事件冒泡
  event.stopPropagation();
  // 这里不需要额外逻辑，el-color-picker会自己处理显示颜色面板
};

// Filtered photos based on current time and selected year
const filteredPhotos = computed(() => {
  let photos = [...allPhotos.value];

  // Filter by time position
  if (currentTime.value > 0) {
    const timePercentage = currentPercentage.value;

    // Use markers to determine which photos to show based on time
    const relevantMarkers = timeMarkers.value.filter(marker => marker.position <= timePercentage);

    if (relevantMarkers.length > 0) {
      const latestMarker = relevantMarkers[relevantMarkers.length - 1];
      photos = photos.filter(photo => photo.year <= latestMarker.label);
    }
  }

  // Apply year filter if selected
  if (selectedYear.value) {
    photos = photos.filter(photo => photo.year === selectedYear.value);
  }

  return photos;
});

// Time recorder handlers
const handleTimeUpdate = ({time, percentage}) => {
  currentTime.value = time;
  currentPercentage.value = percentage;
  console.log(`Current time: ${time}s, Progress: ${percentage}%`);
};

const handleMarkerAdded = (marker) => {
  console.log('Added new marker:', marker);
  timeMarkers.value.push(marker);
};

const handleTimelineReset = () => {
  console.log('Timeline reset');
  currentTime.value = 0;
  currentPercentage.value = 0;
};

// Get year for a specific photo index
const getYearForPhoto = (index) => {
  if (index >= 0 && index < filteredPhotos.value.length) {
    return filteredPhotos.value[index].year;
  }
  return '';
};

// Filter photos by year
const filterByYear = (year) => {
  selectedYear.value = year;
};

// Clear year filter
const clearYearFilter = () => {
  selectedYear.value = null;
};

// Reset all filters
const resetFilters = () => {
  selectedYear.value = null;
  currentTime.value = 0;
  currentPercentage.value = 0;
};

// Expand photo when clicked
const expandPhoto = (index) => {
  selectedPhotoIndex.value = index;
  showExpandedPhoto.value = true;
};

// Apply color filter to selected photo
const applyFilter = () => {
  if (selectedPhotoIndex.value !== null) {
    const index = selectedPhotoIndex.value;
    const pickerIndex = index % colorpickers.value.length;
    allPhotos.value[index].filter = colorpickers.value[pickerIndex].color;

    // Apply CSS filter effect based on color
    updatePhotoFilter(index);
  }
};

// Update photo filter based on color picker
const updatePhotoFilter = (index) => {
  // This would apply CSS filters based on the selected color
  // For example, adding sepia, brightness, or contrast effects
  console.log(`Updating filter for photo ${index}`);
};

// Component mounted
onMounted(() => {
  console.log("Photo page loaded");
});
</script>