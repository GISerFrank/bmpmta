<template>
  <div class="login-container">
    <h1 class="title">欢迎回来！</h1>
    <div class="cards-container">
      <!-- 小熊卡片 - 修改点击逻辑 -->
      <div class="card-wrapper" @click="toggleBearCard">
        <div class="card" :class="{ 'flipped': isBearActive }">
          <div class="card-front">
            <div class="card-content">
              <div class="card-icon">🃏</div>
              <p>点击翻开</p>
            </div>
          </div>
          <div class="card-back">
            <div class="card-content" @click="expandBearLogin">
              <div class="animal-icon">🐻</div>
              <p>小熊</p>
            </div>
          </div>
        </div>
      </div>

      <!-- 狮子卡片 - 修改点击逻辑 -->
      <div class="card-wrapper" @click="toggleLionCard">
        <div class="card" :class="{ 'flipped': isLionActive }">
          <div class="card-front">
            <div class="card-content">
              <div class="card-icon">🃏</div>
              <p>点击翻开</p>
            </div>
          </div>
          <div class="card-back">
            <div class="card-content" @click="expandLionLogin">
              <div class="animal-icon">🦁</div>
              <p>狮子</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 小熊登录表单 -->
    <div class="login-form bear-form" v-if="bearLoginExpanded">
      <h2>小熊的蜂蜜收集挑战</h2>
      <p>拖动蜂蜜罐到正确的位置来登录</p>

      <div class="honey-game">
        <div
            class="honey-jar"
            :style="{ left: honeyPosition.x + 'px', top: honeyPosition.y + 'px' }"
            @mousedown="startDragHoney"
            @touchstart="startDragHoney"
        >
          🍯
        </div>
        <div class="target-zone" :class="{ 'target-active': isHoneyInTarget }">
          <span v-if="!isHoneyInTarget">放置区域</span>
          <span v-else>完成！</span>
        </div>
      </div>

      <div class="login-actions">
        <button @click="bearLoginExpanded = false; resetGame()" class="cancel-btn">取消</button>
        <button @click="loginAsBear" :disabled="!isHoneyInTarget" class="login-btn">登录</button>
      </div>
    </div>

    <!-- 狮子登录表单 -->
    <div class="login-form lion-form" v-if="lionLoginExpanded">
      <h2>狮子的丛林迷宫</h2>
      <p>完成迷宫小游戏来登录</p>

      <div class="maze-game">
        <div class="maze-container">
          <div
              class="lion-avatar"
              :style="{ left: lionPosition.x + 'px', top: lionPosition.y + 'px' }"
          >
            🦁
          </div>
          <div class="maze-end" :class="{ 'end-reached': isLionAtEnd }">
            🚩
          </div>
          <div
              v-for="(wall, index) in mazeWalls"
              :key="'wall-' + index"
              class="maze-wall"
              :style="{ left: wall.x + 'px', top: wall.y + 'px', width: wall.width + 'px', height: wall.height + 'px' }"
          ></div>
        </div>

        <div class="maze-controls">
          <button @click="moveLion('up')">↑</button>
          <div class="horizontal-controls">
            <button @click="moveLion('left')">←</button>
            <button @click="moveLion('right')">→</button>
          </div>
          <button @click="moveLion('down')">↓</button>
        </div>
      </div>

      <div class="login-actions">
        <button @click="lionLoginExpanded = false; resetMaze()" class="cancel-btn">取消</button>
        <button @click="loginAsLion" :disabled="!isLionAtEnd" class="login-btn">登录</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.login-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  font-family: 'Arial', sans-serif;
  padding: 20px;
  position: relative;
}

.title {
  font-size: 2.5rem;
  color: #2c3e50;
  margin-bottom: 40px;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
}

.cards-container {
  display: flex;
  justify-content: center;
  gap: 50px;
  margin-bottom: 40px;
}

.card-wrapper {
  perspective: 1000px;
  width: 200px;
  height: 300px;
}

.card {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.8s ease;
  cursor: pointer;
}

.card.flipped {
  transform: rotateY(180deg);
}

.card-front, .card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.card-front {
  background: linear-gradient(45deg, #3498db, #2980b9);
  color: white;
}

.card-back {
  background: linear-gradient(45deg, #e74c3c, #c0392b);
  color: white;
  transform: rotateY(180deg);
}

.card-content {
  text-align: center;
  padding: 20px;
}

.card-icon, .animal-icon {
  font-size: 5rem;
  margin-bottom: 15px;
}

.login-form {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  padding: 30px;
  border-radius: 15px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.2);
  max-width: 500px;
  width: 90%;
  text-align: center;
  z-index: 10;
}

.honey-game, .maze-game {
  position: relative;
  height: 250px;
  background: #f9f9f9;
  border-radius: 10px;
  margin: 20px 0;
  border: 2px dashed #ddd;
  overflow: hidden;
}

.honey-jar {
  position: absolute;
  font-size: 40px;
  cursor: grab;
  user-select: none;
  z-index: 2;
}

.target-zone {
  position: absolute;
  right: 30px;
  bottom: 30px;
  width: 80px;
  height: 80px;
  border: 2px dashed #e67e22;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #e67e22;
  transition: all 0.3s;
}

.target-active {
  background: rgba(230, 126, 34, 0.1);
  border-style: solid;
}

.maze-container {
  position: relative;
  height: 180px;
  background: #f5f5f5;
  margin-bottom: 10px;
}

.lion-avatar {
  position: absolute;
  font-size: 24px;
  z-index: 2;
}

.maze-end {
  position: absolute;
  right: 10px;
  bottom: 10px;
  font-size: 24px;
}

.maze-wall {
  position: absolute;
  background: #34495e;
  border-radius: 4px;
}

.maze-controls {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.horizontal-controls {
  display: flex;
  gap: 20px;
  margin: 10px 0;
}

.maze-controls button {
  width: 50px;
  height: 50px;
  font-size: 20px;
  background: #3498db;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s;
}

.maze-controls button:hover {
  background: #2980b9;
}

.end-reached {
  animation: pulse 1s infinite;
}

.login-actions {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.login-btn, .cancel-btn {
  padding: 10px 25px;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
}

.login-btn {
  background: #27ae60;
  color: white;
}

.login-btn:disabled {
  background: #95a5a6;
  cursor: not-allowed;
}

.login-btn:not(:disabled):hover {
  background: #2ecc71;
}

.cancel-btn {
  background: #e74c3c;
  color: white;
}

.cancel-btn:hover {
  background: #c0392b;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}
</style>

<script>
// import { api } from '@/utils/request'
// import useTokenStore from '@/stores/useToken'

export default {
  name: "LoginView",

  data() {
    return {
      user_id: '', // 用于接收后端传来的userId
      user_name: '', // 用于接收后端传来的userName

      // 卡片状态
      isBearActive: false,
      isLionActive: false,

      // 登录表单状态
      bearLoginExpanded: false,
      lionLoginExpanded: false,

      // 小熊蜂蜜游戏状态
      honeyPosition: { x: 50, y: 50 },
      isDraggingHoney: false,
      honeyOffset: { x: 0, y: 0 },
      isHoneyInTarget: false,

      // 狮子迷宫游戏状态
      lionPosition: { x: 10, y: 10 },
      isLionAtEnd: false,
      mazeWalls: [
        { x: 0, y: 60, width: 150, height: 15 },
        { x: 150, y: 60, width: 15, height: 80 },
        { x: 60, y: 140, width: 105, height: 15 },
        { x: 60, y: 0, width: 15, height: 80 },
        { x: 230, y: 30, width: 15, height: 120 },
        { x: 150, y: 30, width: 80, height: 15 }
      ]
    }
  },

  mounted() {
    window.addEventListener('mousemove', this.moveHoney);
    window.addEventListener('mouseup', this.stopDragHoney);
    window.addEventListener('touchmove', this.moveHoney);
    window.addEventListener('touchend', this.stopDragHoney);
  },

  beforeUnmount() {
    window.removeEventListener('mousemove', this.moveHoney);
    window.removeEventListener('mouseup', this.stopDragHoney);
    window.removeEventListener('touchmove', this.moveHoney);
    window.removeEventListener('touchend', this.stopDragHoney);
  },

  methods: {
    // 添加新的切换方法
    toggleBearCard() {
      // 如果已经展开任一登录表单，则不做任何操作
      if (this.bearLoginExpanded || this.lionLoginExpanded) return;

      // 如果当前是小熊激活状态，则取消激活
      if (this.isBearActive) {
        this.isBearActive = false;
      } else {
        // 否则激活小熊卡片并取消狮子卡片的激活状态
        this.isBearActive = true;
        this.isLionActive = false;
      }
    },

    toggleLionCard() {
      // 如果已经展开任一登录表单，则不做任何操作
      if (this.bearLoginExpanded || this.lionLoginExpanded) return;

      // 如果当前是狮子激活状态，则取消激活
      if (this.isLionActive) {
        this.isLionActive = false;
      } else {
        // 否则激活狮子卡片并取消小熊卡片的激活状态
        this.isLionActive = true;
        this.isBearActive = false;
      }
    },

    // 卡片翻转方法
    flipBearCard() {
      this.isBearActive = true;
      this.isLionActive = false;
    },

    flipLionCard() {
      this.isLionActive = true;
      this.isBearActive = false;
    },

    // 展开登录表单
    expandBearLogin() {
      if (this.isBearActive) {
        this.bearLoginExpanded = true;
        this.resetGame();
      }
    },

    expandLionLogin() {
      if (this.isLionActive) {
        this.lionLoginExpanded = true;
        this.resetMaze();
      }
    },

    // 蜂蜜游戏方法
    startDragHoney(e) {
      this.isDraggingHoney = true;

      // 计算鼠标点击位置与蜂蜜罐位置的偏移
      const pageX = e.touches ? e.touches[0].pageX : e.pageX;
      const pageY = e.touches ? e.touches[0].pageY : e.pageY;

      const honeyElement = e.target;
      const honeyRect = honeyElement.getBoundingClientRect();

      this.honeyOffset = {
        x: pageX - honeyRect.left,
        y: pageY - honeyRect.top
      };

      // 防止拖动时选中文本
      e.preventDefault();
    },

    moveHoney(e) {
      if (!this.isDraggingHoney) return;

      const containerElement = document.querySelector('.honey-game');
      if (!containerElement) return;

      const containerRect = containerElement.getBoundingClientRect();

      const pageX = e.touches ? e.touches[0].pageX : e.pageX;
      const pageY = e.touches ? e.touches[0].pageY : e.pageY;

      // 计算新位置
      let newX = pageX - containerRect.left - this.honeyOffset.x;
      let newY = pageY - containerRect.top - this.honeyOffset.y;

      // 确保蜂蜜罐不超出容器
      newX = Math.max(0, Math.min(newX, containerRect.width - 40));
      newY = Math.max(0, Math.min(newY, containerRect.height - 40));

      this.honeyPosition = { x: newX, y: newY };

      // 检查是否放置在目标区域
      this.checkHoneyTarget();
    },

    stopDragHoney() {
      this.isDraggingHoney = false;
    },

    checkHoneyTarget() {
      const targetElement = document.querySelector('.target-zone');
      if (!targetElement) return;

      const targetRect = targetElement.getBoundingClientRect();
      const honeyElement = document.querySelector('.honey-jar');
      const honeyRect = honeyElement.getBoundingClientRect();

      // 检查蜂蜜罐中心点是否在目标区域内
      const honeyCenter = {
        x: honeyRect.left + honeyRect.width / 2,
        y: honeyRect.top + honeyRect.height / 2
      };

      this.isHoneyInTarget =
          honeyCenter.x >= targetRect.left &&
          honeyCenter.x <= targetRect.right &&
          honeyCenter.y >= targetRect.top &&
          honeyCenter.y <= targetRect.bottom;
    },

    resetGame() {
      this.honeyPosition = { x: 50, y: 50 };
      this.isHoneyInTarget = false;
    },

    // 狮子迷宫方法
    moveLion(direction) {
      const step = 10;
      let newX = this.lionPosition.x;
      let newY = this.lionPosition.y;

      switch (direction) {
        case 'up':
          newY -= step;
          break;
        case 'down':
          newY += step;
          break;
        case 'left':
          newX -= step;
          break;
        case 'right':
          newX += step;
          break;
      }

      // 检查新位置是否碰到墙壁
      if (!this.checkWallCollision(newX, newY)) {
        // 检查是否超出边界
        const container = document.querySelector('.maze-container');
        if (container) {
          const containerWidth = container.clientWidth;
          const containerHeight = container.clientHeight;

          if (newX >= 0 && newX <= containerWidth - 30 &&
              newY >= 0 && newY <= containerHeight - 30) {
            this.lionPosition = { x: newX, y: newY };
            this.checkMazeEnd();
          }
        }
      }
    },

    checkWallCollision(x, y) {
      // 狮子的大小
      const lionSize = 24;

      // 检查是否与任何一面墙碰撞
      for (const wall of this.mazeWalls) {
        if (
            x < wall.x + wall.width &&
            x + lionSize > wall.x &&
            y < wall.y + wall.height &&
            y + lionSize > wall.y
        ) {
          return true; // 碰撞
        }
      }

      return false; // 没有碰撞
    },

    checkMazeEnd() {
      const endElement = document.querySelector('.maze-end');
      if (!endElement) return;

      const endRect = endElement.getBoundingClientRect();
      const lionElement = document.querySelector('.lion-avatar');
      const lionRect = lionElement.getBoundingClientRect();

      // 检查狮子是否接触到终点
      this.isLionAtEnd =
          lionRect.right >= endRect.left &&
          lionRect.left <= endRect.right &&
          lionRect.bottom >= endRect.top &&
          lionRect.top <= endRect.bottom;
    },

    resetMaze() {
      this.lionPosition = { x: 10, y: 10 };
      this.isLionAtEnd = false;
    },

    // 登录方法
    loginAsBear() {
      if (this.isHoneyInTarget) {
        this.user_id = 'bear';
        // 保存到localStorage以便在页面刷新后仍能保持登录状态
        localStorage.setItem('user_id', this.user_id);

        // 检查是否有重定向信息
        const redirect = this.$route.query.redirect || '/layout';
        this.$router.push(redirect);
      }
    },

    loginAsLion() {
      if (this.isLionAtEnd) {
        this.user_id = 'lion';
        // 保存到localStorage以便在页面刷新后仍能保持登录状态
        localStorage.setItem('user_id', this.user_id);

        // 检查是否有重定向信息
        const redirect = this.$route.query.redirect || '/layout';
        this.$router.push(redirect);
      }
    }
  }
}
</script>