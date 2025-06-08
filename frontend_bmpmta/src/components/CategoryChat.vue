<template>
  <div class="chat-category-container">
    <div class="category-header">
      <h2>聊天分类</h2>
      <div class="header-actions">
        <el-button type="primary" size="small" @click="showNewCategoryDialog">
          <i class="el-icon-plus"></i> 新建分类
        </el-button>
        <el-button v-if="editMode" type="info" size="small" @click="toggleEditMode">
          完成
        </el-button>
        <el-button v-else type="info" size="small" @click="toggleEditMode">
          <i class="el-icon-edit"></i> 编辑
        </el-button>
      </div>
    </div>

    <div class="category-search">
      <el-input
          v-model="searchQuery"
          placeholder="搜索分类..."
          prefix-icon="el-icon-search"
          clearable
      />
    </div>

    <div class="category-view">
      <!-- 没有分类时显示的引导内容 -->
      <div v-if="filteredCategories.length === 0" class="empty-state">
        <div class="empty-state-icon">
          <i class="el-icon-collection-tag"></i>
        </div>
        <h3>暂无聊天分类</h3>
        <p>创建分类来组织您的聊天，便于管理和查找</p>
        <el-button type="primary" @click="showNewCategoryDialog">创建第一个分类</el-button>
      </div>

      <!-- 分类列表 -->
      <div v-else class="category-grid">
        <el-card
            v-for="category in filteredCategories"
            :key="category.id"
            :class="['category-card', { 'edit-mode': editMode }]"
            shadow="hover"
            @click="onCategoryClick(category)"
        >
          <!-- 编辑模式下显示的操作按钮 -->
          <div v-if="editMode" class="edit-actions">
            <el-button
                type="text"
                class="edit-btn"
                @click.stop="showEditCategoryDialog(category)"
            >
              <i class="el-icon-edit"></i>
            </el-button>
            <el-button
                type="text"
                class="delete-btn"
                @click.stop="showDeleteCategoryDialog(category)"
            >
              <i class="el-icon-delete"></i>
            </el-button>
          </div>

          <!-- 分类图标 -->
          <div class="category-icon" :style="{ backgroundColor: category.color }">
            <i :class="category.icon || 'el-icon-chat-dot-square'"></i>
          </div>

          <!-- 分类信息 -->
          <div class="category-info">
            <h3>{{ category.name }}</h3>
            <p class="category-description">{{ category.description }}</p>
            <div class="category-meta">
              <span class="chat-count">
                <i class="el-icon-chat-line-square"></i> {{ category.chatCount }}
              </span>
              <span class="updated-time">
                {{ formatTime(category.updatedAt) }}
              </span>
            </div>
          </div>
        </el-card>
      </div>
    </div>

    <!-- 选中分类后显示的聊天列表 -->
    <div v-if="selectedCategory" class="category-chats">
      <div class="category-chats-header">
        <el-button type="text" @click="selectedCategory = null">
          <i class="el-icon-back"></i> 返回分类
        </el-button>
        <h3>{{ selectedCategory.name }}</h3>
      </div>

      <div v-if="categoryChats.length > 0" class="chats-list">
        <el-card
            v-for="chat in categoryChats"
            :key="chat.id"
            class="chat-item"
            shadow="hover"
            @click="openChat(chat)"
        >
          <div class="chat-item-content">
            <h4>{{ chat.title }}</h4>
            <p class="chat-preview">{{ chat.preview }}</p>
            <div class="chat-meta">
              <span class="message-count">
                <i class="el-icon-chat-line-round"></i> {{ chat.messageCount }}条消息
              </span>
              <span class="updated-time">
                {{ formatTime(chat.updatedAt) }}
              </span>
            </div>
          </div>
        </el-card>
      </div>

      <div v-else class="empty-chats">
        <i class="el-icon-document"></i>
        <p>该分类下暂无聊天</p>
        <el-button type="primary" size="small" @click="createNewChatInCategory">
          新建聊天
        </el-button>
      </div>
    </div>

    <!-- 新建分类对话框 -->
    <el-dialog
        v-model="newCategoryDialogVisible"
        title="新建分类"
        width="400px"
    >
      <el-form :model="newCategory" label-position="top">
        <el-form-item label="分类名称" required>
          <el-input v-model="newCategory.name" placeholder="输入分类名称" />
        </el-form-item>
        <el-form-item label="描述">
          <el-input
              v-model="newCategory.description"
              type="textarea"
              placeholder="描述该分类的用途"
          />
        </el-form-item>
        <el-form-item label="图标">
          <div class="icon-selector">
            <div
                v-for="icon in availableIcons"
                :key="icon"
                :class="['icon-option', { active: newCategory.icon === icon }]"
                @click="newCategory.icon = icon"
            >
              <i :class="icon"></i>
            </div>
          </div>
        </el-form-item>
        <el-form-item label="颜色">
          <div class="color-selector">
            <div
                v-for="color in availableColors"
                :key="color"
                :class="['color-option', { active: newCategory.color === color }]"
                :style="{ backgroundColor: color }"
                @click="newCategory.color = color"
            >
            </div>
          </div>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="newCategoryDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="createCategory" :disabled="!newCategory.name">
            创建
          </el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 编辑分类对话框 -->
    <el-dialog
        v-model="editCategoryDialogVisible"
        title="编辑分类"
        width="400px"
    >
      <el-form :model="editingCategory" label-position="top" v-if="editingCategory">
        <el-form-item label="分类名称" required>
          <el-input v-model="editingCategory.name" placeholder="输入分类名称" />
        </el-form-item>
        <el-form-item label="描述">
          <el-input
              v-model="editingCategory.description"
              type="textarea"
              placeholder="描述该分类的用途"
          />
        </el-form-item>
        <el-form-item label="图标">
          <div class="icon-selector">
            <div
                v-for="icon in availableIcons"
                :key="icon"
                :class="['icon-option', { active: editingCategory.icon === icon }]"
                @click="editingCategory.icon = icon"
            >
              <i :class="icon"></i>
            </div>
          </div>
        </el-form-item>
        <el-form-item label="颜色">
          <div class="color-selector">
            <div
                v-for="color in availableColors"
                :key="color"
                :class="['color-option', { active: editingCategory.color === color }]"
                :style="{ backgroundColor: color }"
                @click="editingCategory.color = color"
            >
            </div>
          </div>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="editCategoryDialogVisible = false">取消</el-button>
          <el-button
              type="primary"
              @click="updateCategory"
              :disabled="!editingCategory || !editingCategory.name"
          >
            保存
          </el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 删除分类确认对话框 -->
    <el-dialog
        v-model="deleteCategoryDialogVisible"
        title="删除分类"
        width="400px"
    >
      <p>确定要删除分类 "{{ categoryToDelete?.name }}" 吗？</p>
      <p class="delete-warning">
        <i class="el-icon-warning"></i>
        此操作不会删除分类中的聊天，但它们将不再属于此分类。
      </p>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="deleteCategoryDialogVisible = false">取消</el-button>
          <el-button type="danger" @click="deleteCategory">
            删除
          </el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

// 状态变量
const searchQuery = ref('');
const editMode = ref(false);
const selectedCategory = ref(null);
const newCategoryDialogVisible = ref(false);
const editCategoryDialogVisible = ref(false);
const deleteCategoryDialogVisible = ref(false);
const editingCategory = ref(null);
const categoryToDelete = ref(null);

// 新分类表单数据
const newCategory = ref({
  name: '',
  description: '',
  icon: 'el-icon-chat-dot-square',
  color: '#409EFF'
});

// 可用图标列表
const availableIcons = [
  'el-icon-chat-dot-square',
  'el-icon-chat-line-square',
  'el-icon-chat-round',
  'el-icon-chat-line-round',
  'el-icon-s-order',
  'el-icon-s-cooperation',
  'el-icon-s-management',
  'el-icon-s-tools',
  'el-icon-s-marketing',
  'el-icon-trophy',
  'el-icon-collection',
  'el-icon-star-off',
  'el-icon-office-building',
  'el-icon-school',
  'el-icon-data-analysis',
  'el-icon-reading'
];

// 可用颜色列表
const availableColors = [
  '#409EFF', // 蓝色
  '#67C23A', // 绿色
  '#E6A23C', // 黄色
  '#F56C6C', // 红色
  '#909399', // 灰色
  '#9370DB', // 紫色
  '#00BFFF', // 天蓝色
  '#FF8C00', // 橙色
  '#008080', // 青色
  '#FF1493', // 粉色
  '#2E8B57', // 海绿色
  '#8B4513'  // 棕色
];

// 示例数据：分类列表
const categories = ref([
  {
    id: 'cat-001',
    name: '工作项目',
    description: '与工作相关的项目讨论和会议记录',
    icon: 'el-icon-s-cooperation',
    color: '#409EFF',
    chatCount: 5,
    updatedAt: Date.now() - 86400000 // 1天前
  },
  {
    id: 'cat-002',
    name: '学习笔记',
    description: '各类学习资料和笔记汇总',
    icon: 'el-icon-reading',
    color: '#67C23A',
    chatCount: 8,
    updatedAt: Date.now() - 3600000 // 1小时前
  },
  {
    id: 'cat-003',
    name: '创意构思',
    description: '头脑风暴和创意收集',
    icon: 'el-icon-trophy',
    color: '#E6A23C',
    chatCount: 3,
    updatedAt: Date.now() - 172800000 // 2天前
  },
  {
    id: 'cat-004',
    name: '个人助手',
    description: '日常任务和个人助理对话',
    icon: 'el-icon-s-management',
    color: '#9370DB',
    chatCount: 12,
    updatedAt: Date.now() - 7200000 // 2小时前
  }
]);

// 示例数据：某分类下的聊天列表
const categoryChats = ref([
  {
    id: 'chat-101',
    title: '周一项目会议记录',
    preview: '讨论了项目进度和下一步计划，分配了新任务...',
    messageCount: 28,
    updatedAt: Date.now() - 86400000 // 1天前
  },
  {
    id: 'chat-102',
    title: '产品设计讨论',
    preview: '分析了用户反馈，讨论了界面优化方案...',
    messageCount: 42,
    updatedAt: Date.now() - 43200000 // 12小时前
  },
  {
    id: 'chat-103',
    title: '营销策略规划',
    preview: '制定了下个季度的营销计划，包括社交媒体推广...',
    messageCount: 15,
    updatedAt: Date.now() - 7200000 // 2小时前
  },
  {
    id: 'chat-104',
    title: '客户反馈汇总',
    preview: '整理了最近一个月的客户反馈，重点关注了几个常见问题...',
    messageCount: 33,
    updatedAt: Date.now() - 3600000 // 1小时前
  },
  {
    id: 'chat-105',
    title: '技术架构评审',
    preview: '讨论了系统架构的优化方案，决定采用微服务架构...',
    messageCount: 56,
    updatedAt: Date.now() - 10800000 // 3小时前
  }
]);

// 根据搜索筛选分类
const filteredCategories = computed(() => {
  if (!searchQuery.value) {
    return categories.value;
  }

  return categories.value.filter(category =>
      category.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      category.description.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

// 切换编辑模式
const toggleEditMode = () => {
  editMode.value = !editMode.value;
};

// 格式化时间
const formatTime = (timestamp) => {
  const now = Date.now();
  const diff = now - timestamp;

  if (diff < 60000) return '刚刚';
  if (diff < 3600000) return `${Math.floor(diff / 60000)}分钟前`;
  if (diff < 86400000) return `${Math.floor(diff / 3600000)}小时前`;
  if (diff < 604800000) return `${Math.floor(diff / 86400000)}天前`;

  const date = new Date(timestamp);
  return `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`;
};

// 点击分类
const onCategoryClick = (category) => {
  if (editMode.value) return;
  selectedCategory.value = category;

  // 这里可以加载该分类下的聊天列表
  console.log(`加载分类 ${category.id} 下的聊天列表`);
};

// 在分类中打开聊天
const openChat = (chat) => {
  console.log(`打开聊天: ${chat.id}`);
  // 切换到聊天页面组件
  emit('change-component', 'ChatPage');
};

// 在当前分类中创建新聊天
const createNewChatInCategory = () => {
  console.log(`在分类 ${selectedCategory.value.id} 中创建新聊天`);

  // 这里应该创建新聊天并关联到当前分类
  // 然后切换到聊天页面
  emit('change-component', 'ChatPage');
};

// 显示新建分类对话框
const showNewCategoryDialog = () => {
  newCategory.value = {
    name: '',
    description: '',
    icon: 'el-icon-chat-dot-square',
    color: '#409EFF'
  };
  newCategoryDialogVisible.value = true;
};

// 创建新分类
const createCategory = () => {
  if (!newCategory.value.name) return;

  const newId = `cat-${String(categories.value.length + 1).padStart(3, '0')}`;

  categories.value.push({
    id: newId,
    name: newCategory.value.name,
    description: newCategory.value.description,
    icon: newCategory.value.icon,
    color: newCategory.value.color,
    chatCount: 0,
    updatedAt: Date.now()
  });

  newCategoryDialogVisible.value = false;
};

// 显示编辑分类对话框
const showEditCategoryDialog = (category) => {
  editingCategory.value = { ...category };
  editCategoryDialogVisible.value = true;
};

// 更新分类
const updateCategory = () => {
  if (!editingCategory.value || !editingCategory.value.name) return;

  const index = categories.value.findIndex(c => c.id === editingCategory.value.id);
  if (index !== -1) {
    categories.value[index] = {
      ...categories.value[index],
      name: editingCategory.value.name,
      description: editingCategory.value.description,
      icon: editingCategory.value.icon,
      color: editingCategory.value.color,
      updatedAt: Date.now()
    };
  }

  editCategoryDialogVisible.value = false;
};

// 显示删除分类确认对话框
const showDeleteCategoryDialog = (category) => {
  categoryToDelete.value = category;
  deleteCategoryDialogVisible.value = true;
};

// 删除分类
const deleteCategory = () => {
  if (!categoryToDelete.value) return;

  categories.value = categories.value.filter(c => c.id !== categoryToDelete.value.id);

  deleteCategoryDialogVisible.value = false;

  // 如果删除的是当前选中的分类，则清除选中状态
  if (selectedCategory.value && selectedCategory.value.id === categoryToDelete.value.id) {
    selectedCategory.value = null;
  }
};

// 定义组件事件
const emit = defineEmits(['change-component']);

// 组件挂载时执行
onMounted(() => {
  console.log('ChatCategory 组件已挂载');
});
</script>

<style scoped>
.chat-category-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.category-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.category-header h2 {
  margin: 0;
  color: #333;
  font-size: 24px;
  font-weight: 600;
}

.header-actions {
  display: flex;
  gap: 10px;
}

.category-search {
  margin-bottom: 20px;
}

.category-view {
  flex: 1;
  overflow: auto;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  text-align: center;
  height: 100%;
  min-height: 300px;
}

.empty-state-icon {
  font-size: 60px;
  color: #ccc;
  margin-bottom: 20px;
}

.empty-state h3 {
  margin: 0 0 10px;
  font-size: 20px;
  color: #333;
}

.empty-state p {
  color: #666;
  margin-bottom: 20px;
  font-size: 16px;
}

.category-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
}

.category-card {
  position: relative;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  padding: 16px;
  border-radius: 12px;
  overflow: hidden;
}

.category-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
}

.category-card.edit-mode {
  padding-top: 40px;
}

.edit-actions {
  position: absolute;
  top: 10px;
  right: 10px;
  display: flex;
  gap: 5px;
}

.edit-btn, .delete-btn {
  padding: 6px;
  font-size: 16px;
  border-radius: 50%;
}

.edit-btn:hover {
  color: #409EFF;
  background-color: rgba(64, 158, 255, 0.1);
}

.delete-btn:hover {
  color: #F56C6C;
  background-color: rgba(245, 108, 108, 0.1);
}

.category-icon {
  width: 60px;
  height: 60px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 16px;
  flex-shrink: 0;
}

.category-icon i {
  font-size: 30px;
  color: white;
}

.category-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.category-info h3 {
  margin: 0 0 8px;
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.category-description {
  font-size: 14px;
  color: #666;
  margin-bottom: 10px;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}

.category-meta {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
  color: #999;
}

.chat-count i, .updated-time i {
  margin-right: 5px;
}

/* 分类下的聊天列表 */
.category-chats {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #f8f9fa;
  z-index: 10;
  padding: 20px;
  display: flex;
  flex-direction: column;
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    transform: translateX(30px);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.category-chats-header {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.category-chats-header h3 {
  margin: 0 0 0 10px;
  font-size: 20px;
  font-weight: 600;
}

.chats-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
  overflow-y: auto;
  padding-right: 5px;
}

.chat-item {
  cursor: pointer;
  transition: all 0.2s ease;
}

.chat-item:hover {
  transform: translateX(5px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.chat-item-content {
  display: flex;
  flex-direction: column;
}

.chat-item-content h4 {
  margin: 0 0 8px;
  font-size: 16px;
  font-weight: 600;
  color: #333;
}

.chat-preview {
  font-size: 14px;
  color: #666;
  margin-bottom: 10px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.chat-meta {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
  color: #999;
}

.empty-chats {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  text-align: center;
  height: 100%;
  min-height: 200px;
}

.empty-chats i {
  font-size: 50px;
  color: #ccc;
  margin-bottom: 20px;
}

.empty-chats p {
  color: #666;
  margin-bottom: 20px;
  font-size: 16px;
}

/* 图标选择器 */
.icon-selector {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 10px;
}

.icon-option {
  width: 36px;
  height: 36px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  background-color: #f5f7fa;
  transition: all 0.2s;
}

.icon-option i {
  font-size: 20px;
  color: #606266;
}

.icon-option:hover {
  background-color: #ecf5ff;
}

.icon-option.active {
  background-color: #409EFF;
}

.icon-option.active i {
  color: white;
}

/* 颜色选择器 */
.color-selector {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 10px;
}

.color-option {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  cursor: pointer;
  position: relative;
  transition: all 0.2s;
}

.color-option:hover {
  transform: scale(1.1);
  box-shadow: 0 0 8px rgba(0, 0, 0, 0.2);
}

.color-option.active::after {
  content: '✓';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-weight: bold;
}

/* 删除警告 */
.delete-warning {
  color: #E6A23C;
  display: flex;
  align-items: center;
  font-size: 14px;
  background-color: #fdf6ec;
  padding: 10px;
  border-radius: 4px;
  margin-top: 10px;
}

.delete-warning i {
  margin-right: 5px;
  font-size: 16px;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .category-grid {
    grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  }

  .icon-selector {
    grid-template-columns: repeat(6, 1fr);
  }

  .color-selector {
    grid-template-columns: repeat(4, 1fr);
  }
}
</style>