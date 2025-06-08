<template>
  <div>
    <el-container class="page">
<!--      <el-header class="header">Books</el-header>-->
      <el-container class="container">
        <el-container class="main_content">
          <AuthorCard />
          <el-divider/>
          <!--books of the week-->
          <el-main class="books">
            <!-- 图书卡片1 -->
            <el-card id="book1" :class="{ 'card':true, 'flipped':isFlipped[0] }">
              <!-- 卡片头部 -->
              <div class="card-header">
                <div class="front">
                  <input
                      ref="input_ref"
                      v-model="book_names[0]"
                      placeholder="Book Name:/书名 ："
                      class="input"
                      aria-label="书名输入框"
                  />
                </div>
                <div class="back" v-if="isBackRendered[0]">
                  <img
                      v-if="userislion===true"
                      src="../assets/avatar_lion.png"
                      alt="狮子头像"
                      class="user-avatar"
                  />
                  <img
                      v-else-if="userisbear===true"
                      src="../assets/avatar_bear.png"
                      alt="熊头像"
                      class="user-avatar"
                  />
                  <p class="book-title">{{book_names[0]}}</p>
                </div>
              </div>

              <!-- 卡片内容 -->
              <el-main ref="body_ref" class="card_body" @click="flipCard(0)">
                <!-- 正面内容 -->
                <div class="front">
                  <el-divider class="divider">
                    <div class="icons-container">
                      <el-icon v-for="n in 10" :key="n" class="animal-icon">
                        <img
                            v-if="n%2===1"
                            src="../assets/svg/bear-svgrepo-com.svg"
                            alt="熊图标"
                            class="animal-svg"
                        />
                        <img
                            v-else
                            src="../assets/svg/lion-svgrepo-com.svg"
                            alt="狮子图标"
                            class="animal-svg"
                        />
                      </el-icon>
                    </div>
                  </el-divider>

                  <div class="block">
                    <span class="demonstration">Reading</span>
                    <div class="image-container">
                      <el-image class="placeholder_image" :src="src" alt="书籍封面">
                        <template #placeholder>
                          <div class="image-slot">
                            Loading<span class="dot">...</span>
                          </div>
                        </template>
                      </el-image>
                    </div>
                  </div>
                </div>

                <!-- 背面内容 -->
                <div class="back">
                  <p class="book-content">{{book_names[0]}}</p>
                </div>
              </el-main>
            </el-card>

            <!-- 图书卡片2 -->
            <el-card id="book2" :class="{ 'card':true, 'flipped':isFlipped[1] }">
              <!-- 卡片头部 -->
              <div class="card-header">
                <div class="front">
                  <input
                      v-model="book_names[1]"
                      placeholder="Book Name:/书名："
                      class="input"
                      aria-label="书名输入框"
                  />
                </div>
                <div class="back" v-if="isBackRendered[1]">
                  <img
                      v-if="userislion===true"
                      src="../assets/avatar_lion.png"
                      alt="狮子头像"
                      class="user-avatar"
                  />
                  <img
                      v-else-if="userisbear===true"
                      src="../assets/avatar_bear.png"
                      alt="熊头像"
                      class="user-avatar"
                  />
                  <p class="book-title">{{book_names[1]}}</p>
                </div>
              </div>

              <!-- 卡片内容 -->
              <el-main class="card_body" @click="flipCard(1)">
                <!-- 正面内容 -->
                <div class="front">
                  <el-divider class="divider">
                    <div class="icons-container">
                      <el-icon v-for="n in 10" :key="n" class="animal-icon">
                        <img
                            v-if="n%2===1"
                            src="../assets/svg/bear-svgrepo-com.svg"
                            alt="熊图标"
                            class="animal-svg"
                        />
                        <img
                            v-else
                            src="../assets/svg/lion-svgrepo-com.svg"
                            alt="狮子图标"
                            class="animal-svg"
                        />
                      </el-icon>
                    </div>
                  </el-divider>

                  <div class="block">
                    <span class="demonstration">Reading</span>
                    <div class="image-container">
                      <el-image class="placeholder_image" :src="src" alt="书籍封面">
                        <template #placeholder>
                          <div class="image-slot">
                            Loading<span class="dot">...</span>
                          </div>
                        </template>
                      </el-image>
                    </div>
                  </div>
                </div>

                <!-- 背面内容 -->
                <div class="back">
                  <p class="book-content">{{book_names[1]}}</p>
                </div>
              </el-main>
            </el-card>

            <!-- 图书卡片3 -->
            <el-card id="book3" :class="{ 'card':true, 'flipped':isFlipped[2] }">
              <!-- 卡片头部 -->
              <div class="card-header">
                <div class="front">
                  <input
                      v-model="book_names[2]"
                      placeholder="Book Name:/书名："
                      class="input"
                      aria-label="书名输入框"
                  />
                </div>
                <div class="back" v-if="isBackRendered[2]">
                  <img
                      v-if="userislion===true"
                      src="../assets/avatar_lion.png"
                      alt="狮子头像"
                      class="user-avatar"
                  />
                  <img
                      v-else-if="userisbear===true"
                      src="../assets/avatar_bear.png"
                      alt="熊头像"
                      class="user-avatar"
                  />
                  <p class="bookname_onback">{{book_names[2]}}</p>
                </div>
              </div>

              <!-- 卡片内容 -->
              <el-main class="card_body" @click="flipCard(2)">
                <!-- 正面内容 -->
                <div class="front">
                  <el-divider class="divider">
                    <div class="icons-container">
                      <el-icon v-for="n in 10" :key="n" class="animal-icon">
                        <img
                            v-if="n%2===1"
                            src="../assets/svg/bear-svgrepo-com.svg"
                            alt="熊图标"
                            class="animal-svg"
                        />
                        <img
                            v-else
                            src="../assets/svg/lion-svgrepo-com.svg"
                            alt="狮子图标"
                            class="animal-svg"
                        />
                      </el-icon>
                    </div>
                  </el-divider>

                  <div class="block">
                    <span class="demonstration">Reading</span>
                    <div class="image-container">
                      <el-image class="placeholder_image" :src="src" alt="书籍封面">
                        <template #placeholder>
                          <div class="image-slot">
                            Loading<span class="dot">...</span>
                          </div>
                        </template>
                      </el-image>
                    </div>
                  </div>
                </div>

                <!-- 背面内容 -->
                <div class="back">
                  <p class="signature_onback">user's signature</p>
                </div>
              </el-main>

              <!-- 卡片页脚 -->
              <el-footer class="card_footer">
<!--                <div class="card_footer_author">作者3</div>-->
              </el-footer>
            </el-card>

            <!--图书信息&书摘&想法输入/编辑弹窗-->
            <el-dialog ref="initial_ref" v-if="!isBookExist[this.currentFlippedCard]" v-model="dialogTableVisible" :title="book_names[this.currentFlippedCard]" width="80vh" @close="resetCard">
              <!--书摘输入区-->
              <el-input placeholder="请输入书摘" v-model="book_excerpts[this.currentFlippedCard]" type="textarea" autosize style="margin-bottom: 10px"/>
              <!--封面上传区-->
              <el-upload
                  class="upload-demo"
                  :limit="1"
                  accept=".jpg, .png, .webp"
                  :before-upload="beforeUpload"
                  :on-success="uploadSuccess"
                  drag
                  action="https://run.mocky.io/v3/9d059bf9-4660-45f2-925d-ce80ad6c4d15"
                  multiple
              >
                <el-icon class="el-icon--upload"><upload-filled /></el-icon>
                <div class="el-upload__text">
                  请上传图书封面
                  <br>
                  Drop file here or <em>click to upload</em>
                </div>
                <template #tip>
                  <div class="el-upload__tip">
                    创意无限，封面不限
                  </div>
                </template>
              </el-upload>
              <el-button-group class="ml-4">
                <el-tooltip
                    class="box-item"
                    effect="dark"
                    content="Save"
                    placement="top-start"
                >
                  <el-button type="primary" :icon="Select" @click="saveBookCard"/>
                </el-tooltip>
                <el-tooltip
                    class="box-item"
                    effect="dark"
                    content="Cancel"
                    placement="top-start"
                >
                  <el-button type="primary" :icon="CloseBold" @click="cancleEdit" />
                </el-tooltip>
              </el-button-group>
            </el-dialog>
            <el-dialog ref="initial_ref" v-if="isBookExist[this.currentFlippedCard]" v-model="dialogTableVisible" :title="book_names[this.currentFlippedCard]" class="excerpt_dialog" @close="resetCard" :lock-scroll="true">
              <!--书摘展示区-->
              <div class="demo-collapse">
                <el-collapse v-model="this.activeExcerpt" @change="handleChange">
                  <el-collapse-item name='1'>
                    <template #title>
                      {{this.book_chapters[0][0]}}<el-icon class="header-icon">
                      <info-filled />
                    </el-icon>
                    </template>
                    <div class="book_excerpt" @click="ideasVisible=true">
                      {{this.book_excerpts[0][0]}}
                    </div>
                    <el-input class="excerpt_add" placeholder="请输入书摘"></el-input>
                    <el-dialog
                        class="ideas_dialog"
                        v-model="ideasVisible"
                        title="Tips"
                        width="500"
                        :before-close="handleClose"
                        :lock-scroll="true"
                    >
                      <div class="ideas">
                        <div
                          v-for="[index,idea] in userIdeas_fromBack"
                          :key="index"
                          :content="idea"
                          class="idea.ideasBoxClass"
                        >
                          <img :src="idea.userId==='lion'?dialogLion:dialogBear" class="dialog_animal"/>
                          <div class="idea.triangleClass"></div>
                          <div class="ideasBody">
                            <span>{{idea.ideaContent}}</span>
                          </div>
                        </div>
                      </div>
                      <template #footer>
                        <div class="dialog-footer">
                          <el-button @click="ideasVisible = false">Cancel</el-button>
                          <el-button type="primary" @click="ideasVisible = false">
                            Confirm
                          </el-button>
                        </div>
                      </template>
                    </el-dialog>
                  </el-collapse-item>
                  <el-collapse-item title="Feedback" name="2">
                    <div>
                      Operation feedback: enable the users to clearly perceive their
                      operations by style updates and interactive effects;
                    </div>
                    <div>
                      Visual feedback: reflect current state by updating or rearranging
                      elements of the page.
                    </div>
                  </el-collapse-item>
                  <el-collapse-item title="Efficiency" name="3">
                    <div>
                      Simplify the process: keep operating process simple and intuitive;
                    </div>
                    <div>
                      Definite and clear: enunciate your intentions clearly so that the
                      users can quickly understand and make decisions;
                    </div>
                    <div>
                      Easy to identify: the interface should be straightforward, which helps
                      the users to identify and frees them from memorizing and recalling.
                    </div>
                  </el-collapse-item>
                  <el-collapse-item title="Controllability" name="4">
                    <div>
                      Decision making: giving advices about operations is acceptable, but do
                      not make decisions for the users;
                    </div>
                    <div>
                      Controlled consequences: users should be granted the freedom to
                      operate, including canceling, aborting or terminating current
                      operation.
                    </div>
                  </el-collapse-item>
                </el-collapse>
              </div>
              <el-button-group class="ml-4">
                <el-tooltip
                    class="box-item"
                    effect="dark"
                    content="Save"
                    placement="top-start"
                >
                  <el-button type="primary" :icon="Select" @click="saveBookCard"/>
                </el-tooltip>
                <el-tooltip
                    class="box-item"
                    effect="dark"
                    content="Cancel"
                    placement="top-start"
                >
                  <el-button type="primary" :icon="CloseBold" @click="cancleEdit" />
                </el-tooltip>
              </el-button-group>
            </el-dialog>
          </el-main>
        </el-container>
      </el-container>
      <el-footer class="footer">Footer
        <el-tooltip
            class="box-item"
            content="Back to Top"
            effect="light"
            placement="top-start"
            transition="fade-in-linear"
        >
          <el-backtop :right="100" :bottom="100" />
        </el-tooltip>
      </el-footer>
    </el-container>
  </div>

</template>

<style scoped>
/* 整体页面布局 */
.page {
  height: auto;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  width: 100%;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.header {
  display: flex;
  background-color: #B3C0D1;
  color: #333;
  justify-content: center;
  align-items: center;
  line-height: 10vh;
  min-height: 60px;
  width: 100%;
  box-sizing: border-box;
}

.container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  flex: 1;
  width: 100%;
  box-sizing: border-box;
}

.main_content {
  display: flex;
  flex-direction: column; /* 纵向排列 daily_author 和 books */
  width: 80%; /* 中间部分的宽度，确保占据剩余空间 */
  min-height: 100%;
  padding: 20px;
  box-sizing: border-box;
}

.footer {
  display: flex;
  background-color: #B3C0D1;
  color: #333;
  justify-content: center;
  align-items: center;
  min-height: 50px;
  width: 100%;
  position: relative;
  box-sizing: border-box;
  padding: 10px 0;
}

/* 图书卡片区域样式 */
/* 卡片容器布局 */
.books {
  display: flex;
  justify-content: space-evenly; /* 使用space-evenly确保间距均匀 */
  align-items: stretch; /* 确保所有卡片高度一致 */
  flex-wrap: wrap; /* 允许换行 */
  min-height: 30vh; /* 最小高度 */
  padding: 2vh 1vw; /* 内边距 */
  gap: 2vw; /* 卡片间距 */
  width: 100%; /* 容器宽度 */
  box-sizing: border-box; /* 确保padding不会影响宽度 */
}

/* 卡片整体布局：从上到下依次排列 */
.card {
  display: flex;
  flex-direction: column;
  position: relative;
  width: 31%;
  min-width: 240px;
  max-width: 400px;
  height: auto;
  min-height: 400px;
  box-sizing: border-box;
  perspective: 1000px;
}

/* 卡片头部（包含输入框）*/
.card-header {
  width: 100%;
  position: relative;
  z-index: 2;
  margin-top: -10px;
  margin-bottom: 15px;
  order: 1; /* 确保头部在最上方 */
}

/* 卡片主体部分，包含分割线和内容 */
.card_body {
  display: flex;
  flex-direction: column;
  padding: 0;
  flex: 1;
  order: 2; /* 中间位置 */
}

.card:hover {
  cursor: pointer;
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
}

/* 卡片头部输入框 */
.input {
  width: 100%; /* 宽度100%适应卡片 */
  height: 30px; /* 固定高度 */
  font-size: 16px; /* 固定字体大小 */
  font-family: "Brush Script MT", 楷体, cursive;
  padding: 5px 10px; /* 内边距 */
  box-sizing: border-box; /* 确保padding不会影响宽度 */
  border: 1px solid #dcdfe6; /* 边框 */
  border-radius: 4px; /* 圆角边框 */
}

.input:focus {
  border-color: #409EFF;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
  outline: none;
}

/* 用户头像样式 */
.user-avatar {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  object-fit: cover;
  margin-right: 10px;
  display: inline-block;
  vertical-align: middle;
}

/* 书名标题样式 */
.book-title {
  display: inline-block;
  vertical-align: middle;
  font-size: calc(1rem + 0.2vw);
  font-weight: bold;
  margin: 0;
  max-width: calc(100% - 50px); /* 留出头像空间 */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 分隔线 */
/* 不要给 el-divider 全局套上 space-between */
/* 分割线位置修正 */
.divider {
  margin: 60px 0 5px 0; /* 上边距为0，下边距保持 */
  width: 100%;
  position: relative;
  z-index: 2;
}

/* 图标容器样式 */
.icons-container {
  display: flex;
  justify-content: center;
  width: 100%;
  flex-wrap: nowrap;
  padding: 5px 0;
}

/* 动物图标样式 */
.animal-icon {
  padding: 0 10px;
}

.animal-svg {
  width: 2vh;
  min-width: 16px;
  max-width: 24px;
  height: auto;
}

/* 主要内容区块 */
.block {
  margin: 20px; /* 移除上边距 */
  padding: 10px 0;
  text-align: center;
  border: solid 1px var(--el-border-color);
  width: 100%;
  box-sizing: border-box;
  order: 3; /* 在分割线之后 */
}

/* 标题文本 */
.demonstration {
  display: block;
  color: var(--el-text-color-secondary);
  font-size: calc(1rem + 0.2vw); /* 响应式字体大小 */
  margin-bottom: 1vh;
  white-space: nowrap; /* 防止文本换行 */
  overflow: hidden; /* 隐藏溢出内容 */
  text-overflow: ellipsis; /* 文本溢出显示省略号 */
}

/* 图片容器 */
.image-container {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  margin-top: 10px;
}

/* 图片样式 */
.placeholder_image {
  /* padding: 0 2vh; */ /* Remove this */
  max-width: 100%;
  max-height: 30vh;
  width: auto;
  height: auto;
  object-fit: contain;
}

/* 加载占位符 */
.image-slot {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  min-height: 150px; /* 设置最小高度 */
  background: var(--el-fill-color-light);
  color: var(--el-text-color-secondary);
  font-size: calc(0.8rem + 0.2vw); /* 响应式字体大小 */
}

/* 加载动画 */
.dot {
  animation: dot 2s infinite steps(3, start);
  overflow: hidden;
}

@keyframes dot {
  0% { content: '.'; }
  33% { content: '..'; }
  66% { content: '...'; }
}

/* 卡片正面和背面的共同样式 */
.card .front, .card .back {
  display: flex; /* 改用flex布局 */
  flex-direction: column; /* 垂直排列内容 */
  align-items: center; /* 水平居中 */
  justify-content: flex-start; /* 从上向下排列 */
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  backface-visibility: hidden;
  transition: transform 0.6s ease-in-out;
  padding: 10px;
  box-sizing: border-box;
}

/* 书籍内容样式 */
.book-content {
  font-size: calc(1rem + 0.2vw);
  text-align: center;
  padding: 10px;
  margin: 0;
  word-break: break-word; /* 允许长单词换行 */
}

/* 背面书名样式 */
.bookname_onback {
  font-size: calc(1.2rem + 0.3vw); /* 响应式字体大小 */
  font-family: "Brush Script MT", 楷体, cursive;
  text-align: center; /* 居中对齐 */
  padding: 10px; /* 添加内边距 */
  word-break: break-word; /* 允许长单词换行 */
}

/* 背面签名样式 */
.signature_onback {
  font-size: calc(0.8rem + 0.2vw); /* 响应式字体大小 */
  font-family: "Brush Script MT", 楷体, cursive;
  text-align: center; /* 居中对齐 */
}

/* 卡片页脚 */
.card_footer {
  position: relative;
  width: 100%;
  display: flex;
  justify-content: flex-end;
  margin-top: 10px;
  order: 3; /* 确保页脚在最下方 */
}

/* 页脚作者 */
.card_footer_author {
  background-color: #3967FF;
  width: auto; /* 宽度自适应内容 */
  min-width: 40px; /* 最小宽度 */
  max-width: 150px; /* 最大宽度 */
  height: auto; /* 高度自适应 */
  padding: 5px 10px; /* 添加内边距 */
  font-size: calc(1rem + 0.2vw); /* 响应式字体大小 */
  color: white; /* 文字颜色 */
  border-radius: 4px; /* 圆角边框 */
  text-align: center; /* 文本居中 */
  overflow: hidden; /* 隐藏溢出内容 */
  text-overflow: ellipsis; /* 文本溢出显示省略号 */
  white-space: nowrap; /* 防止文本换行 */
}

/* 响应式调整 */
@media (max-width: 768px) {
  .divider {
    margin: 0 0 10px 0;
  }

  .icons-container {
    padding: 3px 0;
  }

  .animal-icon {
    padding: 0 3px;
  }

  .block {
    padding: 5px 0;
  }

  .image-container {
    margin-top: 5px;
  }
}
/* 按钮组样式 */
.button-group {
  display: flex;
  justify-content: flex-end;
  margin-top: 15px;
  gap: 10px;
}

/* 确保按钮组响应式 */
.ml-4 {
  margin-top: 15px;
  display: flex;
  justify-content: flex-end;
  flex-wrap: wrap;
  gap: 10px;
}

/* 对话框样式 */
:deep(.el-dialog) {
  width: auto !important; /* 覆盖原来的固定宽度 */
  max-width: 90vw; /* 最大宽度为视口宽度的90% */
  margin: 0 auto;
  max-height: 90vh; /* 最大高度为视口高度的90% */
  display: flex;
  flex-direction: column;
}

:deep(.el-dialog__body) {
  width: auto !important; /* 覆盖原来的固定宽度 */
  max-width: 100%; /* 确保不超出对话框 */
  height: auto !important; /* 高度自适应 */
  max-height: calc(90vh - 120px); /* 减去头部和底部的高度 */
  overflow: auto; /* 内容超出时显示滚动条 */
  padding: 15px !important; /* 统一内边距 */
}

/* 确保上传组件响应式 */
.upload-demo {
  width: 100%;
  max-width: 100%;
  box-sizing: border-box;
}

:deep(.el-upload) {
  width: 100%;
  max-width: 100%;
}

:deep(.el-upload-dragger) {
  width: 100% !important;
  max-width: 100%;
}

/* 针对折叠面板的响应式调整 */
.demo-collapse {
  width: 100% !important; /* 宽度100%适应对话框 */
  max-width: 100%;
  padding: 10px;
  box-sizing: border-box;
}

:deep(.el-collapse-item__content) {
  display: grid;
  grid-template-columns: 1fr; /* 单列布局 */
  gap: 10px;
}

:deep(.el-collapse-item__header) {
  font-size: 18px; /* 设置你想要的字号 */
}

/* 针对输入框的响应式调整 */
:deep(.el-input__wrapper) {
  width: 100% !important; /* 宽度100%适应容器 */
  max-width: 100%;
  padding: 5px !important;
  box-sizing: border-box;
}

/* 书摘样式的响应式调整 */
.book_excerpt {
  text-indent: 2em; /* 首行缩进两字符 */
  background-color: v-bind(ideaBox_color);
  font-size: calc(1rem + 0.2vw); /* 响应式字体大小 */
  height: fit-content;
  width: auto !important; /* 宽度自适应 */
  max-width: 100%; /* 不超出容器 */
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 5px;
  border: #ffecec solid 2px;
  cursor: pointer;
  font-family: 华文仿宋, "Times New Roman", cursive;
  font-weight: bold;
  word-break: break-word; /* 允许长单词换行 */
}

/* 书摘输入框的响应式调整 */
.excerpt_add {
  justify-self: center;
  width: 100% !important; /* 宽度100%适应容器 */
  max-width: 100%;
  height: auto;
  min-height: 100px;
  background-color: v-bind(ideaBox_color);
  border: #8c939d solid 3px;
  border-radius: 5px;
  font-size: calc(0.9rem + 0.1vw); /* 响应式字体大小 */
  box-sizing: border-box;
}

/* 想法对话框的响应式调整 */
.ideas {
  position: relative !important; /* 改为相对定位 */
  display: grid;
  grid-template-columns: 1fr; /* 单列布局 */
  width: 100% !important; /* 宽度100%适应对话框 */
  max-width: 100%;
  margin: 0 !important;
  box-sizing: border-box;
}

/* 响应式调整动物图标动画 */
.dialog_animal {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 25px;
  height: 25px;
  max-width: 100%;
  max-height: 100%;
  animation: wavingAnimal 3s linear infinite;
}

@keyframes wavingAnimal {
  0% { transform: rotateZ(-20deg); }
  25% { transform: rotateZ(0); }
  50% { transform: rotateZ(20deg); }
  75% { transform: rotateZ(0); }
  100% { transform: rotateZ(-20deg); }
}

/* 想法框的响应式调整 */
.ideasBox_lion, .ideasBox_bear {
  position: relative; /* 相对定位，便于三角形定位 */
  background-color: v-bind(ideaBox_color); /* 对话框背景色 */
  border: 1px solid #ccc; /* 边框 */
  border-radius: 5px; /* 圆角 */
  padding: 10px 15px 20px 10px;
  width: fit-content;
  max-width: 100%; /* 不超出容器 */
  box-sizing: border-box;
  word-break: break-word; /* 允许长单词换行 */
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1); /* 阴影效果 */
}

.ideasBox_bear {
  justify-self: end;
}

/* 想法内容的响应式调整 */
.ideasBody {
  background-color: v-bind(ideaBox_color);
  width: auto !important; /* 宽度自适应 */
  max-width: 100%; /* 不超出容器 */
  height: fit-content;
  padding: 5px;
  border-radius: 3px;
  font-family: "Brush Script MT", 楷体, cursive;
  font-size: calc(1rem + 0.2vw); /* 响应式字体大小 */
  color: black;
  word-break: break-word; /* 允许长单词换行 */
}

/* 对话框页脚样式 */
.dialog-footer {
  align-self: end;
  justify-self: end;
  display: flex;
  gap: 10px;
  margin-top: 15px;
}

/* 处理卡片翻转效果 */
.card .front {
  transform: rotateY(0deg); /* 正面初始显示 */
}

.card .back {
  transform: rotateY(180deg); /* 背面初始隐藏 */
}

.card.flipped .front {
  transform: rotateY(180deg); /* 正面翻转 */
}

.card.flipped .back {
  transform: rotateY(0deg); /* 背面翻转 */
}

/* 适配返回顶部按钮 */
:deep(.el-backtop) {
  right: 20px !important;
  bottom: 20px !important;
}

/* 创建加载动画 */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 为每张卡片设置不同的延迟，创造级联效果 */
#book1 {
  animation-delay: 0.1s;
}

#book2 {
  animation-delay: 0.2s;
}

#book3 {
  animation-delay: 0.3s;
}

/* 只为中文字符定义楷体 */
@font-face {
  font-family: 楷体;
  src: local(楷体); /* 假设系统有楷体字体 */
  unicode-range: U+4E00-9FFF; /* 中文字符的 Unicode 范围 */
}

/* 只为英文字符定义花体 */
@font-face {
  font-family: 'Brush Script MT';
  src: local('Brush Script MT'); /* 系统中的花体字体 */
  unicode-range: U+0020-007F; /* 基本拉丁字符的 Unicode 范围，包括空格和标点符号 */
}

/* 响应式媒体查询 */
/* 大屏幕 */
@media (min-width: 1200px) {
  .card {
    width: calc(33.33% - 2vw);
  }

  .main_content {
    width: 80%;
  }
}

/* 中等屏幕 */
@media (max-width: 1199px) and (min-width: 769px) {
  .card {
    width: calc(50% - 2vw);
  }

  .books {
    justify-content: space-around;
  }

  .main_content {
    width: 90%;
  }

  .card-header {
    flex-direction: column;
    align-items: center;
  }

  .user-avatar {
    margin-right: 0;
    margin-bottom: 5px;
  }

  .book-title {
    max-width: 100%;
    text-align: center;
  }
}

/* 小屏幕 */
@media (max-width: 768px) {
  .card {
    width: 100%; /* 小屏幕每行一张卡片 */
    max-width: none; /* 取消最大宽度限制 */
    min-height: 350px; /* 减小最小高度 */
  }

  .books {
    padding: 1vh 0; /* 减小内边距 */
  }

  .placeholder_image {
    max-height: 25vh; /* 减小图片最大高度 */
  }

  .card_footer_author {
    max-width: 100px; /* 减小最大宽度 */
  }

  .main_content {
    width: 95%;
    padding: 10px;
  }

  .animal-icon {
    padding: 0 1vh; /* 减小内边距 */
  }

  .icons-container {
    padding: 0 2px;
  }

  :deep(.el-backtop) {
    right: 10px !important;
    bottom: 10px !important;
  }

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

  .demo-collapse {
    padding: 5px;
  }
}

/* 超小屏幕 */
@media (max-width: 480px) {
  .card {
    min-height: 300px; /* 进一步减小最小高度 */
  }

  .input {
    font-size: 0.9rem; /* 减小字体大小 */
  }

  .demonstration {
    font-size: 0.9rem; /* 减小字体大小 */
  }

  .placeholder_image {
    padding: 0 1vh; /* 减小内边距 */
    max-height: 20vh; /* 进一步减小图片最大高度 */
  }

  .image-slot {
    min-height: 100px; /* 减小最小高度 */
  }

  .card_footer_author {
    font-size: 0.9rem; /* 减小字体大小 */
  }

  .ideasBody {
    font-size: 0.9rem; /* 减小字体大小 */
    padding: 3px; /* 减小内边距 */
  }

  .dialog_animal {
    width: 20px; /* 减小尺寸 */
    height: 20px;
  }

  :deep(.el-dialog) {
    max-width: 95vw; /* 增加对话框在小屏幕上的宽度 */
  }

  :deep(.el-dialog__body) {
    padding: 10px !important; /* 减小内边距 */
  }

  .book_excerpt {
    font-size: 0.9rem; /* 减小字体大小 */
  }

  .main_content {
    width: 100%;
    padding: 5px;
  }
}

/* 针对Safari浏览器的特殊处理 */
@supports (-webkit-touch-callout: none) {
  .card .front, .card .back {
    -webkit-backface-visibility: hidden;
    backface-visibility: hidden;
  }

  /* Safari对flex布局的特殊处理 */
  .books {
    display: -webkit-box;
    display: -webkit-flex;
    display: flex;
    -webkit-flex-wrap: wrap;
    flex-wrap: wrap;
  }
}

/* 适配高DPI屏幕 */
@media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
  .input, .demonstration, .book-title, .book-content, .bookname_onback, .signature_onback {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
}

/* 处理打印样式 */
@media print {
  .card {
    box-shadow: none !important;
    border: 1px solid #ddd;
    break-inside: avoid;
  }

  .books {
    display: block;
  }

  .el-dialog, .el-backtop, .footer {
    display: none !important;
  }
}

/* 处理夜间模式 */
@media (prefers-color-scheme: dark) {
  .card-front {
    background-color: #283142;
    border-color: #3b4558;
  }

  .card-back {
    background-color: #1e2736;
    border-color: #3b4558;
  }

  .input {
    background-color: #283142;
    color: #e0e0e0;
    border-color: #3b4558;
  }

  .demonstration, .question-text, .author-name, .book-title {
    color: #e0e0e0;
  }

  .tip-text, .author-brief, .card-tip {
    color: #a0a0a0;
  }

  .block {
    border-color: #3b4558;
    background-color: #283142;
  }

  .image-slot {
    background-color: #1e2736;
    color: #a0a0a0;
  }

  .book_excerpt {
    border-color: #3b4558;
  }

  :deep(.el-dialog) {
    background-color: #283142;
    color: #e0e0e0;
  }

  :deep(.el-dialog__title) {
    color: #e0e0e0;
  }

  :deep(.el-dialog__body) {
    color: #e0e0e0;
  }

  :deep(.el-collapse-item__header) {
    background-color: #283142;
    color: #e0e0e0;
    border-color: #3b4558;
  }

  :deep(.el-collapse-item__content) {
    background-color: #1e2736;
    color: #e0e0e0;
  }

  :deep(.el-tabs__item) {
    color: #a0a0a0;
  }

  :deep(.el-tabs__item.is-active) {
    color: #409EFF;
  }

  :deep(.el-tabs__nav-wrap::after) {
    background-color: #3b4558;
  }

  :deep(.el-upload-dragger) {
    background-color: #1e2736;
    border-color: #3b4558;
  }

  :deep(.el-upload__text) {
    color: #a0a0a0;
  }

  :deep(.el-button--default) {
    background-color: #283142;
    border-color: #3b4558;
    color: #e0e0e0;
  }
}

/* 无障碍优化 */
@media (prefers-reduced-motion: reduce) {
  .card, .author-card {
    transition: none;
  }

  .card:hover {
    transform: none;
  }

  .card .front, .card .back {
    transition: none;
  }

  .dialog_animal {
    animation: none;
  }

  .books, .card {
    animation: none;
  }
}

/* 键盘焦点样式 */
.input:focus-visible,
:deep(.el-button:focus-visible),
:deep(.el-collapse-item__header:focus-visible) {
  outline: 2px solid #409EFF;
  outline-offset: 2px;
}

/* 响应式字体大小 */
html {
  font-size: 16px;
}

@media (max-width: 768px) {
  html {
    font-size: 14px;
  }
}

@media (max-width: 480px) {
  html {
    font-size: 12px;
  }
}

/* 添加平滑滚动 */
html {
  scroll-behavior: smooth;
}

/* 确保全局盒模型一致 */
* {
  box-sizing: border-box;
}

/* 添加渐变背景效果 */
.page {
  background: linear-gradient(135deg, #f5f7fa 0%, #e4ecfb 100%);
}

/* 添加卡片阴影效果 */
.card, .author-card {
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

/* 添加边缘变亮效果 */
.card:before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 5px;
  background: linear-gradient(90deg, #3967FF, #409EFF);
  border-radius: 5px 5px 0 0;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.card:hover:before {
  opacity: 1;
}

/* 添加微妙的动画效果 */
@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

.card_footer_author:hover {
  animation: pulse 1s infinite;
}

/* 修复iOS上的一些问题 */
button,
input,
select,
textarea {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  border-radius: 0;
}

input[type="search"]::-webkit-search-decoration,
input[type="search"]::-webkit-search-cancel-button,
input[type="search"]::-webkit-search-results-button,
input[type="search"]::-webkit-search-results-decoration {
  -webkit-appearance: none;
}

/* 确保所有图片响应式 */
img {
  max-width: 100%;
  height: auto;
}

/* 添加焦点状态 */
*:focus {
  outline: none;
}

*:focus-visible {
  outline: 2px solid #409EFF;
  outline-offset: 2px;
}

/* 可见的跳过导航链接（辅助功能） */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #409EFF;
  color: white;
  padding: 8px;
  z-index: 100;
  transition: top 0.3s;
}

.skip-link:focus {
  top: 0;
}

/* 添加触觉反馈样式（用于支持触觉反馈的设备） */
@media (hover: hover) {
  .card:active {
    transform: scale(0.98);
  }
}
</style>

<script>
import {Select, CloseBold} from "@element-plus/icons-vue";
import {ElMessage, ElMessageBox} from "element-plus";
import {mapGetters} from "vuex";
import dialogLion from '../assets/dialog_lion.png';
import dialogBear from '../assets/dialog_bear.png'
import AuthorCard from './AuthorCard.vue';


export default{
  name: 'BooksPage',
  components: {
    AuthorCard
  },
  computed: {
    Select() {
      return Select
    },
    CloseBold() {
      return CloseBold
    },
    ...mapGetters(['getUserId']),
    user_id(){
      return this.getUserId;
    },
  },
 // 组件名称

  data(){
    return{
      src: 'https://cube.elemecdn.com/6/94/4d3ea53c084bad6931a56d5158a48jpeg.jpeg',
      isFlipped: [false,false,false],
      isBackRendered: [false,false,false], // 控制反面内容的渲染
      userislion: false, // 用户是廖博文
      userisbear: true, // 用户是张艺
      userislion_src: "../assets/avatar_lion.png", // 廖博文头像
      userisbear_src: "../assets/avatar_bear.png", // 张艺头像
      user_name: 'bowen',
      dialogTableVisible: false,
      currentFlippedCard: null,
      book_names: ["","",""],
      flag: 0,
      ideasVisible: false,
      book_ideas: [[[['想法1']]], [[[]]], [[[]]]], // 书-章节-书摘-想法
      book_chapters: [['第一章',''], [''], ['']],
      book_excerpts: [['那天我又独自坐在屋里，看着窗外的树叶“唰唰啦啦”地飘落。母亲进来了，挡在窗前：“北海的菊花开了，我推着你去看看吧。”她憔悴的脸上现出央求般的神色。“什么时候？”“你要是愿意，就明天?”她说。我的回答已经让她喜出望外了。“好吧，就明天。”我说。她高兴得一会坐下，一会站起：“那就赶紧准备准备。”“哎呀，烦不烦？几步路，有什么好准备的！”她也笑了，坐在我身边，絮絮叨叨地说着：“看完菊花，咱们就去‘仿膳’，你小时候最爱吃那儿的豌豆黄儿。还记得那回我带你去北海吗？你偏说那杨树花是毛毛虫，跑着，一脚踩扁一个……”她忽然不说了。对于“跑”和“踩”一类的字眼，她比我还敏感。她又悄悄地出去了。'], [''], ['']],
      isSaved: false, // 后端是否保存成功
      isBookExist: [true, false, false], // 图书卡片中是否以有图书
      activeExcerpt: '1',
      excerptFlag: '1',
      ideaBox_color: '#98FB98',
      dialog_animal: '', // idea对话框显示的animal：lion/bear
      userIdeas_input: [ // 用于将用户输入数据打包发送到后端，存入数据库
        {
          userId: '',
          userName: '',
          ideaContent: "",
          excerpt: [[['']]], // 三维数组，存储书摘内容，第一维以bookNum为索引，第二维以chapterNum为索引
          chapterNum: null,
          chapter: [['']], // 二维数组，存储章节名称，第一维以bookNum为索引
          bookNum: null,
          bookName: ''
        }
      ],
      userIdeas_fromBack: [ // 用于接收从后端获取的数据，进行渲染
        {
          userId: '',
          userName: '',
          ideaContent: '',
          excerpt: '',
          chapter: [[]],
          bookNum: null,
          bookName: '',
        }
      ],
      dialogLion,
      dialogBear,
      // 作者功能相关数据
      author_name: "",
      author_profile: "",
      author_photo: "",
      author_loading: false,
      authorSearchQuery: "",
      selectedGenre: "文学", // 默认作者类型
      genres: ["文学", "科幻", "奇幻", "悬疑", "历史", "哲学", "诗歌", "散文"],
      // 一些默认作者用于随机展示
      defaultAuthors: [
        {
          name: "鲁迅",
          profile: "中国现代文学家、思想家、革命家。原名周树人，字豫才。代表作有《狂人日记》、《阿Q正传》等。",
          photo: "https://placehold.co/150x200?text=鲁迅"
        },
        {
          name: "莫言",
          profile: "中国当代作家，2012年诺贝尔文学奖获得者。代表作有《红高粱》、《丰乳肥臀》等。",
          photo: "https://placehold.co/150x200?text=莫言"
        },
        {
          name: "余华",
          profile: "中国当代作家，代表作有《活着》、《许三观卖血记》等。",
          photo: "https://placehold.co/150x200?text=余华"
        }
      ],
      // 响应式布局相关属性
      cardsPerRow: 3, // 默认每行3张卡片
      resizeTimer: null, // 用于窗口大小变化的节流处理
      dialogWidth: '70%', // 默认对话框宽度
      isMobile: false, // 是否为移动设备
      isTablet: false, // 是否为平板设备
      touchEnabled: 'ontouchstart' in window, // 检测是否支持触摸
      observerSupported: typeof IntersectionObserver !== 'undefined', // 检测是否支持IntersectionObserver
      browserInfo: {
        isSafari: /^((?!chrome|android).)*safari/i.test(navigator.userAgent),
        isIOS: /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream,
        isFirefox: navigator.userAgent.toLowerCase().indexOf('firefox') > -1,
        isEdge: navigator.userAgent.indexOf('Edge') > -1
      },
    }
  },

  mounted() {
    // 选择ideaBox背景颜色、对话框动物
    if (this.user_id==='lion'){
      console.log(this.user_id);
      this.ideaBox_color = '#87CEFA';
      this.dialog_animal = dialogLion;
    }
    else if (this.user_id==='bear'){
      this.ideaBox_color = '#98FB98';
      this.dialog_animal = dialogBear;
    }
    // 初始化响应式功能
    this.initResponsiveFeatures();

    // 添加窗口大小变化监听
    window.addEventListener('resize', this.handleResize);

    // 检测当前屏幕尺寸
    this.checkScreenSize();

    // 如果支持触摸，初始化触摸事件
    if (this.touchEnabled) {
      this.$nextTick(() => {
        this.initTouchEvents();
      });
    }

    // 如果支持IntersectionObserver，优化图片加载
    if (this.observerSupported) {
      this.$nextTick(() => {
        this.optimizeImages();
      });
    }

    // 特殊浏览器的兼容性处理
    this.handleBrowserCompatibility();
  },

  // 更新后执行
  updated() {
    // 调整卡片高度
    this.$nextTick(() => {
      this.adjustCardHeights();
    });
  },

// 组件销毁前执行
  beforeUnmount() {
    // 移除窗口大小变化监听
    window.removeEventListener('resize', this.handleResize);

    // 清除计时器
    clearTimeout(this.resizeTimer);
  },

  methods: {// 初始化响应式功能
    initResponsiveFeatures() {
      // 检测设备类型
      const width = window.innerWidth;
      this.isMobile = width < 768;
      this.isTablet = width >= 768 && width < 1200;

      // 根据设备类型调整对话框宽度
      if (this.isMobile) {
        this.dialogWidth = '95%';
      } else if (this.isTablet) {
        this.dialogWidth = '80%';
      } else {
        this.dialogWidth = '70%';
      }

      // 移动设备特殊处理
      if (this.isMobile || this.isTablet) {
        // 简化动画以提高性能
        document.documentElement.style.setProperty('--transition-duration', '0.3s');
      }
    },

    // 处理浏览器兼容性
    handleBrowserCompatibility() {
      // Safari特殊处理
      if (this.browserInfo.isSafari) {
        // 解决Safari中3D翻转的问题
        document.querySelectorAll('.card').forEach(card => {
          card.style.webkitTransformStyle = 'preserve-3d';
        });
      }

      // iOS特殊处理
      if (this.browserInfo.isIOS) {
        // 解决iOS中滚动问题
        document.querySelectorAll('.card_body').forEach(body => {
          body.style.webkitOverflowScrolling = 'touch';
        });
      }

      // Firefox特殊处理
      if (this.browserInfo.isFirefox) {
        // 解决Firefox中某些flex布局问题
        document.querySelectorAll('.books').forEach(container => {
          container.style.display = '-moz-box';
          container.style.display = 'flex';
        });
      }
    },
    async fetchUserIdeas() {
      const response = [
        {userId:'lion', userName:'liaobowen', ideaContent:'', excerpt:'', chapter:'', book:''},
        {userId:'bear', userName:'zhangyi'}
      ];
      this.userIdeas_fromBack = response.map(userIdea=>({
        ...userIdea, // javascript展开运算符：用于将userIdea的所有属性复制到新的对象中（此处为userIdeas_fromBack）
        triangleClass: userIdea.userId === 'lion' ? 'triangle_lion' : 'triangle_bear',
        ideaBoxClass: userIdea.userId === 'lion' ? 'ideaBox_lion' : 'ideaBox_bear',
      }))
    },

// 检测屏幕尺寸并调整卡片布局
    checkScreenSize() {
      // 获取当前窗口宽度
      const width = window.innerWidth;

      // 根据窗口宽度设置每行卡片数量
      if (width < 768) {
        this.cardsPerRow = 1; // 小屏幕每行1张卡片
      } else if (width < 1200) {
        this.cardsPerRow = 2; // 中等屏幕每行2张卡片
      } else {
        this.cardsPerRow = 3; // 大屏幕每行3张卡片
      }

      // 调整卡片高度，确保一致性
      this.$nextTick(() => {
        this.adjustCardHeights();
      });
    },

// 调整所有卡片的高度使其一致
    adjustCardHeights() {
      // 获取所有卡片元素
      const cards = document.querySelectorAll('.card');
      if (!cards.length) return;

      // 重置所有卡片高度
      cards.forEach(card => {
        card.style.height = 'auto';
      });

      // 按行分组卡片
      const rows = [];
      for (let i = 0; i < cards.length; i += this.cardsPerRow) {
        rows.push(Array.from(cards).slice(i, i + this.cardsPerRow));
      }

      // 对每行的卡片设置相同的高度
      rows.forEach(row => {
        // 找出该行中最高的卡片
        let maxHeight = 0;
        row.forEach(card => {
          maxHeight = Math.max(maxHeight, card.offsetHeight);
        });

        // 将该行所有卡片设置为相同高度
        row.forEach(card => {
          card.style.height = `${maxHeight}px`;
        });
      });
    },

// 处理窗口大小变化
    handleResize() {
      // 添加节流处理，避免频繁调用
      clearTimeout(this.resizeTimer);
      this.resizeTimer = setTimeout(() => {
        this.checkScreenSize();
      }, 200);
    },

// 增强版翻转卡片函数
    flipCard(index) {
      // 先翻转卡片
      this.isFlipped[index] = !this.isFlipped[index];

      // 在移动设备上，滚动到卡片位置确保可见
      if (window.innerWidth < 768) {
        const card = document.getElementById(`book${index + 1}`);
        if (card) {
          // 使用平滑滚动
          card.scrollIntoView({ behavior: 'smooth', block: 'center' });
        }
      }

      // 使用 setTimeout 进行延迟，让背面内容渲染出来
      setTimeout(() => {
        this.isBackRendered[index] = true;
      }, 250); // 250ms 延迟时间，实现翻转与背面显示无缝衔接的效果

      this.dialogTableVisible = true;
      this.currentFlippedCard = index;

      // 阻止事件冒泡
      event.stopPropagation();
    },

// 改进的卡片重置函数
    resetCard() {
      // 记录当前滚动位置
      const scrollPosition = window.scrollY;

      // 当关闭弹出框时，重置 isFlipped 为 false
      this.isFlipped.forEach((value, index) => {
        this.isFlipped[index] = false; // 重置每个卡片的翻转状态为 false
        setTimeout(() => {
          this.isBackRendered[index] = false;
        }, 300); // 300ms 延迟时间，实现翻转与背面显示无缝衔接的效果
      });

      // 重置滚动位置，防止页面跳动
      window.scrollTo({
        top: scrollPosition,
        behavior: 'auto'
      });

      // 调整卡片高度
      this.$nextTick(() => {
        this.adjustCardHeights();
      });
    },

// 优化的对话框处理函数
    handleDialogOpen() {
      // 根据屏幕大小调整对话框宽度
      const width = window.innerWidth;
      if (width < 768) {
        this.dialogWidth = '95%';
      } else if (width < 1200) {
        this.dialogWidth = '80%';
      } else {
        this.dialogWidth = '70%';
      }

      // 锁定背景滚动
      document.body.style.overflow = 'hidden';
    },

// 对话框关闭处理
    handleDialogClose() {
      // 解锁背景滚动
      document.body.style.overflow = 'auto';

      // 调用卡片重置
      this.resetCard();
    },

// 处理触摸设备的手势
    initTouchEvents() {
      // 获取所有卡片元素
      const cards = document.querySelectorAll('.card');

      cards.forEach((card, index) => {
        let touchStartX = 0;
        let touchEndX = 0;

        // 触摸开始
        card.addEventListener('touchstart', (e) => {
          touchStartX = e.changedTouches[0].screenX;
        }, { passive: true });

        // 触摸结束
        card.addEventListener('touchend', (e) => {
          touchEndX = e.changedTouches[0].screenX;

          // 检测左右滑动
          const swipeThreshold = 50; // 滑动阈值
          const swipeDistance = touchEndX - touchStartX;

          // 向左滑动翻转卡片
          if (swipeDistance < -swipeThreshold && !this.isFlipped[index]) {
            this.flipCard(index);
          }

          // 向右滑动将卡片翻回
          if (swipeDistance > swipeThreshold && this.isFlipped[index]) {
            this.flipCard(index);
          }
        }, { passive: true });
      });
    },

// 优化图片加载
    optimizeImages() {
      // 懒加载图片
      const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            const img = entry.target;
            if (img.dataset.src) {
              img.src = img.dataset.src;
              img.removeAttribute('data-src');
            }
            observer.unobserve(img);
          }
        });
      });

      // 监视所有图片
      document.querySelectorAll('.placeholder_image').forEach(img => {
        observer.observe(img);
      });
    },

    changeIcon() {

    },
    beforeUpload() {

    },
    uploadSuccess() {

    },
    saveBookCard(currentFlippedCard, bookName) { // 保存图书卡片内容
      if (currentFlippedCard!=null&&bookName!=='') {
        console.log("ready to upload");

        this.flag++;

        if (this.isSaved) {
          ElMessage({
            message: 'Save Success!😊',
            type: 'success',
          });
          this.isBookExist[currentFlippedCard] = true;
        }
        else {
          ElMessage({
            message: 'Save Failure!😭',
            type: 'error',
          })
        }
      }
    },
    cancleEdit() { // 取消图书卡片的编辑
      ElMessageBox.confirm(
          '取消编辑将清除当前输入的所有内容，确认取消吗？',
          '三思而后行',
          {
            confirmButtonText: 'OK',
            cancelButtonText: 'Cancel',
            type: 'warning',
          }
      )
          .then(() => {
            ElMessage({
              type: 'success',
              message: 'Edit Canceled',
            })
          })
          .catch(() => {
            ElMessage({
              type: 'info',
              message: 'Continue Editing',
            })
          })
    },
    handleClose(done) {
      ElMessageBox.confirm('Are you sure to close this dialog?')
          .then(() => {
            done();
          })
          .catch(() => {
            // 在这里处理取消关闭的情况
          });
    },
    // flipCard(index){
    //   // 先翻转卡片
    //   this.isFlipped[index] = !this.isFlipped[index];
    //   // 再使用 setTimeout 进行延迟，让背面内容渲染出来
    //   setTimeout(() => {
    //     this.isBackRendered[index] = true;
    //   }, 250); // 250ms 延迟时间，实现翻转与背面显示无缝衔接的效果
    //   this.dialogTableVisible = true;
    //   this.currentFlippedCard = index;
    // },
    // resetCard(){
    //   // 当关闭弹出框时，重置 isFlipped 为 false
    //   this.isFlipped.forEach((value, index) => {
    //     this.isFlipped[index] = false; // 重置每个卡片的翻转状态为 false
    //     setTimeout(() => {
    //       this.isBackRendered[index] = false;
    //     }, 300); // 300ms 延迟时间，实现翻转与背面显示无缝衔接的效果
    //   });
    // }
  }

};
</script>