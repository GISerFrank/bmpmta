<template>
  <div class="thinking-page" :class="{ 'dark-mode': isDarkMode }">
    <!-- 侧边栏 -->
    <div class="sidebar" :class="{ 'collapsed': sidebarCollapsed }">
      <!-- 顶部头部 -->
      <div class="sidebar-header">
        <button class="menu-toggle" @click="toggleSidebar">
          <span class="hamburger"></span>
        </button>
        <h1 v-if="!sidebarCollapsed" class="app-title">Thinking Space</h1>
        <button class="theme-toggle" @click="toggleTheme">
          <span v-if="isDarkMode">☀️</span>
          <span v-else>🌙</span>
        </button>
      </div>

      <!-- 搜索框 -->
      <div v-if="!sidebarCollapsed" class="search-section">
        <div class="search-container">
          <input
              v-model="searchQuery"
              type="text"
              placeholder="Search topics..."
              class="search-input"
          >
          <span class="search-icon">🔍</span>
        </div>
      </div>

      <!-- 导航菜单 -->
      <div v-if="!sidebarCollapsed" class="navigation">
        <div v-for="section in filteredSections" :key="section.id" class="menu-section">
          <button
              class="section-header"
              @click="toggleSection(section.id)"
              :class="{ 'expanded': section.expanded }"
          >
            <span class="section-icon">{{ section.emoji }}</span>
            <span class="section-title">{{ section.title }}</span>
            <span class="chevron">{{ section.expanded ? '▼' : '▶' }}</span>
          </button>

          <div v-show="section.expanded" class="section-content">
            <div v-for="category in section.categories" :key="category.id" class="category">
              <h4 class="category-title">{{ category.title }}</h4>
              <div class="menu-items">
                <button
                    v-for="item in category.items"
                    :key="item.id"
                    class="menu-item"
                    :class="{ 'active': activeItemId === item.id }"
                    @click="selectMenuItem(item)"
                >
                  <span class="item-text">{{ item.title }}</span>
                  <span v-if="item.count" class="item-badge">{{ item.count }}</span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 主内容区 -->
    <div class="main-content">
      <!-- 欢迎界面 -->
      <div v-if="showWelcome" class="welcome-container">
        <div class="welcome-header">
          <div class="title-animation">
            <h1 class="main-title">
              <span v-for="(letter, index) in titleLetters"
                    :key="index"
                    class="letter"
                    :style="{ animationDelay: index * 0.1 + 's' }">
                {{ letter }}
              </span>
            </h1>
          </div>
          <p class="subtitle">Your personal space for ideas, thoughts, and creativity</p>
        </div>

        <div class="welcome-cards">
          <div
              v-for="card in welcomeCards"
              :key="card.id"
              class="welcome-card"
              @click="selectWelcomeCard(card)"
          >
            <div class="card-icon">{{ card.emoji }}</div>
            <h3 class="card-title">{{ card.title }}</h3>
            <p class="card-description">{{ card.description }}</p>
          </div>
        </div>
      </div>

      <!-- 内容界面 -->
      <div v-else class="content-container">
        <!-- 内容头部 -->
        <div class="content-header">
          <div class="header-left">
            <button class="back-btn" @click="goBack">
              ← Back
            </button>
            <div class="header-info">
              <h2 class="content-title">{{ currentContent?.title }}</h2>
              <p v-if="currentContent?.description" class="content-desc">
                {{ currentContent.description }}
              </p>
            </div>
          </div>
          <div class="header-actions">
            <button
                v-if="currentContent?.type === 'note'"
                class="action-btn primary"
                @click="toggleEdit"
            >
              {{ isEditing ? '💾 Save' : '✏️ Edit' }}
            </button>
            <button class="action-btn secondary" @click="shareContent">
              📤 Share
            </button>
          </div>
        </div>

        <!-- 加载状态 -->
        <div v-if="loading" class="loading-state">
          <div class="spinner"></div>
          <p>Loading content...</p>
        </div>

        <!-- 错误状态 -->
        <div v-else-if="error" class="error-state">
          <div class="error-icon">⚠️</div>
          <h3>Something went wrong</h3>
          <p>{{ error }}</p>
          <button class="retry-btn" @click="retryLoad">Try Again</button>
        </div>

        <!-- 动态内容 -->
        <div v-else-if="currentContent" class="dynamic-content">
          <!-- 列表视图 -->
          <div v-if="currentContent.type === 'list'" class="list-content">
            <div class="list-controls">
              <div class="view-selector">
                <button
                    v-for="view in viewModes"
                    :key="view"
                    class="view-btn"
                    :class="{ 'active': currentView === view }"
                    @click="setView(view)"
                >
                  {{ view }}
                </button>
              </div>
              <button class="add-btn" @click="addNewItem">
                ➕ Add New
              </button>
            </div>

            <div class="items-container" :class="`view-${currentView}`">
              <div
                  v-for="item in currentContent.items"
                  :key="item.id"
                  class="list-item"
                  @click="openItem(item)"
              >
                <div class="item-main">
                  <div class="item-header">
                    <h3 class="item-title">{{ item.title }}</h3>
                    <span class="item-date">{{ formatDate(item.date) }}</span>
                  </div>
                  <p class="item-description">{{ item.description }}</p>
                  <div v-if="item.tags" class="item-tags">
                    <span v-for="tag in item.tags" :key="tag" class="tag">
                      {{ tag }}
                    </span>
                  </div>
                </div>
                <div class="item-actions">
                  <button @click.stop="editItem(item)" class="action-icon">✏️</button>
                  <button @click.stop="deleteItem(item)" class="action-icon">🗑️</button>
                </div>
              </div>
            </div>

            <div v-if="hasMore" class="load-more">
              <button @click="loadMore" :disabled="loadingMore" class="load-more-btn">
                {{ loadingMore ? 'Loading...' : 'Load More' }}
              </button>
            </div>
          </div>

          <!-- 笔记视图 -->
          <div v-else-if="currentContent.type === 'note'" class="note-content">
            <div class="note-editor">
              <textarea
                  v-if="isEditing"
                  v-model="noteText"
                  class="note-textarea"
                  placeholder="Start writing your thoughts..."
                  ref="noteTextarea"
              ></textarea>
              <div v-else class="note-display" v-html="formattedNoteText"></div>
            </div>

            <div class="note-footer">
              <span class="note-info">
                Last edited: {{ formatDate(currentContent.lastModified) }}
              </span>
              <span class="word-count">{{ wordCount }} words</span>
            </div>
          </div>

          <!-- 书籍视图 -->
          <div v-else-if="currentContent.type === 'book'" class="book-content">
            <div class="book-reader">
              <div class="book-page left-page">
                <h3>{{ currentBookData.left.title }}</h3>
                <div class="page-content">{{ currentBookData.left.content }}</div>
              </div>
              <div class="book-page right-page">
                <h3>{{ currentBookData.right.title }}</h3>
                <div class="page-content">{{ currentBookData.right.content }}</div>
              </div>
            </div>

            <div class="book-controls">
              <button @click="prevPage" :disabled="currentPage === 1" class="page-btn">
                ← Previous
              </button>
              <div class="page-info">
                <span>Page {{ currentPage }} of {{ totalPages }}</span>
                <div class="progress-bar">
                  <div class="progress" :style="{ width: progressWidth + '%' }"></div>
                </div>
              </div>
              <button @click="nextPage" :disabled="currentPage === totalPages" class="page-btn">
                Next →
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 通知弹窗 -->
    <div v-if="notification" class="notification" :class="notification.type">
      <span class="notification-icon">{{ getNotificationIcon(notification.type) }}</span>
      <span class="notification-text">{{ notification.message }}</span>
      <button @click="dismissNotification" class="close-btn">✕</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "ThinkingPage",
  data() {
    return {
      isDarkMode: false,
      sidebarCollapsed: false,
      searchQuery: '',
      showWelcome: true,
      activeItemId: null,
      loading: false,
      error: null,
      notification: null,
      isEditing: false,
      noteText: '',
      currentView: 'grid',
      currentPage: 1,
      totalPages: 5,
      loadingMore: false,
      hasMore: true,

      // 数据
      titleLetters: 'THINKING'.split(''),
      viewModes: ['grid', 'list', 'cards'],

      menuSections: [
        {
          id: 'life',
          title: 'Live a Life',
          emoji: '❤️',
          expanded: true,
          categories: [
            {
              id: 'activities',
              title: 'Activities',
              items: [
                { id: 'travel', title: 'Travel Adventures', count: 5 },
                { id: 'hobbies', title: 'Creative Hobbies', count: 8 }
              ]
            },
            {
              id: 'goals',
              title: 'Goals',
              items: [
                { id: 'bucket', title: 'Bucket List', count: 12 },
                { id: 'routines', title: 'Daily Routines', count: 3 }
              ]
            }
          ]
        },
        {
          id: 'learn',
          title: 'Learn Something',
          emoji: '💡',
          expanded: false,
          categories: [
            {
              id: 'topics',
              title: 'Topics',
              items: [
                { id: 'science', title: 'Science & Tech', count: 15 },
                { id: 'arts', title: 'Arts & Culture', count: 7 }
              ]
            }
          ]
        },
        {
          id: 'work',
          title: 'Work Hard',
          emoji: '💼',
          expanded: false,
          categories: [
            {
              id: 'productivity',
              title: 'Productivity',
              items: [
                { id: 'focus', title: 'Focus Techniques', count: 6 },
                { id: 'goals-work', title: 'Goal Setting', count: 9 }
              ]
            }
          ]
        },
        {
          id: 'think',
          title: 'Thinking Out Loud',
          emoji: '🤔',
          expanded: false,
          categories: [
            {
              id: 'reflections',
              title: 'Reflections',
              items: [
                { id: 'journal', title: 'Journal Entries', count: 18 },
                { id: 'random', title: 'Random Thoughts', count: 25 }
              ]
            }
          ]
        }
      ],

      welcomeCards: [
        {
          id: 'quick-note',
          title: 'Quick Note',
          description: 'Start writing your thoughts immediately',
          emoji: '📝',
          type: 'note'
        },
        {
          id: 'recent',
          title: 'Recent Entries',
          description: 'Continue where you left off',
          emoji: '📖',
          type: 'list'
        },
        {
          id: 'explore',
          title: 'Explore Ideas',
          description: 'Browse your creative thoughts',
          emoji: '🚀',
          type: 'list'
        }
      ],

      currentContent: null
    }
  },

  computed: {
    filteredSections() {
      if (!this.searchQuery) return this.menuSections

      return this.menuSections.map(section => ({
        ...section,
        categories: section.categories.map(category => ({
          ...category,
          items: category.items.filter(item =>
              item.title.toLowerCase().includes(this.searchQuery.toLowerCase())
          )
        })).filter(category => category.items.length > 0)
      })).filter(section => section.categories.length > 0)
    },

    formattedNoteText() {
      return this.noteText.replace(/\n/g, '<br>')
    },

    wordCount() {
      return this.noteText.trim() ? this.noteText.trim().split(/\s+/).length : 0
    },

    currentBookData() {
      if (!this.currentContent || this.currentContent.type !== 'book') {
        return { left: { title: '', content: '' }, right: { title: '', content: '' } }
      }

      const pageIndex = this.currentPage - 1
      return this.currentContent.pages[pageIndex] || {
        left: { title: '', content: '' },
        right: { title: '', content: '' }
      }
    },

    progressWidth() {
      return (this.currentPage / this.totalPages) * 100
    }
  },

  methods: {
    toggleTheme() {
      this.isDarkMode = !this.isDarkMode
      localStorage.setItem('thinking-theme', this.isDarkMode ? 'dark' : 'light')
    },

    toggleSidebar() {
      this.sidebarCollapsed = !this.sidebarCollapsed
    },

    toggleSection(sectionId) {
      const section = this.menuSections.find(s => s.id === sectionId)
      if (section) {
        section.expanded = !section.expanded
      }
    },

    selectMenuItem(item) {
      this.loading = true
      this.error = null
      this.activeItemId = item.id
      this.showWelcome = false

      // 模拟异步加载
      setTimeout(() => {
        this.currentContent = this.generateContent(item)
        if (this.currentContent.type === 'note') {
          this.noteText = this.currentContent.text || ''
        }
        this.loading = false
      }, 500)
    },

    selectWelcomeCard(card) {
      this.selectMenuItem({ id: card.id, type: card.type })
    },

    generateContent(item) {
      const contentTypes = {
        'quick-note': {
          type: 'note',
          title: 'Quick Note',
          description: 'Capture your thoughts instantly',
          text: 'Start writing here...',
          lastModified: new Date()
        },
        'travel': {
          type: 'list',
          title: 'Travel Adventures',
          description: 'Plan and document your journeys',
          items: [
            {
              id: 1,
              title: 'Japan Spring 2025',
              description: 'Cherry blossom season in Tokyo and Kyoto',
              date: new Date('2025-03-15'),
              tags: ['travel', 'culture', 'photography']
            },
            {
              id: 2,
              title: 'Iceland Ring Road',
              description: 'Complete circuit around the beautiful island',
              date: new Date('2024-08-20'),
              tags: ['adventure', 'nature', 'road-trip']
            },
            {
              id: 3,
              title: 'Italy Food Tour',
              description: 'Authentic cuisine across different regions',
              date: new Date('2024-05-10'),
              tags: ['food', 'culture', 'italy']
            }
          ]
        },
        'science': {
          type: 'book',
          title: 'Science & Technology',
          description: 'Latest discoveries and innovations',
          pages: [
            {
              left: {
                title: 'Quantum Computing',
                content: 'Quantum computing represents a fundamental shift in how we process information. Unlike classical computers that use bits, quantum computers use quantum bits or qubits...'
              },
              right: {
                title: 'Artificial Intelligence',
                content: 'AI continues to evolve at an unprecedented pace. Machine learning algorithms are becoming more sophisticated, enabling computers to learn and adapt...'
              }
            },
            {
              left: {
                title: 'Biotechnology',
                content: 'Recent advances in biotechnology are opening new frontiers in medicine and agriculture. CRISPR gene editing technology...'
              },
              right: {
                title: 'Space Exploration',
                content: 'Private companies are revolutionizing space exploration. Mars missions are becoming more realistic as technology advances...'
              }
            }
          ]
        },
        'journal': {
          type: 'note',
          title: 'Personal Journal',
          description: 'Your private thoughts and reflections',
          text: 'Today was an interesting day. I spent time reflecting on my goals and what I want to achieve this year. The weather was perfect for a walk, and I found myself thinking about...',
          lastModified: new Date()
        }
      }

      return contentTypes[item.id] || {
        type: 'note',
        title: item.title || 'New Content',
        description: 'This content is under development',
        text: 'Coming soon...',
        lastModified: new Date()
      }
    },

    goBack() {
      this.showWelcome = true
      this.currentContent = null
      this.activeItemId = null
      this.isEditing = false
    },

    toggleEdit() {
      this.isEditing = !this.isEditing
      if (this.isEditing) {
        this.$nextTick(() => {
          if (this.$refs.noteTextarea) {
            this.$refs.noteTextarea.focus()
          }
        })
      } else {
        // 保存内容
        if (this.currentContent) {
          this.currentContent.text = this.noteText
          this.currentContent.lastModified = new Date()
        }
        this.showNotification('Note saved successfully!', 'success')
      }
    },

    shareContent() {
      if (navigator.share) {
        navigator.share({
          title: this.currentContent?.title,
          text: this.currentContent?.description,
          url: window.location.href
        })
      } else {
        navigator.clipboard.writeText(window.location.href).then(() => {
          this.showNotification('Link copied to clipboard!', 'info')
        })
      }
    },

    setView(view) {
      this.currentView = view
    },

    addNewItem() {
      this.showNotification('Add new item feature coming soon!', 'info')
    },

    openItem(item) {
      console.log('Opening item:', item)
    },

    editItem(item) {
      this.showNotification(`Editing: ${item.title}`, 'info')
    },

    deleteItem(item) {
      this.showNotification(`Deleted: ${item.title}`, 'success')
    },

    loadMore() {
      this.loadingMore = true

      setTimeout(() => {
        // 模拟加载更多数据
        const newItems = [
          {
            id: Date.now(),
            title: `New Item ${Date.now()}`,
            description: 'Newly loaded content',
            date: new Date(),
            tags: ['new']
          }
        ]

        if (this.currentContent && this.currentContent.items) {
          this.currentContent.items.push(...newItems)
          if (this.currentContent.items.length >= 10) {
            this.hasMore = false
          }
        }

        this.loadingMore = false
      }, 1000)
    },

    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++
      }
    },

    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--
      }
    },

    retryLoad() {
      this.error = null
      if (this.activeItemId) {
        this.selectMenuItem({ id: this.activeItemId })
      }
    },

    showNotification(message, type = 'info') {
      this.notification = { message, type }
      setTimeout(() => {
        this.notification = null
      }, 3000)
    },

    dismissNotification() {
      this.notification = null
    },

    getNotificationIcon(type) {
      const icons = {
        success: '✅',
        error: '❌',
        info: 'ℹ️'
      }
      return icons[type] || 'ℹ️'
    },

    formatDate(date) {
      if (!date) return ''
      return new Date(date).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    }
  },

  mounted() {
    // 加载保存的主题
    const savedTheme = localStorage.getItem('thinking-theme')
    if (savedTheme === 'dark') {
      this.isDarkMode = true
    }

    // 键盘快捷键
    document.addEventListener('keydown', (e) => {
      if (e.ctrlKey || e.metaKey) {
        switch (e.key) {
          case 'k':
            e.preventDefault()
            // eslint-disable-next-line no-case-declarations
            const searchInput = document.querySelector('.search-input')
            if (searchInput) searchInput.focus()
            break
          case 's':
            e.preventDefault()
            if (this.currentContent?.type === 'note') {
              this.toggleEdit()
            }
            break
          case 'b':
            e.preventDefault()
            this.toggleSidebar()
            break
        }
      }

      if (e.key === 'Escape') {
        if (this.isEditing) {
          this.isEditing = false
        } else if (!this.showWelcome) {
          this.goBack()
        }
      }
    })
  }
}
</script>

<style scoped>
/* 基础重置和变量 */
* {
  box-sizing: border-box;
}

.thinking-page {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  display: flex;
  min-height: 100vh;
  background-color: #f8fafc;
  color: #1a202c;
  transition: all 0.3s ease;
}

.dark-mode {
  background-color: #1a202c;
  color: #f7fafc;
}

/* 侧边栏样式 */
.sidebar {
  width: 320px;
  background-color: white;
  border-right: 1px solid #e2e8f0;
  transition: width 0.3s ease;
  flex-shrink: 0;
  overflow: hidden;
}

.dark-mode .sidebar {
  background-color: #2d3748;
  border-color: #4a5568;
}

.sidebar.collapsed {
  width: 60px;
}

.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem;
  border-bottom: 1px solid #e2e8f0;
  gap: 0.5rem;
}

.dark-mode .sidebar-header {
  border-color: #4a5568;
}

.menu-toggle,
.theme-toggle {
  background: none;
  border: none;
  padding: 0.5rem;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
}

.menu-toggle:hover,
.theme-toggle:hover {
  background-color: #f1f5f9;
}

.dark-mode .menu-toggle:hover,
.dark-mode .theme-toggle:hover {
  background-color: #4a5568;
}

.hamburger {
  display: block;
  width: 18px;
  height: 2px;
  background-color: #4a5568;
  position: relative;
}

.hamburger::before,
.hamburger::after {
  content: '';
  display: block;
  width: 18px;
  height: 2px;
  background-color: #4a5568;
  position: absolute;
}

.hamburger::before {
  top: -6px;
}

.hamburger::after {
  bottom: -6px;
}

.dark-mode .hamburger,
.dark-mode .hamburger::before,
.dark-mode .hamburger::after {
  background-color: #f7fafc;
}

.app-title {
  font-size: 1.25rem;
  font-weight: 600;
  margin: 0;
  color: #2d3748;
}

.dark-mode .app-title {
  color: #f7fafc;
}

.search-section {
  padding: 1rem;
  border-bottom: 1px solid #e2e8f0;
}

.dark-mode .search-section {
  border-color: #4a5568;
}

.search-container {
  position: relative;
}

.search-input {
  width: 100%;
  padding: 0.75rem 2.5rem 0.75rem 1rem;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  background-color: #f8fafc;
  font-size: 0.875rem;
  transition: all 0.2s;
}

.search-input:focus {
  outline: none;
  border-color: #6366f1;
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
}

.dark-mode .search-input {
  background-color: #4a5568;
  border-color: #718096;
  color: #f7fafc;
}

.search-icon {
  position: absolute;
  right: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  color: #a0aec0;
}

.navigation {
  flex: 1;
  overflow-y: auto;
  padding: 1rem 0;
}

.menu-section {
  margin-bottom: 0.5rem;
}

.section-header {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1rem;
  background: none;
  border: none;
  text-align: left;
  cursor: pointer;
  transition: background-color 0.2s;
  color: inherit;
}

.section-header:hover {
  background-color: #f1f5f9;
}

.dark-mode .section-header:hover {
  background-color: #4a5568;
}

.section-icon {
  font-size: 1rem;
}

.section-title {
  flex: 1;
  font-weight: 500;
  font-size: 0.875rem;
}

.chevron {
  font-size: 0.75rem;
  transition: transform 0.2s;
}

.section-header.expanded .chevron {
  transform: rotate(90deg);
}

.section-content {
  padding-left: 1rem;
}

.category {
  margin-bottom: 1rem;
}

.category-title {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #718096;
  margin: 0 0 0.5rem 1rem;
}

.menu-items {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.menu-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.5rem 1rem;
  background: none;
  border: none;
  text-align: left;
  cursor: pointer;
  border-radius: 0 6px 6px 0;
  margin-right: 0.5rem;
  transition: all 0.2s;
  color: inherit;
}

.menu-item:hover {
  background-color: #f1f5f9;
}

.menu-item.active {
  background-color: #6366f1;
  color: white;
}

.dark-mode .menu-item:hover {
  background-color: #4a5568;
}

.item-text {
  font-size: 0.875rem;
}

.item-badge {
  font-size: 0.75rem;
  background-color: #e2e8f0;
  color: #4a5568;
  padding: 0.125rem 0.375rem;
  border-radius: 12px;
  min-width: 1.25rem;
  text-align: center;
}

.menu-item.active .item-badge {
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
}

/* 主内容区样式 */
.main-content {
  flex: 1;
  overflow-y: auto;
  background-color: #f8fafc;
}

.dark-mode .main-content {
  background-color: #1a202c;
}

/* 欢迎界面 */
.welcome-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 4rem 2rem;
  text-align: center;
}

.welcome-header {
  margin-bottom: 4rem;
}

.title-animation {
  margin-bottom: 2rem;
}

.main-title {
  font-size: 4rem;
  font-weight: 800;
  margin: 0;
  display: flex;
  justify-content: center;
  gap: 0.1em;
}

.letter {
  display: inline-block;
  background: linear-gradient(45deg, #6366f1, #8b5cf6, #ec4899);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: bounce 2s ease-in-out infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.subtitle {
  font-size: 1.25rem;
  color: #64748b;
  margin: 0;
}

.dark-mode .subtitle {
  color: #a0aec0;
}

.welcome-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.welcome-card {
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 2rem;
  cursor: pointer;
  transition: all 0.3s ease;
  text-align: center;
}

.welcome-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
  border-color: #6366f1;
}

.dark-mode .welcome-card {
  background-color: #2d3748;
  border-color: #4a5568;
}

.dark-mode .welcome-card:hover {
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
}

.card-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.card-title {
  font-size: 1.25rem;
  font-weight: 600;
  margin: 0 0 0.5rem 0;
  color: #2d3748;
}

.dark-mode .card-title {
  color: #f7fafc;
}

.card-description {
  color: #64748b;
  margin: 0;
}

.dark-mode .card-description {
  color: #a0aec0;
}

/* 内容界面 */
.content-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.content-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  margin-bottom: 2rem;
  gap: 1rem;
}

.header-left {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
  flex: 1;
}

.back-btn {
  background: none;
  border: 1px solid #e2e8f0;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  color: #64748b;
  font-size: 0.875rem;
  transition: all 0.2s;
}

.back-btn:hover {
  background-color: #f1f5f9;
  border-color: #6366f1;
  color: #6366f1;
}

.dark-mode .back-btn {
  border-color: #4a5568;
  color: #a0aec0;
}

.dark-mode .back-btn:hover {
  background-color: #4a5568;
  border-color: #6366f1;
  color: #6366f1;
}

.header-info {
  flex: 1;
}

.content-title {
  font-size: 1.875rem;
  font-weight: 700;
  margin: 0 0 0.5rem 0;
  color: #1a202c;
}

.dark-mode .content-title {
  color: #f7fafc;
}

.content-desc {
  color: #64748b;
  margin: 0;
}

.dark-mode .content-desc {
  color: #a0aec0;
}

.header-actions {
  display: flex;
  gap: 0.75rem;
  flex-shrink: 0;
}

.action-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.875rem;
  font-weight: 500;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.action-btn.primary {
  background-color: #6366f1;
  color: white;
}

.action-btn.primary:hover {
  background-color: #5855eb;
}

.action-btn.secondary {
  background-color: #f1f5f9;
  color: #64748b;
  border: 1px solid #e2e8f0;
}

.action-btn.secondary:hover {
  background-color: #e2e8f0;
  color: #1a202c;
}

.dark-mode .action-btn.secondary {
  background-color: #4a5568;
  color: #a0aec0;
  border-color: #718096;
}

.dark-mode .action-btn.secondary:hover {
  background-color: #718096;
  color: #f7fafc;
}

/* 状态样式 */
.loading-state,
.error-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 4rem 2rem;
  text-align: center;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #e2e8f0;
  border-top: 3px solid #6366f1;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 1rem;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.retry-btn {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  background-color: #6366f1;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.retry-btn:hover {
  background-color: #5855eb;
}

/* 动态内容样式 */
.dynamic-content {
  background-color: white;
  border-radius: 12px;
  border: 1px solid #e2e8f0;
  overflow: hidden;
}

.dark-mode .dynamic-content {
  background-color: #2d3748;
  border-color: #4a5568;
}

/* 列表内容 */
.list-content {
  padding: 2rem;
}

.list-controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 2rem;
}

.view-selector {
  display: flex;
  gap: 0.5rem;
}

.view-btn {
  padding: 0.5rem 1rem;
  background: none;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.875rem;
  color: #64748b;
  transition: all 0.2s;
}

.view-btn:hover,
.view-btn.active {
  background-color: #6366f1;
  color: white;
  border-color: #6366f1;
}

.dark-mode .view-btn {
  border-color: #4a5568;
  color: #a0aec0;
}

.add-btn {
  padding: 0.5rem 1rem;
  background-color: #6366f1;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: background-color 0.2s;
}

.add-btn:hover {
  background-color: #5855eb;
}

.items-container {
  display: grid;
  gap: 1.5rem;
}

.view-grid .items-container {
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
}

.view-list .items-container {
  grid-template-columns: 1fr;
}

.view-cards .items-container {
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
}

.list-item {
  background-color: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1.5rem;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.list-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border-color: #6366f1;
}

.dark-mode .list-item {
  background-color: #4a5568;
  border-color: #718096;
}

.item-main {
  flex: 1;
}

.item-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  margin-bottom: 0.5rem;
}

.item-title {
  font-size: 1.125rem;
  font-weight: 600;
  margin: 0;
  color: #1a202c;
}

.dark-mode .item-title {
  color: #f7fafc;
}

.item-date {
  font-size: 0.75rem;
  color: #a0aec0;
  margin-left: 1rem;
}

.item-description {
  color: #64748b;
  margin: 0 0 1rem 0;
  line-height: 1.5;
}

.dark-mode .item-description {
  color: #a0aec0;
}

.item-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tag {
  font-size: 0.75rem;
  padding: 0.25rem 0.5rem;
  background-color: #e2e8f0;
  color: #64748b;
  border-radius: 12px;
}

.dark-mode .tag {
  background-color: #718096;
  color: #f7fafc;
}

.item-actions {
  display: flex;
  gap: 0.5rem;
  opacity: 0;
  transition: opacity 0.2s;
}

.list-item:hover .item-actions {
  opacity: 1;
}

.action-icon {
  background: none;
  border: none;
  padding: 0.375rem;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.action-icon:hover {
  background-color: #e2e8f0;
}

.dark-mode .action-icon:hover {
  background-color: #718096;
}

.load-more {
  text-align: center;
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid #e2e8f0;
}

.dark-mode .load-more {
  border-color: #4a5568;
}

.load-more-btn {
  padding: 0.75rem 1.5rem;
  background-color: #6366f1;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: background-color 0.2s;
}

.load-more-btn:hover:not(:disabled) {
  background-color: #5855eb;
}

.load-more-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* 笔记内容 */
.note-content {
  padding: 2rem;
}

.note-editor {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  overflow: hidden;
  margin-bottom: 1rem;
}

.dark-mode .note-editor {
  border-color: #4a5568;
}

.note-textarea {
  width: 100%;
  min-height: 400px;
  padding: 1.5rem;
  border: none;
  background-color: #f8fafc;
  color: #1a202c;
  font-family: inherit;
  font-size: 1rem;
  line-height: 1.6;
  resize: vertical;
}

.note-textarea:focus {
  outline: none;
}

.dark-mode .note-textarea {
  background-color: #4a5568;
  color: #f7fafc;
}

.note-display {
  padding: 1.5rem;
  min-height: 400px;
  line-height: 1.6;
  background-color: #f8fafc;
  color: #1a202c;
}

.dark-mode .note-display {
  background-color: #4a5568;
  color: #f7fafc;
}

.note-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #a0aec0;
  font-size: 0.875rem;
  padding-top: 1rem;
  border-top: 1px solid #e2e8f0;
}

.dark-mode .note-footer {
  border-color: #4a5568;
}

/* 书籍内容 */
.book-content {
  padding: 2rem;
}

.book-reader {
  display: flex;
  background-color: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  overflow: hidden;
  margin-bottom: 2rem;
  min-height: 500px;
}

.dark-mode .book-reader {
  background-color: #4a5568;
  border-color: #718096;
}

.book-page {
  flex: 1;
  padding: 2rem;
  border-right: 1px solid #e2e8f0;
}

.book-page:last-child {
  border-right: none;
}

.dark-mode .book-page {
  border-color: #718096;
}

.book-page h3 {
  font-size: 1.25rem;
  font-weight: 600;
  margin: 0 0 1rem 0;
  color: #1a202c;
  border-bottom: 1px solid #e2e8f0;
  padding-bottom: 0.5rem;
}

.dark-mode .book-page h3 {
  color: #f7fafc;
  border-color: #718096;
}

.page-content {
  line-height: 1.7;
  color: #1a202c;
}

.dark-mode .page-content {
  color: #f7fafc;
}

.book-controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.page-btn {
  padding: 0.5rem 1rem;
  background-color: #6366f1;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: background-color 0.2s;
}

.page-btn:hover:not(:disabled) {
  background-color: #5855eb;
}

.page-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background-color: #a0aec0;
}

.page-info {
  text-align: center;
  color: #64748b;
}

.dark-mode .page-info {
  color: #a0aec0;
}

.progress-bar {
  width: 200px;
  height: 4px;
  background-color: #e2e8f0;
  border-radius: 2px;
  margin-top: 0.5rem;
  overflow: hidden;
}

.dark-mode .progress-bar {
  background-color: #4a5568;
}

.progress {
  height: 100%;
  background-color: #6366f1;
  transition: width 0.3s ease;
}

/* 通知样式 */
.notification {
  position: fixed;
  top: 2rem;
  right: 2rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem 1.25rem;
  background-color: white;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 1000;
  max-width: 400px;
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.dark-mode .notification {
  background-color: #2d3748;
  border-color: #4a5568;
}

.notification.success {
  border-color: #10b981;
}

.notification.error {
  border-color: #ef4444;
}

.notification.info {
  border-color: #6366f1;
}

.notification-icon {
  font-size: 1.25rem;
  flex-shrink: 0;
}

.notification-text {
  flex: 1;
  color: #1a202c;
}

.dark-mode .notification-text {
  color: #f7fafc;
}

.close-btn {
  background: none;
  border: none;
  color: #a0aec0;
  cursor: pointer;
  padding: 0.25rem;
  border-radius: 4px;
  transition: all 0.2s;
  font-size: 1rem;
  line-height: 1;
}

.close-btn:hover {
  background-color: #f1f5f9;
  color: #1a202c;
}

.dark-mode .close-btn:hover {
  background-color: #4a5568;
  color: #f7fafc;
}

/* 响应式设计 */
@media (max-width: 1024px) {
  .content-container {
    padding: 1rem;
  }

  .welcome-cards {
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .items-container {
    grid-template-columns: 1fr !important;
  }
}

@media (max-width: 768px) {
  .thinking-page {
    flex-direction: column;
  }

  .sidebar {
    width: 100%;
    height: auto;
  }

  .sidebar.collapsed {
    width: 100%;
    height: 60px;
  }

  .navigation {
    display: none;
  }

  .sidebar.collapsed .navigation,
  .sidebar.collapsed .search-section {
    display: none;
  }

  .content-header {
    flex-direction: column;
    align-items: stretch;
    gap: 1rem;
  }

  .header-left {
    flex-direction: column;
    gap: 0.5rem;
  }

  .list-controls {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }

  .view-selector {
    justify-content: center;
  }

  .book-reader {
    flex-direction: column;
  }

  .book-page {
    border-right: none;
    border-bottom: 1px solid #e2e8f0;
  }

  .book-page:last-child {
    border-bottom: none;
  }

  .dark-mode .book-page {
    border-color: #718096;
  }

  .book-controls {
    flex-direction: column;
    gap: 1rem;
  }

  .main-title {
    font-size: 2.5rem;
  }

  .notification {
    top: 1rem;
    right: 1rem;
    left: 1rem;
    max-width: none;
  }
}

/* 无障碍和可访问性 */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

button:focus,
input:focus,
textarea:focus {
  outline: 2px solid #6366f1;
  outline-offset: 2px;
}

@media (prefers-contrast: high) {
  .thinking-page {
    border: 2px solid;
  }

  .sidebar {
    border-right: 2px solid;
  }
}
</style>