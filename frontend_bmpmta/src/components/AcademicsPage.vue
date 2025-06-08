<template>
  <div class="chat-app-container">
    <!-- Sidebar with chat list -->
    <div class="sidebar">
      <div class="sidebar-header">
        <h3>My Chats</h3>
        <el-button type="primary" size="small" @click="createNewChat">
          <i class="el-icon-plus"></i> New Chat
        </el-button>
      </div>

      <!-- Search bar -->
      <div class="search-container">
        <el-input
            v-model="searchQuery"
            placeholder="Search chats..."
            prefix-icon="el-icon-search"
            clearable
        />
      </div>

      <!-- Chat list with infinite scroll -->
      <div class="chat-list-container">
        <ul
            v-infinite-scroll="loadInfinite"
            class="infinite-list"
            :infinite-scroll-disabled="infiniteDisabled"
        >
          <li
              v-for="chat in filteredChats"
              :id="chat.id"
              :key="chat.id"
              class="chat-list-item"
              :class="{ active: activeChat === chat.index }"
              draggable="true"
              @click="openChat(chat.index, $event)"
              @dragstart="onDragStart($event, chat)"
              @dragover.prevent
              @drop="onDrop($event, chat)"
          >
            <div class="chat-item-content">
              <div class="chat-preview">{{ chat.content }}</div>
              <div class="chat-meta">
                <span class="chat-time">{{ formatTime(chat.timestamp) }}</span>
              </div>
            </div>
            <el-dropdown trigger="click" @command="handleCommand($event, chat)">
              <span class="el-dropdown-link">
                <i class="el-icon-more"></i>
              </span>
              <template #dropdown>
                <el-dropdown-menu>
                  <el-dropdown-item command="rename">Rename</el-dropdown-item>
                  <el-dropdown-item command="delete" divided>Delete</el-dropdown-item>
                </el-dropdown-menu>
              </template>
            </el-dropdown>
          </li>
        </ul>
        <div v-if="loading" class="loading-more">
          <el-spinner size="small" />
          Loading more...
        </div>
        <div v-if="filteredChats.length === 0" class="no-chats">
          <i class="el-icon-chat-dot-square"></i>
          <p>{{ searchQuery ? 'No chats match your search' : 'No chats yet' }}</p>
        </div>
      </div>
    </div>

    <!-- Main content area -->
    <div class="main-content">
      <keep-alive>
        <component
            :is="currentComponent"
            :currentChatId="currentChatId"
            @change-component="handleComponentChange"
        />
      </keep-alive>

      <!-- Floating action buttons -->
      <div class="floating-button-group" :class="{ 'menu-open': buttonMenuOpen }">
        <div class="connecting-lines">
          <div class="line line-to-category"></div>
          <div class="line line-to-statistics"></div>
          <div class="line line-to-thinking"></div>
        </div>

        <!-- 主按钮 -->
        <el-button
            type="primary"
            :style="{ backgroundImage: `url(${mindful_chat})` }"
            class="floating-button main-button"
            :class="{ 'active': buttonMenuOpen }"
            @click="toggleButtonMenu"
            title="Chat Menu"
        />

        <!-- 子按钮 -->
        <el-button
            type="primary"
            :style="{ backgroundImage: `url(${chat_category})` }"
            class="floating-button category-button"
            @click="switchToChatCategory"
            title="Categories"
        />
        <el-button
            type="primary"
            :style="{ backgroundImage: `url(${chat_statistics})` }"
            class="floating-button statistics-button"
            @click="switchToStatistics"
            title="Statistics"
        />
        <el-button
            type="primary"
            class="floating-button thinking-button"
            @click="switchToThinking"
            title="Thinking Mode"
        >
          <i class="el-icon-light-rain"></i>
        </el-button>
      </div>

      <!-- 遮罩层，点击关闭菜单 -->
      <div
          v-if="buttonMenuOpen"
          class="menu-overlay"
          @click="closeButtonMenu"
      ></div>
    </div>

    <!-- Rename dialog -->
    <el-dialog v-model="renameDialogVisible" title="Rename Chat" width="30%">
      <el-input v-model="newChatName" placeholder="Enter new name"></el-input>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="renameDialogVisible = false">Cancel</el-button>
          <el-button type="primary" @click="confirmRename">Confirm</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- Delete confirmation dialog -->
    <el-dialog v-model="deleteDialogVisible" title="Delete Chat" width="30%">
      <p>Are you sure you want to delete this chat?</p>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="deleteDialogVisible = false">Cancel</el-button>
          <el-button type="danger" @click="confirmDelete">Delete</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from "vue";
import ChatPage from "@/components/ChatPage.vue";
import ChatStatistics from "@/components/ChatStatistics.vue";
import ThinkingPage from "@/components/Thinking.vue";
import ChatCategory from "@/components/CategoryChat.vue";
import chat_category from "@/assets/chat/chat_category.png";
import chat_statistics from "@/assets/chat/chat_statistics.png";
import mindful_chat from "@/assets/chat/mindful_chat.png";

// Component state
const currentComponent = ref(ChatStatistics);
const activeChat = ref(null);
const currentChatId = ref(null);
const loading = ref(false);
const buttonMenuOpen = ref(false);
const searchQuery = ref('');
const infiniteDisabled = ref(false);

// Dialog states
const renameDialogVisible = ref(false);
const deleteDialogVisible = ref(false);
const newChatName = ref('');
const chatToModify = ref(null);

// Sample chat data with more realistic content and timestamps
const chats = ref([
  {index: 1, id: 'chat_001', content: 'Project brainstorming session', timestamp: Date.now() - 3600000 * 24 * 3},
  {index: 2, id: 'chat_002', content: 'Marketing strategy discussion', timestamp: Date.now() - 3600000 * 24 * 2},
  {index: 3, id: 'chat_003', content: 'Weekly team meeting notes', timestamp: Date.now() - 3600000 * 24},
  {index: 4, id: 'chat_004', content: 'Product feature ideas', timestamp: Date.now() - 3600000 * 12},
  {index: 5, id: 'chat_005', content: 'Customer feedback analysis', timestamp: Date.now() - 3600000 * 6},
  {index: 6, id: 'chat_006', content: 'Quarterly goals review', timestamp: Date.now() - 3600000 * 3},
  {index: 7, id: 'chat_007', content: 'Design system updates', timestamp: Date.now() - 3600000 * 2},
  {index: 8, id: 'chat_008', content: 'Bug tracking discussion', timestamp: Date.now() - 3600000},
  {index: 9, id: 'chat_009', content: 'Project timeline planning', timestamp: Date.now() - 1800000},
  {index: 10, id: 'chat_010', content: 'Social media campaign ideas', timestamp: Date.now() - 900000},
  {index: 11, id: 'chat_011', content: 'Content calendar for Q2', timestamp: Date.now() - 600000},
  {index: 12, id: 'chat_012', content: 'Technical interview questions', timestamp: Date.now() - 300000},
  {index: 13, id: 'chat_013', content: 'Website redesign feedback', timestamp: Date.now() - 180000},
  {index: 14, id: 'chat_014', content: 'User testing results', timestamp: Date.now() - 60000},
  {index: 15, id: 'chat_015', content: 'Latest AI research discussion', timestamp: Date.now()},
]);

// Visible chats count for infinite scrolling
const visibleChatsCount = ref(5);

// Filtered chats based on search query
const filteredChats = computed(() => {
  if (!searchQuery.value) {
    return chats.value.slice(0, visibleChatsCount.value).sort((a, b) => b.timestamp - a.timestamp);
  }

  return chats.value
      .filter(chat =>
          chat.content.toLowerCase().includes(searchQuery.value.toLowerCase())
      )
      .sort((a, b) => b.timestamp - a.timestamp);
});

// Handle opening a chat
const openChat = (chatIndex) => {
  activeChat.value = chatIndex;
  const clickedChat = chats.value.find(chat => chat.index === chatIndex);
  currentChatId.value = clickedChat?.id;

  console.log("Opening chat:", chatIndex, "with ID:", currentChatId.value);

  // Switch to chat page component
  currentComponent.value = ChatPage;

  // Update timestamp when chat is opened
  if (clickedChat) {
    clickedChat.timestamp = Date.now();
  }
};

// Load more chats for infinite scrolling
const loadInfinite = async () => {
  if (visibleChatsCount.value >= chats.value.length) {
    infiniteDisabled.value = true;
    return;
  }

  loading.value = true;

  // Simulate network delay
  setTimeout(() => {
    const increment = Math.min(3, chats.value.length - visibleChatsCount.value);
    visibleChatsCount.value += increment;
    loading.value = false;

    // Disable infinite scroll when all chats are loaded
    if (visibleChatsCount.value >= chats.value.length) {
      infiniteDisabled.value = true;
    }
  }, 500);
};

// Format timestamp to relative time
const formatTime = (timestamp) => {
  const diff = Date.now() - timestamp;

  if (diff < 60000) return 'Just now';
  if (diff < 3600000) return `${Math.floor(diff / 60000)}m ago`;
  if (diff < 86400000) return `${Math.floor(diff / 3600000)}h ago`;

  return new Date(timestamp).toLocaleDateString();
};

// Component switching handlers
const handleComponentChange = (componentName) => {
  const componentMap = {
    'ChatStatistics': ChatStatistics,
    'ThinkingPage': ThinkingPage,
    'ChatCategory': ChatCategory,
    'ChatPage': ChatPage
  };

  if (componentMap[componentName]) {
    currentComponent.value = componentMap[componentName];
  }
};

// Button menu toggle - 修复后的菜单控制
const toggleButtonMenu = () => {
  buttonMenuOpen.value = !buttonMenuOpen.value;
};

const closeButtonMenu = () => {
  buttonMenuOpen.value = false;
};

// Component switching methods - 点击子按钮时关闭菜单
const switchToStatistics = () => {
  currentComponent.value = ChatStatistics;
  buttonMenuOpen.value = false;
};

const switchToThinking = () => {
  currentComponent.value = ThinkingPage;
  buttonMenuOpen.value = false;
};

const switchToChatCategory = () => {
  currentComponent.value = ChatCategory;
  buttonMenuOpen.value = false;
};

// Create new chat
const createNewChat = () => {
  const newIndex = chats.value.length + 1;
  const newId = `chat_${String(newIndex).padStart(3, '0')}`;

  chats.value.push({
    index: newIndex,
    id: newId,
    content: 'New Chat',
    timestamp: Date.now()
  });

  // Open the new chat immediately
  openChat(newIndex, {currentTarget: {id: newId}});
};

// Drag and drop functionality
const onDragStart = (event, chat) => {
  event.dataTransfer.setData('text/plain', chat.index.toString());
};

const onDrop = (event, targetChat) => {
  const draggedIndex = parseInt(event.dataTransfer.getData('text/plain'));
  const targetIndex = targetChat.index;

  if (draggedIndex === targetIndex) return;

  // Find the chats by index
  const draggedChat = chats.value.find(chat => chat.index === draggedIndex);

  if (!draggedChat) return;

  // Reorder the chats array
  const newChats = chats.value.filter(chat => chat.index !== draggedIndex);
  const targetPosition = newChats.findIndex(chat => chat.index === targetIndex);

  newChats.splice(targetPosition, 0, draggedChat);

  // Update indices
  newChats.forEach((chat, idx) => {
    chat.index = idx + 1;
  });

  chats.value = newChats;
};

// Dialog command handlers
const handleCommand = (command, chat) => {
  chatToModify.value = chat;

  if (command === 'rename') {
    newChatName.value = chat.content;
    renameDialogVisible.value = true;
  } else if (command === 'delete') {
    deleteDialogVisible.value = true;
  }
};

const confirmRename = () => {
  if (chatToModify.value && newChatName.value.trim()) {
    const chat = chats.value.find(c => c.id === chatToModify.value.id);
    if (chat) {
      chat.content = newChatName.value.trim();
      chat.timestamp = Date.now(); // Update timestamp
    }
  }
  renameDialogVisible.value = false;
};

const confirmDelete = () => {
  if (chatToModify.value) {
    chats.value = chats.value.filter(chat => chat.id !== chatToModify.value.id);

    // Reset current chat if the deleted one was active
    if (activeChat.value === chatToModify.value.index) {
      activeChat.value = null;
      currentChatId.value = null;
      currentComponent.value = ChatStatistics;
    }

    // Update indices
    chats.value.forEach((chat, idx) => {
      chat.index = idx + 1;
    });
  }
  deleteDialogVisible.value = false;
};

// Initialize the app
const load = () => {
  // Sort chats by timestamp on initial load
  chats.value.sort((a, b) => b.timestamp - a.timestamp);

  // Set initial visible chats
  visibleChatsCount.value = Math.min(5, chats.value.length);
};

// Reset infinite scroll when search query changes
watch(searchQuery, () => {
  if (searchQuery.value) {
    infiniteDisabled.value = true; // Disable infinite scroll during search
  } else {
    visibleChatsCount.value = Math.min(5, chats.value.length);
    infiniteDisabled.value = visibleChatsCount.value >= chats.value.length;
  }
});

// Run on component mount
onMounted(() => {
  load();

  // Add keyboard shortcuts
  document.addEventListener('keydown', (e) => {
    // Ctrl/Cmd + N to create new chat
    if ((e.ctrlKey || e.metaKey) && e.key === 'n') {
      e.preventDefault();
      createNewChat();
    }

    // Escape to close the floating menu
    if (e.key === 'Escape' && buttonMenuOpen.value) {
      buttonMenuOpen.value = false;
    }
  });
});
</script>

<style scoped>
.chat-app-container {
  display: grid;
  grid-template-columns: 300px 1fr;
  height: 100vh;
  background-color: #f8f2e8;
  overflow: hidden;
}

/* Sidebar styling */
.sidebar {
  background-color: white;
  border-right: 1px solid #e8e0d8;
  display: flex;
  flex-direction: column;
  height: 100%;
  box-shadow: 2px 0 8px rgba(0, 0, 0, 0.05);
}

.sidebar-header {
  padding: 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #e8e0d8;
}

.sidebar-header h3 {
  margin: 0;
  font-weight: 600;
  color: #333;
}

.search-container {
  padding: 12px 16px;
  border-bottom: 1px solid #e8e0d8;
}

.chat-list-container {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.infinite-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
  margin: 0;
  list-style: none;
}

.chat-list-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  margin-bottom: 8px;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: all 0.2s ease;
  cursor: pointer;
}

.chat-list-item:hover {
  background-color: #f8f8f8;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.chat-list-item.active {
  background-color: #e6f7ff;
  border-left: 3px solid #1890ff;
  box-shadow: 0 2px 8px rgba(24, 144, 255, 0.2);
}

.chat-item-content {
  flex: 1;
  overflow: hidden;
}

.chat-preview {
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-bottom: 4px;
  color: #333;
}

.chat-meta {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #999;
}

.el-dropdown-link {
  cursor: pointer;
  color: #666;
  padding: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: background-color 0.2s;
}

.el-dropdown-link:hover {
  background-color: #f0f0f0;
}

.loading-more {
  padding: 12px;
  text-align: center;
  color: #999;
  font-size: 14px;
}

.no-chats {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #999;
  font-size: 16px;
  padding: 32px;
  text-align: center;
}

.no-chats i {
  font-size: 48px;
  margin-bottom: 16px;
  opacity: 0.5;
}

/* Main content area */
.main-content {
  position: relative;
  padding: 24px;
  overflow: auto;
}

/* 遮罩层 */
.menu-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 99;
  opacity: 0;
  animation: fadeIn 0.3s ease forwards;
}

@keyframes fadeIn {
  to {
    opacity: 1;
  }
}

/* Floating action buttons - 修复后的样式 */
.floating-button-group {
  position: fixed;
  bottom: 100px;
  right: 100px;
  z-index: 100;
}

.connecting-lines {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  z-index: 0;
  transform: translate(-30px, -30px);
  pointer-events: none;
}

.line {
  position: absolute;
  top: 30px;
  left: 30px;
  height: 2px;
  background-color: rgba(24, 144, 255, 0.6);
  transform-origin: left center;
  width: 0;
  transition: all 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  opacity: 0;
  border-radius: 2px;
  z-index: 0;
}

.line-to-category {
  transform: rotate(210deg);
}

.line-to-statistics {
  transform: rotate(240deg);
}

.line-to-thinking {
  transform: rotate(270deg);
}

.floating-button {
  position: absolute;
  width: 50px;
  height: 50px;
  padding: 0;
  border: none;
  background-color: white;
  background-size: contain;
  background-position: center;
  background-repeat: no-repeat;
  border-radius: 50%;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  top: 50%;
  left: 50%;
  margin: -25px 0 0 -25px;
  transition: all 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  z-index: 1;
  cursor: pointer;
}

.main-button {
  width: 60px;
  height: 60px;
  margin: -30px 0 0 -30px;
  z-index: 10;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
}

.main-button.active {
  transform: rotate(45deg);
  background-color: #f56565;
}

.category-button,
.statistics-button,
.thinking-button {
  opacity: 0;
  transform: translate(0, 0);
  visibility: hidden;
  pointer-events: none;
}

/* Thinking button special styling */
.thinking-button {
  background-color: #8a2be2;
  color: white;
  font-size: 18px;
}

/* 菜单打开状态的样式 */
.floating-button-group.menu-open .line-to-category,
.floating-button-group.menu-open .line-to-statistics,
.floating-button-group.menu-open .line-to-thinking {
  width: 90px;
  opacity: 1;
}

.floating-button-group.menu-open .category-button,
.floating-button-group.menu-open .statistics-button,
.floating-button-group.menu-open .thinking-button {
  opacity: 1;
  visibility: visible;
  pointer-events: auto;
}

.floating-button-group.menu-open .category-button {
  transform: rotate(210deg) translate(90px) rotate(-210deg);
}

.floating-button-group.menu-open .statistics-button {
  transform: rotate(240deg) translate(90px) rotate(-240deg);
}

.floating-button-group.menu-open .thinking-button {
  transform: rotate(270deg) translate(90px) rotate(-270deg);
}

/* 悬停效果 */
.floating-button:hover {
  transform: scale(1.1);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.25);
}

.main-button:hover {
  transform: scale(1.1);
}

.main-button.active:hover {
  transform: rotate(45deg) scale(1.1);
}

.floating-button-group.menu-open .category-button:hover {
  transform: rotate(210deg) translate(90px) rotate(-210deg) scale(1.1);
}

.floating-button-group.menu-open .statistics-button:hover {
  transform: rotate(240deg) translate(90px) rotate(-240deg) scale(1.1);
}

.floating-button-group.menu-open .thinking-button:hover {
  transform: rotate(270deg) translate(90px) rotate(-270deg) scale(1.1);
}

/* Dialog styling */
.dialog-footer {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .chat-app-container {
    grid-template-columns: 1fr;
  }

  .sidebar {
    position: fixed;
    left: 0;
    width: 80%;
    z-index: 1000;
    transform: translateX(-100%);
    transition: transform 0.3s ease;
  }

  .sidebar.active {
    transform: translateX(0);
  }

  .floating-button-group {
    bottom: 80px;
    right: 20px;
  }

  /* 移动端调整展开角度，避免超出屏幕 */
  .floating-button-group.menu-open .category-button {
    transform: rotate(180deg) translate(80px) rotate(-180deg);
  }

  .floating-button-group.menu-open .statistics-button {
    transform: rotate(225deg) translate(80px) rotate(-225deg);
  }

  .floating-button-group.menu-open .thinking-button {
    transform: rotate(270deg) translate(80px) rotate(-270deg);
  }
}
</style>