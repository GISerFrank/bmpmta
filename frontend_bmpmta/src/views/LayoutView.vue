<template>
  <div style="margin: 0; padding: 0; width: 100%;">
    <!-- 页面切换器 -->
    <nav style="display: flex; justify-content: space-between; align-items: center; padding-bottom: 5px; background-image: linear-gradient(60deg, #abecd6 0%, #fbed96 100%);">
      <!-- 左侧图标导航区 -->
      <div
          ref="navList"
          style="display: flex; align-items: center; position: relative;"
      >
        <!-- 左侧图标部分 -->
        <div style="display: flex; align-items: center;">
          <el-tooltip
              v-for="(item, index) in tbItems"
              :key="index"
              :content="item.content"
              placement="bottom"
              effect="light"
              style="display: flex; margin: 0"
          >
            <el-icon
                :id="item.id"
                class="page_icon_slot"
                @click="changePage($event)"
                @mouseover="handleHover(index)"
            >
              <component
                  :is="item.icon"
                  class="page_icon"
              />
            </el-icon>
          </el-tooltip>
        </div>

        <span
            ref="hoverBg"
            class="bg"
        />
      </div>

      <!-- 中间灵动岛部分 - 绝对定位于导航栏中央 -->
      <div
          class="dynamic-island"
          :class="{'pulse-animation': hasNotifications, 'expanded': isExpanded}"
          :data-type="getCurrentNotificationType()"
          @click="expandNotification"
          v-if="hasNotifications"
      >
        <!-- 波动容器 -->
        <div class="wave-container">
          <div class="wave-pulse"></div>
        </div>

        <!-- 通知图标 -->
        <div class="notification-icon-wrapper">
          <el-icon class="notification-icon"><Bell /></el-icon>
        </div>

        <!-- 通知内容 -->
        <div class="notification-content">
          <transition name="fade-slide" mode="out-in">
            <span :key="currentNotification.id">{{ currentNotification.title }}</span>
          </transition>

          <!-- 展开时显示的附加内容 -->
          <transition name="fade">
            <p v-if="isExpanded" class="notification-detail">
              {{ currentNotification.content }}
            </p>
          </transition>
        </div>

        <!-- 通知数量徽标 -->
        <div
            v-if="notifications.length > 1"
            class="notification-badge"
            :class="{'badge-pulse': notifications.length > 2}"
        >
          {{ notifications.length }}
        </div>
      </div>

      <!-- 右侧用户信息区 - 独立于左侧，确保在右侧 -->
      <div style="display: flex; align-items: center;">
        <el-dropdown
            type="primary"
            @click="handleClick"
        >
          <el-avatar
              class="avatar"
              fit="'fit'"
              :src="avatar"
          />
          User Info
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item @click="editAvatar">
                头像
              </el-dropdown-item>
              <el-dropdown-item @click="editSignature('个性签名')">
                个性签名
              </el-dropdown-item>
              <el-dropdown-item @click="editStatus('状态')">
                状态
              </el-dropdown-item>
              <el-dropdown-item @click="logOut('退出登录')">
                退出登录
              </el-dropdown-item>
              <el-dropdown-item />
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </div>
    </nav>

    <!-- 页面显示区 -->
    <keep-alive>
      <component
          :is="currentPage"
          style=""
      />
    </keep-alive>

    <el-drawer
        v-model="userDrawer"
        :direction="direction"
    >
      <template #header>
        <h4>{{ avatarAction }}</h4>
      </template>

      <!-- 头像上传区域 -->
      <div v-if="edit_avatar">
        <el-upload
            class="avatar-uploader"
            action=""
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
        >
          <img
              v-if="imageUrl"
              :src="imageUrl"
              class="avatar"
          >
          <el-icon
              v-else
              class="avatar-uploader-icon"
          >
            <Plus />
          </el-icon>
        </el-upload>
      </div>

      <!-- 个性签名编辑区域 -->
      <div v-if="avatarAction === '个性签名'" class="signature-editor">
        <p>请输入您的个性签名：</p>
        <el-input
            v-model="newSignature"
            type="textarea"
            :rows="3"
            placeholder="请输入个性签名"
            maxlength="50"
            show-word-limit
        ></el-input>
      </div>

      <!-- 状态编辑区域 -->
      <div v-if="avatarAction === '状态'" class="status-editor">
        <p>请选择您的状态：</p>
        <el-radio-group v-model="userStatus">
          <el-radio
              v-for="option in statusOptions"
              :key="option.value"
              :label="option.value"
          >
            {{ option.label }}
          </el-radio>
        </el-radio-group>
      </div>

      <!-- 退出登录确认区域 -->
      <div v-if="avatarAction === '退出登录'" class="logout-confirm">
        <p>确定要退出登录吗？</p>
      </div>

      <template #footer>
        <div style="flex: auto">
          <el-button @click="cancelClick">
            cancel
          </el-button>
          <el-button
              type="primary"
              @click="confirmClick"
          >
            confirm
          </el-button>
        </div>
      </template>
    </el-drawer>
  </div>
</template>

<script>
import HomePage from "@/components/HomePage.vue";
import BooksPage from '@/components/Books.vue';
import MoviesPage from "@/components/Movies.vue";
import MusicPage from '@/components/Music.vue';
import PhotographsPage from "@/components/PhotographsPage.vue";
import ThinkingPage from "@/components/Thinking.vue";
import AcademicsPage from "@/components/AcademicsPage.vue";
import { Reading, Film, Headset, Camera, QuestionFilled, EditPen, Plus } from '@element-plus/icons-vue';
import avatar_bear from "../assets/avatar_bear.png";
import avatar_lion from "../assets/avatar_lion.png"
import index, {mapActions, mapGetters} from "vuex";
import { Bell } from '@element-plus/icons-vue'


export default {

  name: "LayoutView",

  components:{
    Bell,
    Plus,
    HomePage,
    BooksPage,
    MoviesPage,
    MusicPage,
    PhotographsPage,
    ThinkingPage,
    AcademicsPage
  },

  data(){
    return{
      // 通知相关属性
      notifications: [
        {
          id: 1,
          title: '你有一条新消息',
          content: '张三给你发送了一条消息',
          time: '10:30',
          read: false,
          type: 'message' // 消息类型，可以用来决定显示的图标
        },
        {
          id: 2,
          title: '系统通知',
          content: '您的账号安全检查已完成',
          time: '09:15',
          read: false,
          type: 'system'
        },
        {
          id: 3,
          title: '任务提醒',
          content: '您有一个待办任务即将到期',
          time: '昨天',
          read: false,
          type: 'task'
        }
      ],
      currentNotificationIndex: 0,
      notificationTimer: null,
      isExpanded: false,    // 控制灵动岛是否展开
      notificationShown: false, // 跟踪通知是否已显示
      notificationColors: { // 不同类型通知的颜色
        message: '#409EFF',
        system: '#67C23A',
        task: '#E6A23C',
        alert: '#F56C6C'
      },
      animatingNotification: false, // 控制通知切换的动画状态
      user_id: 'bear',
      currentPage: 'HomePage', /*默认显示的界面*/
      avatar: avatar_bear,
      admin_id: "",
      userDrawer: false,
      direction: 'rtl',
      avatarAction: '',
      edit_avatar: false, // 控制头像编辑模块的显示
      imageUrl: '', // 上传的头像图片的url
      tbItems: [
        { content: "Cure-all for Soul", id: "Books", icon: Reading },
        { content: "Life on Film", id: "Movies", icon: Film },
        { content: "Drop the Beat", id: "Music", icon: Headset },
        { content: "Recording the World", id: "Photographs", icon: Camera },
        { content: "What's on Your Mind", id: "Thinking", icon: QuestionFilled },
        { content: "As a Researcher", id: "Academics", icon: EditPen },
      ],
      signature: '这是我的个性签名', // 默认个性签名
      userStatus: 'online', // 默认状态：online, away, busy, offline
      edit_signature: false, // 控制个性签名编辑模块显示
      edit_status: false, // 控制状态编辑模块显示
      newSignature: '', // 新个性签名暂存
      statusOptions: [
        { value: 'online', label: '在线' },
        { value: 'away', label: '离开' },
        { value: 'busy', label: '忙碌' },
        { value: 'offline', label: '隐身' }
      ]
    }
  },
  computed: {
    index() {
      return index
    },
    // 是否有通知
    hasNotifications() {
      return this.notifications && this.notifications.length > 0;
    },

    // 当前显示的通知
    currentNotification() {
      if (this.hasNotifications) {
        return this.notifications[this.currentNotificationIndex];
      }
      return { title: '无通知', content: '' };
    }
  },

  methods:{
    ...mapGetters(['getUserId']),
    ...mapActions(['updateUserId']), // 将store.js中的updateUserId action映射到组件的methods中

    // 修复展开/收起灵动岛的方法
    expandNotification() {
      // 切换展开状态
      this.isExpanded = !this.isExpanded;

      // 获取灵动岛元素
      const island = document.querySelector('.dynamic-island');
      if (!island) return;

      if (this.isExpanded) {
        // 展开时使用更强烈的波动效果
        const type = this.getCurrentNotificationType();
        switch (type) {
          case 'message':
            this.changeWaveEffect('double-wave');
            break;
          case 'system':
            this.changeWaveEffect('aurora-wave');
            break;
          case 'task':
            this.changeWaveEffect('breathing-wave');
            break;
          case 'alert':
            this.changeWaveEffect('glitch-wave');
            break;
          default:
            this.changeWaveEffect('breathing-wave');
        }

        // 清除轮播计时器
        this.clearNotificationTimer();

        // 添加触觉反馈
        if (navigator.vibrate) {
          navigator.vibrate(10);
        }

        // 只在第一次展开时创建通知，防止重复触发
        if (!this.notificationShown) {
          this.notificationShown = true;

          // 设置一个短暂的延时，让扩展动画有时间完成
          setTimeout(() => {
            // 防止用户已经关闭扩展
            if (!this.isExpanded) return;

            // 使用 Element Plus 的通知组件展示详情
            this.$notify({
              title: this.currentNotification.title,
              message: this.currentNotification.content,
              type: this.getNotifyType(this.currentNotification.type),
              duration: 4000,
              position: 'top-right',
              onClose: () => {
                // 仅在灵动岛还处于展开状态时才触发收起
                if (this.isExpanded) {
                  this.isExpanded = false;
                  this.setWaveColorByType();
                  this.startNotificationTimer();
                }
                this.notificationShown = false;
              }
            });
          }, 300);
        }
      } else {
        // 收起时恢复原始波动效果
        this.setWaveColorByType();

        // 重新开始轮播
        this.startNotificationTimer();

        // 重置通知标记
        this.notificationShown = false;
      }
    },

    // 清除通知轮播定时器
    clearNotificationTimer() {
      if (this.notificationTimer) {
        clearInterval(this.notificationTimer);
        this.notificationTimer = null;
      }
    },

    // 开始通知轮播 - 整合波动效果
    startNotificationTimer() {
      this.clearNotificationTimer();

      // 初始设置波动颜色
      this.setWaveColorByType();

      if (this.notifications.length > 1) {
        this.notificationTimer = setInterval(() => {
          // 平滑过渡到下一条通知
          this.currentNotificationIndex = (this.currentNotificationIndex + 1) % this.notifications.length;

          // 根据新通知类型设置波动颜色
          this.setWaveColorByType();

          // 每次切换通知时添加新的脉冲效果
          this.addPulseEffect();
        }, 5000); // 每5秒切换一次通知
      }
    },

    // 辅助方法：将内部通知类型转换为Element UI通知类型
    getNotifyType(notificationType) {
      switch (notificationType) {
        case 'message': return 'info';
        case 'system': return 'success';
        case 'task': return 'warning';
        case 'alert': return 'error';
        default: return 'info';
      }
    },

    // 添加单次脉冲效果
    addPulseEffect() {
      // 获取灵动岛元素
      const island = document.querySelector('.dynamic-island');
      if (!island) return;

      // 添加临时脉冲类
      island.classList.add('pulse-once');

      // 动画结束后移除类
      setTimeout(() => {
        island.classList.remove('pulse-once');
      }, 1000);
    },

    // 获取当前通知的类型
    getCurrentNotificationType() {
      if (!this.hasNotifications) return 'default';
      return this.currentNotification?.type || 'default';
    },

    // 切换波动效果类型
    changeWaveEffect(effectType) {
      // 移除所有效果类
      const island = document.querySelector('.dynamic-island');
      if (!island) return;

      const effectClasses = [
        'double-wave',
        'breathing-wave',
        'glitch-wave',
        'audio-wave',
        'aurora-wave'
      ];

      effectClasses.forEach(cls => {
        island.classList.remove(cls);
      });

      // 添加新的效果类
      if (effectType && effectClasses.includes(effectType)) {
        island.classList.add(effectType);

        // 如果是音频波形，需要添加波形条
        if (effectType === 'audio-wave') {
          const waveContainer = island.querySelector('.wave-container');
          if (waveContainer) {
            // 清空现有内容
            waveContainer.innerHTML = '';

            // 添加音频条
            for (let i = 0; i < 5; i++) {
              const bar = document.createElement('div');
              bar.className = 'audio-bar';
              waveContainer.appendChild(bar);
            }
          }
        }
      }
    },

    // 根据通知类型设置波动颜色
    setWaveColorByType() {
      const island = document.querySelector('.dynamic-island');
      if (!island) return;

      // 获取当前通知类型
      const type = this.getCurrentNotificationType();

      // 设置 data-type 属性
      island.setAttribute('data-type', type);

      // 根据通知类型选择波动效果
      if (!this.isExpanded) {
        // 默认状态下的波动效果
        switch (type) {
          case 'message':
            this.changeWaveEffect('');
            break;
          case 'system':
            this.changeWaveEffect('aurora-wave');
            break;
          case 'task':
            this.changeWaveEffect('double-wave');
            break;
          case 'alert':
            this.changeWaveEffect('glitch-wave');
            break;
          case 'music':
            this.changeWaveEffect('audio-wave');
            break;
          default:
            this.changeWaveEffect('breathing-wave');
        }
      }
    },

    // 清除通知状态的方法
    markNotificationAsRead(notificationId) {
      // 找到指定ID的通知
      const index = this.notifications.findIndex(n => n.id === notificationId);
      if (index !== -1) {
        // 从数组中移除该通知
        this.notifications.splice(index, 1);

        // 如果没有更多通知，重置状态
        if (this.notifications.length === 0) {
          this.isExpanded = false;
        }

        // 重置当前索引，防止超出范围
        this.currentNotificationIndex = Math.min(
            this.currentNotificationIndex,
            Math.max(0, this.notifications.length - 1)
        );

        // 重启通知计时器
        this.startNotificationTimer();
      }
    },

    setUserId(){
      this.updateUserId(this.user_id);
    },

    checkUserId(UserId){
      if (UserId===''){
        console.log('Please login first!');
        return false;
      }
      else {
        this.setUserId();
        console.log('Layout'+this.getUserId());
        if (UserId==='lion'){
          this.avatar = avatar_lion;
        }
        else {
          this.avatar = avatar_bear;
        }
        return true;
      }
    },

    cancelClick() {
      this.userDrawer = false
    },

    changePage(event){
      // 移除所有图标的active类
      const icons = document.querySelectorAll('.page_icon_slot');
      icons.forEach(icon => icon.classList.remove('active'));

      // 给当前选中的图标添加active类
      event.currentTarget.classList.add('active');
      this.currentPage = ''
      const page_name = event.currentTarget.id
      this.currentPage = page_name+"Page"
      console.log(this.currentPage)
    },

    avatar_select(admin_id){
      if (admin_id==="bear"){
        this.avatar = avatar_bear
      }
      else
        this.avatar = avatar_lion
    },

    editAvatar(event) { // 编辑头像
      this.userDrawer = true; // 打开侧边栏以进行编辑
      this.avatarAction = event.target.textContent; // 获取操作名称
      this.edit_avatar = true; // 进入头像编辑模块
    },

    handleAvatarSuccess(response, uploadFile){
      this.imageUrl = URL.createObjectURL(uploadFile?.raw)
      console.log(this.imageUrl)
    },

    saveAvatar() { // 保存头像修改

    },

    initNavEffect() {
      this.$nextTick(() => {
        this.hoverBg = this.$refs.hoverBg;
        this.navList = this.$refs.navList;

        if (!this.hoverBg || !this.navList) return;

        // Get the first icon element
        const iconContainer = this.navList.querySelector('div:first-child');
        if (iconContainer && iconContainer.children && iconContainer.children[0]) {
          const firstIcon = iconContainer.children[0];

          // Initially hide the hover background or position it under the first icon
          this.hoverBg.style.width = firstIcon.offsetWidth + "px";
          this.hoverBg.style.left = firstIcon.offsetLeft + "px";

          // Optional: initially make it invisible and only show on hover
          this.hoverBg.style.opacity = "0";
        }
      });
    },

    // 修改handleHover方法
    handleHover(index) {
      if (!this.hoverBg || !this.navList) return;
      console.log('Hover triggered for index:', index);

      // 获取所有图标元素
      const icons = this.navList.querySelectorAll('.page_icon_slot');
      if (!icons || !icons[index]) {
        console.error('找不到导航图标，索引:', index);
        return;
      }

      // 获取目标图标
      const targetIcon = icons[index];
      console.log('目标图标位置:', targetIcon.offsetLeft);

      // 显示hover背景
      this.hoverBg.style.opacity = "1";

      // 移动hover背景到图标位置
      this.animateMove(this.hoverBg, {
        left: targetIcon.offsetLeft
      });
    },

    handleMouseLeave() {
      // Optional: hide the hover background when not hovering over any icon
      if (this.hoverBg) {
        this.hoverBg.style.opacity = "0";
      }
    },

    animateMove(obj, json){
      clearInterval(obj.timer);
      let iSpeed = 0;
      obj.timer = setInterval(() => {
        let bStop = true;
        let iCur = 0;
        let iAttr = 0;
        for (let attr in json) {
          if (attr==="opacity") {
            iCur = parseFloat(getComputedStyle(obj, null)[attr])*100; // 计算当前值
            iAttr = parseFloat(json[attr])*100; // 计算目标值
          }
          else {
            iCur = parseInt(getComputedStyle(obj, null)[attr]);
            iAttr = parseInt(json[attr]);
          }
          iSpeed += (iAttr - iCur) / 3; // 弹性
          iSpeed *= 0.75; // 摩擦

          if (Math.abs(iAttr - iCur) < 1 && Math.abs(iSpeed) < 1) { // 若目标位置与当前位置间距小于1，且速度小于1，则将目标位置设置为当前interval的计划移动位置，也即滑块最终停靠位置。
            obj.style[attr] = iAttr + "px"; // 这里的iAttr是目标位置值。
          } else {
            bStop = false;
            obj.style[attr] = iCur + iSpeed + "px"; // 每个interval更新一次位置值，计算当前interval的计划移动位置，即确定当前interval结束时滑块需要移动到哪里。/
          }
        }
        if (bStop) {
          clearInterval(obj.timer);
        }
      }, 35); // interval设置为35ms
    },

    // 编辑个性签名
    editSignature(actionName) {
      this.userDrawer = true;
      this.avatarAction = actionName;
      this.edit_signature = true;
      this.edit_avatar = false;
      this.edit_status = false;
      this.newSignature = this.signature;
    },

    // 编辑状态
    editStatus(actionName) {
      this.userDrawer = true;
      this.avatarAction = actionName;
      this.edit_signature = false;
      this.edit_avatar = false;
      this.edit_status = true;
    },

    // 退出登录
    logOut(actionName) {
      this.userDrawer = true;
      this.avatarAction = actionName;
      this.edit_signature = false;
      this.edit_avatar = false;
      this.edit_status = false;
    },

    // 确认按钮点击事件
    confirmClick() {
      if (this.edit_avatar && this.imageUrl) {
        this.saveAvatar();
      } else if (this.edit_signature) {
        this.saveSignature();
      } else if (this.edit_status) {
        this.saveStatus();
      } else if (this.avatarAction === '退出登录') {
        this.confirmLogOut();
      }

      // 关闭抽屉
      this.userDrawer = false;
    },

    // 保存个性签名
    saveSignature() {
      if (this.newSignature && this.newSignature.trim() !== '') {
        this.signature = this.newSignature;
        this.$message({
          message: '个性签名已更新',
          type: 'success'
        });
      } else {
        this.$message({
          message: '个性签名不能为空',
          type: 'warning'
        });
      }
      this.edit_signature = false;
    },

    // 保存状态
    saveStatus() {
      this.$message({
        message: '状态已更新',
        type: 'success'
      });
      this.edit_status = false;
    },

    // 确认退出登录
    confirmLogOut() {
      // 清除用户数据
      this.user_id = '';

      // 如果使用了Vuex状态管理，也需要清除存储的状态
      // this.$store.dispatch('logout');

      // 重置当前页面
      this.currentPage = null;

      this.$message({
        message: '您已成功退出登录',
        type: 'success'
      });

      // 可选：重定向到登录页面
      // this.$router.push('/login');
    },

    // 初始化音频波形效果（如果有音乐类通知）
    initAudioWaveEffect() {
      // 检查是否有音乐类型的通知
      const hasMusicNotification = this.notifications.some(n => n.type === 'music');

      if (hasMusicNotification) {
        // 如果当前显示的是音乐通知，应用音频波形效果
        if (this.currentNotification.type === 'music') {
          this.changeWaveEffect('audio-wave');
        }
      }
    },

    // 处理点击事件（如果需要）
    handleClick() {
      // 可以根据需要添加处理逻辑
    },
  },

  mounted() {
    // this.user_id = this.$route.query.user_id;
    // 设置定时器，轮播通知
    console.log(this.user_id);
    if (!this.checkUserId(this.user_id)) {
      // 处理未登录逻辑
      this.currentPage = null;
    }
    else {
      this.currentPage = 'HomePage';
    }
    this.initNavEffect();

    // 确保在组件挂载后设置波动效果
    this.$nextTick(() => {
      // 设置初始波动效果
      this.setWaveColorByType();

      // 如果有通知，启动轮播
      if (this.hasNotifications) {
        this.startNotificationTimer();
      }

      // 为音乐类型通知添加特殊处理
      this.initAudioWaveEffect();
    });
  },

  beforeUnmount() {
    // 组件销毁前清除定时器
    this.clearNotificationTimer();
  },
}
</script>

<style scoped>
/* 灵动岛基本样式 - 重构添加更优雅的过渡 */
.dynamic-island {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  top: 50%;
  margin-top: -18px;
  background-color: rgba(0, 0, 0, 0.85);
  border-radius: 20px;
  min-width: 120px;
  height: 36px;
  display: flex;
  align-items: center;
  padding: 0 15px;
  cursor: pointer;
  color: white;
  overflow: hidden;
  z-index: 10;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  transition: all 0.3s cubic-bezier(0.25, 1, 0.5, 1);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-sizing: border-box;
}

/* 修复展开状态 - 更明确的尺寸变化 */
.dynamic-island.expanded {
  min-width: 280px;
  height: 80px;
  border-radius: 30px;
  padding: 12px 20px;
  transform: translateX(-50%) translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.4);
  background-color: rgba(0, 0, 0, 0.95);
}

/* 图标容器样式 - 解决展开时的布局问题 */
.notification-icon-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 10px;
  position: relative;
  z-index: 2;
  transition: all 0.3s ease;
}

.notification-icon {
  font-size: 18px;
  color: #f5f5f5;
  transition: all 0.3s ease;
}

/* 通知内容区样式 - 改进内容布局以支持展开 */
.notification-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  flex: 1;
  overflow: hidden;
  transition: all 0.3s ease;
  position: relative;
  z-index: 2;
}

.notification-content span {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;
  font-weight: 500;
  font-size: 13px;
  line-height: 1.2;
  transition: all 0.3s ease;
}

/* 展开时的通知内容样式调整 */
.dynamic-island.expanded .notification-content {
  margin-top: 4px;
}

.dynamic-island.expanded .notification-content span {
  font-size: 15px;
  font-weight: 600;
  margin-bottom: 6px;
}

/* 附加通知详情 */
.notification-detail {
  font-size: 12px;
  margin: 4px 0 0 0;
  color: rgba(255, 255, 255, 0.8);
  line-height: 1.2;
  white-space: normal;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: all 0.3s ease;
}

/* 鼠标悬停效果 */
.dynamic-island:hover {
  transform: translateX(-50%) translateY(-2px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  background-color: rgba(0, 0, 0, 0.9);
}

/* 脉冲动画效果 */
.dynamic-island.pulse-animation {
  animation: island-pulse 2s infinite;
}

@keyframes island-pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(255, 255, 255, 0.4);
  }
  70% {
    box-shadow: 0 0 0 6px rgba(255, 255, 255, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(255, 255, 255, 0);
  }
}

/* 通知数量徽标样式 */
.notification-badge {
  position: absolute;
  right: 8px;
  top: 5px;
  background-color: #f56c6c;
  border-radius: 10px;
  min-width: 16px;
  height: 16px;
  font-size: 12px;
  line-height: 16px;
  text-align: center;
  font-weight: bold;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
  z-index: 2;
}

/* 展开时徽章位置调整 */
.dynamic-island.expanded .notification-badge {
  top: 15px;
  right: 15px;
}

/* 徽标脉冲动画 */
.notification-badge.badge-pulse {
  animation: badge-pulse 1.5s infinite;
}

@keyframes badge-pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}

/* 通知内容过渡动画 */
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.3s ease;
}

.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

/* 淡入淡出动画 */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(5px);
}

/* 背景光晕效果 */
.island-glow {
  position: absolute;
  inset: 0;
  background: radial-gradient(
      circle at 30% 50%,
      rgba(255, 255, 255, 0.1) 0%,
      rgba(255, 255, 255, 0) 70%
  );
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.5s ease;
}

.dynamic-island:hover .island-glow {
  opacity: 1;
}

/* 点击/展开时的波纹效果 */
.dynamic-island::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: inherit;
  top: 0;
  left: 0;
  pointer-events: none;
  background-image: radial-gradient(circle, rgba(255, 255, 255, 0.4) 0%, transparent 60%);
  transform: scale(0);
  opacity: 0;
  transition: transform 0.5s, opacity 0.5s;
}

.dynamic-island:active::after {
  transform: scale(2);
  opacity: 0;
  transition: 0s;
}

/* 单次脉冲效果 */
.dynamic-island.pulse-once {
  animation: single-pulse 1s cubic-bezier(0.22, 1, 0.36, 1);
}

@keyframes single-pulse {
  0% {
    transform: translateX(-50%) scale(1);
  }
  50% {
    transform: translateX(-50%) scale(1.05);
  }
  100% {
    transform: translateX(-50%) scale(1);
  }
}

/* 新通知到达时的动画 */
@keyframes notification-appear {
  0% {
    transform: translateX(-50%) translateY(-20px);
    opacity: 0;
  }
  50% {
    transform: translateX(-50%) translateY(5px);
  }
  100% {
    transform: translateX(-50%) translateY(0);
    opacity: 1;
  }
}

/* 气泡出现效果 */
.dynamic-island-enter-active {
  animation: notification-appear 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.dynamic-island-leave-active {
  animation: notification-appear 0.3s cubic-bezier(0.34, 1.56, 0.64, 1) reverse;
}

/* 波动容器样式 */
.wave-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  border-radius: inherit;
  pointer-events: none;
  z-index: 1;
  transition: all 0.3s ease; /* 确保波动容器也有过渡效果 */
}

/* 波动效果样式 */
.wave-pulse {
  position: absolute;
  top: 0;
  left: -100%; /* 从左侧之外开始 */
  width: 200%; /* 宽度足够让波形完全穿过 */
  height: 100%;
  background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(66, 133, 244, 0.3) 20%, /* 蓝色波前沿 */
      rgba(219, 68, 55, 0.3) 40%,   /* 红色波中部 */
      rgba(244, 180, 0, 0.3) 60%,   /* 黄色波中部 */
      rgba(15, 157, 88, 0.3) 80%,   /* 绿色波尾部 */
      transparent 100%
  );
  animation: wave-move 3s linear infinite;
  z-index: 1;
}

/* 波形从左到右移动的动画 */
@keyframes wave-move {
  0% {
    transform: translateX(0);
    opacity: 0.6;
  }

  100% {
    transform: translateX(50%); /* 移动到右侧消失 */
    opacity: 0.6;
  }
}

/* 为扩展状态调整波动效果 */
.dynamic-island.expanded .wave-pulse {
  background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(66, 133, 244, 0.4) 20%,
      rgba(219, 68, 55, 0.4) 40%,
      rgba(244, 180, 0, 0.4) 60%,
      rgba(15, 157, 88, 0.4) 80%,
      transparent 100%
  );
  animation: wave-move 4s linear infinite;
}

/* 为不同通知类型自定义波动颜色 */
.dynamic-island[data-type="message"] .wave-pulse {
  background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(66, 133, 244, 0.3) 30%,
      rgba(66, 133, 244, 0.5) 50%,
      rgba(66, 133, 244, 0.3) 70%,
      transparent 100%
  );
}

.dynamic-island[data-type="alert"] .wave-pulse {
  background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(219, 68, 55, 0.3) 30%,
      rgba(219, 68, 55, 0.5) 50%,
      rgba(219, 68, 55, 0.3) 70%,
      transparent 100%
  );
}

.dynamic-island[data-type="task"] .wave-pulse {
  background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(244, 180, 0, 0.3) 30%,
      rgba(244, 180, 0, 0.5) 50%,
      rgba(244, 180, 0, 0.3) 70%,
      transparent 100%
  );
}

.dynamic-island[data-type="system"] .wave-pulse {
  background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(15, 157, 88, 0.3) 30%,
      rgba(15, 157, 88, 0.5) 50%,
      rgba(15, 157, 88, 0.3) 70%,
      transparent 100%
  );
}

/* 高级波动效果 - 双波动效果 */
.dynamic-island.double-wave .wave-container::before,
.dynamic-island.double-wave .wave-container::after {
  content: '';
  position: absolute;
  top: 0;
  width: 200%;
  height: 100%;
  background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(255, 255, 255, 0.1) 45%,
      rgba(255, 255, 255, 0.3) 50%,
      rgba(255, 255, 255, 0.1) 55%,
      transparent 100%
  );
  z-index: 1;
}

.dynamic-island.double-wave .wave-container::before {
  left: -200%;
  animation: wave-move-fast 2s linear infinite;
}

.dynamic-island.double-wave .wave-container::after {
  left: -150%;
  animation: wave-move-fast 2s linear infinite 1s; /* 延迟1秒制造错开效果 */
}

@keyframes wave-move-fast {
  to { transform: translateX(100%); }
}

/* 呼吸波动效果 - 从中心向外扩散 */
.dynamic-island.breathing-wave .wave-container {
  background: radial-gradient(
      circle at center,
      rgba(255, 255, 255, 0.3) 0%,
      transparent 70%
  );
  animation: breathing-pulse 3s ease-in-out infinite;
}

@keyframes breathing-pulse {
  0%, 100% {
    opacity: 0.1;
    transform: scale(0.8);
  }
  50% {
    opacity: 0.3;
    transform: scale(1.1);
  }
}

/* 带有干扰的波动效果 - 类似电视静电 */
.dynamic-island.glitch-wave .wave-container::before {
  content: '';
  position: absolute;
  inset: 0;
  background-image:
      repeating-linear-gradient(
          0deg,
          transparent,
          rgba(255, 255, 255, 0.03) 2px,
          transparent 4px
      );
  opacity: 0.2;
  animation: glitch-move 3s steps(10) infinite;
}

@keyframes glitch-move {
  0%, 100% { background-position: 0 0; }
  10% { background-position: -5px 0; }
  20% { background-position: 5px 0; }
  30% { background-position: 0 5px; }
  40% { background-position: 0 -5px; }
  50% { background-position: -5px 5px; }
  60% { background-position: 5px -5px; }
  70% { background-position: 5px 5px; }
  80% { background-position: -5px -5px; }
  90% { background-position: 0 0; }
}

/* 音乐可视化波形效果 */
.dynamic-island.audio-wave .wave-container {
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 3px;
}

.dynamic-island.audio-wave .audio-bar {
  width: 2px;
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 1px;
  transform-origin: bottom;
}

.audio-bar:nth-child(1) { animation: audio-pulse 1.2s ease-in-out 0.0s infinite alternate; }
.audio-bar:nth-child(2) { animation: audio-pulse 1.0s ease-in-out 0.1s infinite alternate; }
.audio-bar:nth-child(3) { animation: audio-pulse 1.3s ease-in-out 0.2s infinite alternate; }
.audio-bar:nth-child(4) { animation: audio-pulse 0.8s ease-in-out 0.3s infinite alternate; }
.audio-bar:nth-child(5) { animation: audio-pulse 1.1s ease-in-out 0.4s infinite alternate; }

@keyframes audio-pulse {
  0% { height: 5px; }
  100% { height: 15px; }
}

/* 极光效果波动 */
.dynamic-island.aurora-wave .wave-container {
  background: linear-gradient(
      135deg,
      rgba(32, 156, 238, 0),
      rgba(32, 156, 238, 0.3),
      rgba(139, 58, 238, 0.3),
      rgba(32, 156, 238, 0.3),
      rgba(32, 156, 238, 0)
  );
  background-size: 400% 400%;
  animation: aurora-shift 8s ease infinite;
}

@keyframes aurora-shift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* 增加手机暗模式支持 */
@media (prefers-color-scheme: dark) {
  .dynamic-island {
    background-color: rgba(30, 30, 30, 0.85);
    border: 1px solid rgba(255, 255, 255, 0.15);
  }

  .dynamic-island:hover {
    background-color: rgba(40, 40, 40, 0.9);
  }
}

/* 响应式调整 */
@media (max-width: 768px) {
  .dynamic-island {
    height: 32px;
    border-radius: 16px;
    min-width: 100px;
    padding: 0 10px;
  }

  .notification-icon {
    font-size: 16px;
    margin-right: 8px;
  }

  .notification-content span {
    font-size: 12px;
    max-width: 120px;
  }

  .notification-badge {
    right: 6px;
    top: 4px;
    min-width: 14px;
    height: 14px;
    font-size: 10px;
    line-height: 14px;
  }
}

/* 导航栏整体样式修复 */
nav {
  position: relative; /* 添加相对定位以便内部元素绝对定位 */
  margin: 0;
  padding: 0 16px; /* 添加左右内边距 */
  width: 100%;
  height: 50px; /* 固定高度 */
  display: flex;
  justify-content: space-between; /* 让左右两侧元素分别位于两端 */
  align-items: center;
  box-sizing: border-box;
}

/* 响应式调整 */
@media (max-width: 768px) {
  nav {
    padding: 0 8px;
  }

  .el-dropdown {
    margin-left: 8px;
  }
}

.moving_bubble {
  transition: transform 0.3s ease-in-out;
  z-index: 10;
}
.active-bubble {
  transform: scale(1);
}


/* 修复头像样式 */
.avatar {
  margin-right: 8px; /* 控制头像和文字之间的间距 */
  background: none;
}

.avatar:hover {
  cursor: pointer;
  transform: scale(1.05);
  transition: transform 0.2s ease;
}
/* 修复左侧图标组样式 */
.page_icon_slot {
  width: 30px;
  height: 40px;
  padding: 10px;
  border-radius: 50%;
  transition: transform 0.2s cubic-bezier(0.08, -0.03, 1.00, 0.16), filter 0.2s cubic-bezier(0.76, 0.12, 1.00, 0.32);
  z-index: 2; /* 确保在hover效果之上 */
}
.page_icon{
  width: fit-content;
  height: fit-content;
  transition: transform 0.2s cubic-bezier(0.08, -0.03, 1.00, 0.16), filter 0.2s cubic-bezier(0.76, 0.12, 1.00, 0.32);
  z-index: 1;
}
.page_icon_slot:hover{
  cursor: pointer;
}
.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter,
.expand-fade-leave-active {
  margin-left: 20px;
  opacity: 0;
}
.avatar-uploader .avatar {
  width: 178px;
  height: 178px;
  display: block;
  position: absolute;
}
.avatar-uploader {
  border: 1px dashed #990055;
  border-radius: 6px;
  cursor: pointer;
  position: absolute;
  width: 250px;
  height: 250px;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
  display: flex;
  justify-content: center;
  align-items: center;
}

.avatar-uploader:hover {
  border-color: var(--el-color-primary);
}

.avatar-uploader-icon {
  font-size: 45px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
  position: absolute;
}

.bg {
  position: absolute;
  top: 7px;
  left: 0;
  width: 40px;
  height: 40px;
  background: linear-gradient(90deg, #e72723 0%, #fd9600 100%);
  z-index: 0;
  transition: left 0.3s cubic-bezier(0.34, 1.56, 0.64, 1), opacity 0.3s;
  border-radius: 50%;
  opacity: 0.85;
  box-shadow: 0 2px 8px rgba(239, 71, 58, 0.3);
}

/* 优化图标悬停效果 */
.page_icon_slot:hover .page_icon {
  transform: scale(1.1);
  filter: brightness(1.2);
}

/* 为当前活动页面添加标识 */
.page_icon_slot.active {
  position: relative;
}

.page_icon_slot.active::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 6px;
  height: 6px;
  background-color: #e72723;
  border-radius: 50%;
}

/* 确保右侧用户区域样式 */
.el-dropdown {
  display: flex;
  align-items: center;
  cursor: pointer;
  margin-left: auto; /* 确保在右侧 */
}
</style>