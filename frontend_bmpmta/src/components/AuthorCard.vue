<template>
  <div class="author-section">
    <!-- 作者卡片（初始只显示这部分） -->
    <div class="author-card-container">
      <!-- 当没有翻转时显示的卡片正面 -->
      <div
          class="author-card"
          :class="{ 'flipped': isCardFlipped }"
          @click="flipCard"
          v-if="!isCardFlipped || !isBackContentLoaded"
      >
        <div class="card-front">
          <div class="card-question">
            <i class="el-icon-question question-icon"></i>
            <div class="question-text">今日推荐作家是谁？</div>
            <div class="tip-text">点击卡片揭晓</div>
          </div>
        </div>
      </div>

      <!-- 翻转后显示的卡片背面 -->
      <div
          class="author-card flipped"
          v-if="isCardFlipped"
          @click="showDetails"
      >
        <div class="card-back">
          <div class="author-photo-container">
            <el-image
                :src="author.photo"
                fit="cover"
                class="author-photo"
                v-loading="photoLoading"
            >
              <template #placeholder>
                <div class="image-slot">加载中<span class="dot">...</span></div>
              </template>
              <template #error>
                <div class="image-slot">
                  <i class="el-icon-picture-outline"></i>
                </div>
              </template>
            </el-image>
          </div>
          <div class="author-name">{{ author.name }}</div>
          <div class="author-brief">{{ author.brief }}</div>
          <div class="card-tip" v-if="!showFullDetails">点击查看详情</div>
        </div>
      </div>
    </div>

    <!-- 作者详情模态框 -->
    <el-dialog
        v-model="showFullDetails"
        :title="author.name + ' 作家档案'"
        width="60%"
        :before-close="closeDetails"
        destroy-on-close
        top="5vh"
    >
      <div class="author-details" v-loading="loading">
        <div class="details-content">
          <div class="author-header">
            <div class="author-portrait">
              <el-image
                  :src="author.photo"
                  fit="cover"
                  class="details-photo"
              ></el-image>
            </div>
            <div class="author-summary">
              <div class="author-title">{{ author.name }}</div>
              <div class="author-subtitle">{{ author.brief }}</div>
              <div class="author-actions">
                <el-button
                    type="primary"
                    size="small"
                    @click="refreshAuthor"
                    :loading="loading"
                >
                  换一位作家
                </el-button>
                <el-button
                    type="info"
                    size="small"
                    @click="searchAuthor"
                    :loading="loading"
                >
                  搜索作家
                </el-button>
              </div>
            </div>
          </div>

          <el-divider></el-divider>

          <el-tabs v-model="activeTab" class="author-tabs">
            <el-tab-pane label="生平简介" name="bio">
              <p class="tab-content">{{ author.bio }}</p>
            </el-tab-pane>

            <el-tab-pane label="写作风格" name="style">
              <p class="tab-content">{{ author.style }}</p>
            </el-tab-pane>

            <el-tab-pane label="代表作品" name="works">
              <ul class="works-list tab-content">
                <li v-for="(work, index) in author.works" :key="index">
                  {{ work }}
                </li>
              </ul>
            </el-tab-pane>

            <el-tab-pane v-if="author.awards && author.awards.length > 0" label="获奖情况" name="awards">
              <ul class="awards-list tab-content">
                <li v-for="(award, index) in author.awards" :key="index">
                  {{ award }}
                </li>
              </ul>
            </el-tab-pane>

            <el-tab-pane v-if="author.impact" label="文学影响" name="impact">
              <p class="tab-content">{{ author.impact }}</p>
            </el-tab-pane>
          </el-tabs>
        </div>
      </div>

      <template #footer>
        <span class="dialog-footer">
          <el-button @click="closeDetails">关闭</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 作家搜索弹窗 -->
    <el-dialog
        title="搜索作家"
        v-model="searchDialogVisible"
        width="500px"
        destroy-on-close
    >
      <div class="search-form">
        <el-input
            v-model="searchQuery"
            placeholder="输入作家姓名"
            clearable
            @keyup.enter="handleSearch"
        >
          <template #append>
            <el-button @click="handleSearch" :loading="searchLoading">搜索</el-button>
          </template>
        </el-input>

        <div class="genre-selector">
          <span class="genre-label">按类型推荐:</span>
          <el-select v-model="selectedGenre" class="genre-select">
            <el-option
                v-for="genre in genres"
                :key="genre"
                :label="genre"
                :value="genre">
            </el-option>
          </el-select>
          <el-button
              type="primary"
              @click="searchByGenre"
              :loading="searchLoading"
          >
            推荐
          </el-button>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios';
import { ElMessage } from 'element-plus';

export default {
  name: 'AuthorCard',
  data() {
    return {
      isCardFlipped: false,
      isBackContentLoaded: false,
      showFullDetails: false,
      loading: false,
      photoLoading: false,
      searchDialogVisible: false,
      searchQuery: '',
      searchLoading: false,
      selectedGenre: '文学',
      activeTab: 'bio',
      genres: ['文学', '科幻', '奇幻', '悬疑', '历史', '哲学', '诗歌', '散文'],

      author: {
        name: '',
        brief: '',
        bio: '',
        photo: '',
        style: '',
        works: [],
        awards: [],
        impact: ''
      },

      // 预设作家数据（作为备用）
      defaultAuthors: [
        {
          name: '莫言',
          brief: '中国当代作家，2012年诺贝尔文学奖获得者',
          bio: '莫言，原名管谟业，1955年2月17日出生于山东高密，中国当代作家，2012年诺贝尔文学奖获得者。其创作以乡土经验为基础，以民间故事、神话传说等为元素，创造了"魔幻现实主义"的独特风格。',
          photo: 'https://placehold.co/300x400?text=莫言',
          style: '结合了民间故事、历史与当代社会现实，形成了独特的"魔幻现实主义"风格，充满奇诡想象与现实批判。',
          works: ['《红高粱家族》', '《丰乳肥臀》', '《酒国》', '《生死疲劳》', '《蛙》'],
          awards: ['2012年诺贝尔文学奖', '第8届茅盾文学奖'],
          impact: '莫言的作品在国内外产生了广泛影响，被翻译成多种语言，其创作为中国文学在世界舞台上的传播做出了重要贡献。'
        },
        {
          name: '余华',
          brief: '中国当代著名作家，代表作《活着》',
          bio: '余华，1960年4月3日出生于浙江杭州，中国当代著名作家。早期创作带有先锋性质，后期转向现实主义，作品多次被改编为电影，产生广泛影响。',
          photo: 'https://placehold.co/300x400?text=余华',
          style: '早期作品具有先锋实验性质，后期转向现实主义，文风朴素冷静，常以平静的语调描述残酷的现实，形成强烈的艺术效果。',
          works: ['《活着》', '《许三观卖血记》', '《兄弟》', '《第七天》', '《文城》'],
          awards: ['法国艺术文学骑士勋章', '意大利格林扎纳·卡佛文学奖'],
          impact: '余华的创作对中国当代文学产生了深远影响，特别是《活着》成为现当代文学的经典之作，被翻译成多种语言在全球出版。'
        },
        {
          name: '鲁迅',
          brief: '中国现代文学的奠基人，思想家、文学家',
          bio: '鲁迅（1881年9月25日-1936年10月19日），原名周树人，浙江绍兴人，中国现代文学的奠基人，思想家、文学家、翻译家、教育家。其作品深刻揭露和批判了旧中国的社会现实，对中国文化和国民性进行了深刻反思。',
          photo: 'https://placehold.co/300x400?text=鲁迅',
          style: '文字锋利犀利，擅长运用象征和隐喻，以冷峻的笔调和讽刺的手法揭露社会问题，文章结构严谨，语言精炼有力。',
          works: ['《狂人日记》', '《阿Q正传》', '《孔乙己》', '《药》', '《野草》'],
          awards: [],
          impact: '鲁迅的创作开创了中国现代文学的新方向，对后世产生了深远影响。他的思想和精神成为中国知识分子的重要精神资源，其作品成为中国现代文学的经典。'
        }
      ]
    };
  },
  mounted() {
    // 预加载默认作家数据，为翻转后的显示做准备
    this.prepareAuthorData();
  },
  methods: {
    // 准备作家数据（随机选择一位预设作家或通过API获取）
    prepareAuthorData() {
      // 为演示目的，先使用预设数据
      const randomIndex = Math.floor(Math.random() * this.defaultAuthors.length);
      const defaultAuthor = this.defaultAuthors[randomIndex];

      // 设置作家数据
      this.author = { ...defaultAuthor };

      // 在实际应用中，这里可以调用API获取作家数据
      // this.fetchAuthorFromAPI();
    },

    // 翻转卡片
    flipCard() {
      if (!this.isCardFlipped) {
        this.isCardFlipped = true;
        this.photoLoading = true;

        // 模拟照片加载
        setTimeout(() => {
          this.photoLoading = false;
          this.isBackContentLoaded = true;
        }, 800);
      }
    },

    // 显示作家详细信息
    showDetails() {
      if (this.isCardFlipped && !this.showFullDetails) {
        this.showFullDetails = true;
      }
    },

    // 关闭详情
    closeDetails() {
      this.showFullDetails = false;
    },

    // 刷新作家
    refreshAuthor() {
      this.loading = true;

      // 重置状态
      this.isCardFlipped = false;
      this.showFullDetails = false;
      this.isBackContentLoaded = false;

      // 获取新的作家数据
      setTimeout(() => {
        this.prepareAuthorData();
        this.loading = false;
        // 如果模态框打开，先关闭
        this.showFullDetails = false;
      }, 1000);
    },

    // 打开搜索对话框
    searchAuthor() {
      this.searchDialogVisible = true;
      this.searchQuery = '';
    },

    // 处理搜索
    handleSearch() {
      if (!this.searchQuery || this.searchQuery.trim() === '') {
        ElMessage.warning('请输入要搜索的作家姓名');
        return;
      }

      this.searchLoading = true;

      // 这里调用API搜索作家
      // 在实际应用中，替换为真实的API调用
      setTimeout(() => {
        // 模拟API返回结果
        const foundAuthor = this.defaultAuthors.find(
            author => author.name.includes(this.searchQuery)
        );

        if (foundAuthor) {
          this.author = { ...foundAuthor };
          this.searchDialogVisible = false;
          this.isCardFlipped = true;
          this.isBackContentLoaded = true;
          ElMessage.success(`已找到作家: ${foundAuthor.name}`);
        } else {
          ElMessage.warning(`未找到作家: ${this.searchQuery}`);
        }

        this.searchLoading = false;
      }, 1500);
    },

    // 按类型搜索
    searchByGenre() {
      this.searchLoading = true;

      // 这里调用API按类型搜索作家
      // 在实际应用中，替换为真实的API调用
      setTimeout(() => {
        // 模拟API返回结果
        const randomIndex = Math.floor(Math.random() * this.defaultAuthors.length);
        const recommendedAuthor = this.defaultAuthors[randomIndex];

        this.author = { ...recommendedAuthor };
        this.searchDialogVisible = false;
        this.isCardFlipped = true;
        this.isBackContentLoaded = true;
        ElMessage.success(`为您推荐${this.selectedGenre}作家: ${recommendedAuthor.name}`);

        this.searchLoading = false;
      }, 1500);
    },

    // 从API获取作家信息
    async fetchAuthorFromAPI() {
      try {
        // 使用Claude API搜索作家信息
        // 这部分代码应替换为实际的API调用
        const claudeApiEndpoint = 'https://api.anthropic.com/v1/messages';
        const claudeApiKey = 'your-claude-api-key'; // 替换为实际API密钥

        const prompt = `请推荐一位著名作家，并提供以下内容：
1. 姓名
2. 简短介绍（一句话）
3. 生平简介
4. 写作风格
5. 代表作品（3-5部）
6. 获得的奖项
7. 文学影响

请以JSON格式返回:
{
  "name": "作家姓名",
  "brief": "简短介绍",
  "bio": "生平简介",
  "style": "写作风格",
  "works": ["作品1", "作品2", "作品3"],
  "awards": ["奖项1", "奖项2"],
  "impact": "文学影响"
}`;

        const response = await axios.post(claudeApiEndpoint,
            {
              model: "claude-3-opus-20240229",
              max_tokens: 1024,
              messages: [
                {
                  role: "user",
                  content: [{ type: "text", text: prompt }]
                }
              ],
              system: "你是一个专业的文学百科全书，擅长推荐各类型的作家。"
            },
            {
              headers: {
                'x-api-key': claudeApiKey,
                'anthropic-version': '2023-06-01',
                'Content-Type': 'application/json'
              }
            }
        );

        if (response.data && response.data.content) {
          const content = response.data.content[0].text;
          const authorInfo = JSON.parse(content);

          // 设置作家信息
          this.author = {
            ...authorInfo,
            photo: `https://placehold.co/300x400?text=${encodeURIComponent(authorInfo.name)}`
          };

          // 尝试获取作家照片
          this.getAuthorPhoto(authorInfo.name);
        }
      } catch (error) {
        console.error("获取作家信息时出错:", error);
        // 回退到默认数据
        this.prepareAuthorData();
      }
    },

    // 获取作家照片
    async getAuthorPhoto(authorName) {
      try {
        // 使用图片搜索API获取作家照片
        // 这里使用占位图作为示例
        this.author.photo = `https://placehold.co/300x400?text=${encodeURIComponent(authorName)}`;

        // 在实际应用中，这里应该调用图片搜索API
      } catch (error) {
        console.error("获取作家照片时出错:", error);
        // 使用占位图
        this.author.photo = `https://placehold.co/300x400?text=${encodeURIComponent(authorName)}`;
      }
    }
  }
};
</script>

<style scoped>
.author-section {
  margin-bottom: 30px;
}

.author-card-container {
  display: flex;
  justify-content: center;
  perspective: 1000px; /* 3D效果 */
  height: 350px;
}

.author-card {
  width: 260px;
  height: 350px;
  position: relative;
  transition: transform 0.8s;
  transform-style: preserve-3d;
  cursor: pointer;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.author-card.flipped {
  transform: rotateY(180deg);
}

.card-front, .card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
  box-sizing: border-box;
}

.card-front {
  background-color: #f0f9ff;
  border: 1px solid #d9ecff;
}

.card-back {
  background-color: #ffffff;
  border: 1px solid #ebeef5;
  transform: rotateY(180deg);
}

.card-question {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.question-icon {
  font-size: 48px;
  color: #409EFF;
  margin-bottom: 20px;
}

.question-text {
  font-size: 22px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 15px;
}

.tip-text {
  font-size: 14px;
  color: #909399;
  margin-top: 10px;
}

.author-photo-container {
  width: 160px;
  height: 200px;
  margin-bottom: 15px;
  border-radius: 5px;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.author-photo {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.author-name {
  font-size: 20px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 10px;
  text-align: center;
}

.author-brief {
  font-size: 14px;
  color: #606266;
  text-align: center;
  margin-bottom: 15px;
}

.card-tip {
  font-size: 13px;
  color: #909399;
  margin-top: 10px;
  text-align: center;
}

/* 模态框内容样式 */
.author-details {
  min-height: 400px;
}

.author-header {
  display: flex;
  margin-bottom: 20px;
}

.author-portrait {
  flex: 0 0 120px;
  margin-right: 20px;
}

.details-photo {
  width: 120px;
  height: 160px;
  border-radius: 4px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.author-summary {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.author-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 10px;
  color: #303133;
}

.author-subtitle {
  font-size: 16px;
  color: #606266;
  margin-bottom: 20px;
}

.author-actions {
  margin-top: auto;
  display: flex;
  gap: 10px;
}

.author-tabs {
  margin-top: 20px;
}

.tab-content {
  font-size: 15px;
  line-height: 1.6;
  color: #606266;
  text-align: justify;
}

.works-list, .awards-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.works-list li, .awards-list li {
  position: relative;
  padding-left: 20px;
  margin-bottom: 12px;
  font-size: 15px;
  color: #606266;
}

.works-list li::before, .awards-list li::before {
  content: '•';
  position: absolute;
  left: 8px;
  color: #409EFF;
}

/* 搜索表单样式 */
.search-form {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.genre-selector {
  display: flex;
  align-items: center;
  margin-top: 10px;
}

.genre-label {
  margin-right: 10px;
  font-size: 14px;
  color: #606266;
  white-space: nowrap;
}

.genre-select {
  flex: 1;
  margin-right: 10px;
}

.image-slot {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  background: #f5f7fa;
  color: #909399;
  font-size: 14px;
}

.dot {
  animation: dot 2s infinite steps(3, start);
  overflow: hidden;
}

@keyframes dot {
  0% { content: '.'; }
  33% { content: '..'; }
  66% { content: '...'; }
}

/* 针对较小屏幕的响应式设计 */
@media (max-width: 768px) {
  .author-card {
    width: 220px;
    height: 300px;
  }

  .author-photo-container {
    width: 140px;
    height: 180px;
  }

  .author-header {
    flex-direction: column;
    align-items: center;
  }

  .author-portrait {
    margin-right: 0;
    margin-bottom: 15px;
  }

  .author-summary {
    align-items: center;
    text-align: center;
  }

  .genre-selector {
    flex-direction: column;
    align-items: flex-start;
  }

  .genre-label {
    margin-bottom: 5px;
  }

  .genre-select {
    width: 100%;
    margin-bottom: 10px;
  }
}
</style>