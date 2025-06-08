<script setup>
// 完整的Vue组件script部分 - 解决模式切换问题

import {ref, onMounted, onUnmounted, computed} from 'vue'

// 👇 响应式数据
const user_id = ref('lion')

// 👇 签到相关数据
const checkedDates = ref(new Set())
const todayDate = ref(new Date().toISOString().split('T')[0])
const selectedDate = ref(null)

// 👇 全局组件控制
const useGlobalTetris = ref(true) // 设置为true使用全局可拖动组件
const showVueTetris = ref(false) // 是否显示Vue组件

// 👇 俄罗斯方块相关数据 - 统一声明
const pieceCount = ref(0)
const stats = ref({})

// 👇 拖动组件相关数据（仅在Vue组件模式时使用）
const tetrisWidget = ref(null)
const isDragging = ref(false)
const widgetPosition = ref({
  x: 20,
  y: 20
})

// 👇 事件监听器引用
let resizeHandler = null
let statsUpdateInterval = null

const debugTetrisComponent = () => {
  const status = checkGlobalComponentStatus()
  const debugInfo = `🔍 俄罗斯方块组件调试信息

🌐 当前模式: ${useGlobalTetris.value ? '全局模式' : 'Vue模式'}
📊 方块数量: ${pieceCount.value}

🔧 全局组件状态:
• 脚本已加载: ${status.hasScript ? '✅' : '❌'}
• 实例存在: ${status.hasInstance ? '✅' : '❌'}
• DOM存在: ${status.hasDOM ? '✅' : '❌'}

💾 本地存储:
• 签到数据: ${localStorage.getItem('tetris_checkin_data') ? '✅' : '❌'}
• 组件位置: ${localStorage.getItem('tetris_widget_position') ? '✅' : '❌'}
• 全局数据: ${localStorage.getItem('tetris_accumulator_data') ? '✅' : '❌'}

🎯 操作建议:
${!status.hasScript ? '• 需要加载全局脚本\n' : ''}
${!status.hasInstance ? '• 需要创建组件实例\n' : ''}
${!status.hasDOM ? '• 需要创建DOM元素\n' : ''}

📝 统计数据: ${JSON.stringify(stats.value, null, 2)}`

  console.log(debugInfo)
  alert(debugInfo)

  // 如果组件有问题，提供修复选项
  if (useGlobalTetris.value && (!status.hasInstance || !status.hasDOM)) {
    const shouldFix = confirm('检测到全局组件可能有问题，是否尝试修复？')
    if (shouldFix) {
      forceRefreshGlobalComponent()
    }
  }
}

// 👇 改进的切换组件模式方法
const switchTetrisMode = () => {
  console.log(`准备切换模式: ${useGlobalTetris.value ? 'Vue' : '全局'}`)

  // 清理现有的定时器
  if (statsUpdateInterval) {
    clearInterval(statsUpdateInterval)
    statsUpdateInterval = null
  }

  useGlobalTetris.value = !useGlobalTetris.value

  if (useGlobalTetris.value) {
    // 切换到全局组件
    console.log('🔄 切换到全局可拖动组件模式')
    showVueTetris.value = false

    // 重新创建全局组件
    createGlobalTetrisComponent()

  } else {
    // 切换到Vue组件
    console.log('🔄 切换到Vue组件模式')

    // 销毁全局组件
    destroyGlobalTetrisComponent()

    // 启用Vue组件
    showVueTetris.value = true

    // 重新初始化Vue组件位置
    setTimeout(() => {
      resetVueComponentPosition()
    }, 100)
  }
}

// 👇 创建全局组件
const createGlobalTetrisComponent = async () => {
  try {
    // 先销毁现有组件
    destroyGlobalTetrisComponent()

    // 确保脚本已加载
    if (!window.TetrisAccumulator) {
      console.log('🔄 全局脚本未加载，正在加载...')
      await loadGlobalTetrisScript()
    }

    // 创建新的组件实例
    if (window.TetrisAccumulator) {
      console.log('🔧 创建新的全局组件实例')

      // 等待一帧，确保DOM清理完成
      await new Promise(resolve => requestAnimationFrame(resolve))

      window.tetrisAccumulator = new window.TetrisAccumulator()

      // 等待组件创建完成
      setTimeout(() => {
        if (document.getElementById('tetris-widget')) {
          console.log('✅ 全局组件创建成功')
          updateTetrisStats()

          // 启动定期更新
          statsUpdateInterval = setInterval(updateTetrisStats, 2000)

          // 触发一些奖励
          setTimeout(() => {
            triggerActivity('切换到全局组件模式')
          }, 500)
        } else {
          console.error('❌ 全局组件DOM未创建')
        }
      }, 1000)
    }
  } catch (error) {
    console.error('❌ 创建全局组件失败:', error)
    // 回退到Vue模式
    useGlobalTetris.value = false
    showVueTetris.value = true
    alert('切换到全局模式失败，已回退到Vue模式')
  }
}

// 👇 销毁全局组件
const destroyGlobalTetrisComponent = () => {
  console.log('🗑️ 销毁全局组件')

  // 清理定时器
  if (statsUpdateInterval) {
    clearInterval(statsUpdateInterval)
    statsUpdateInterval = null
  }

  // 销毁实例
  if (window.tetrisAccumulator) {
    try {
      window.tetrisAccumulator.destroy()
    } catch (error) {
      console.warn('销毁全局组件实例时出错:', error)
    }
    window.tetrisAccumulator = null
  }

  // 移除DOM元素
  const widgets = document.querySelectorAll('#tetris-widget')
  widgets.forEach(widget => widget.remove())

  console.log('✅ 全局组件已清理')
}

// 👇 重置Vue组件位置
const resetVueComponentPosition = () => {
  const isMobile = window.innerWidth <= 768
  widgetPosition.value = {
    x: isMobile ? window.innerWidth - 200 : window.innerWidth - 240,
    y: isMobile ? 60 : 20
  }
  loadWidgetPosition()
  console.log('Vue组件位置已重置:', widgetPosition.value)
}

// 👇 加载全局脚本
const loadGlobalTetrisScript = () => {
  return new Promise((resolve, reject) => {
    console.log('📥 开始加载全局俄罗斯方块脚本')

    // 移除现有脚本
    const existingScripts = document.querySelectorAll('script[src*="tetris-easy-integration"]')
    existingScripts.forEach(script => script.remove())

    // 移除现有样式
    const existingStyles = document.getElementById('tetris-styles')
    if (existingStyles) {
      existingStyles.remove()
    }

    const script = document.createElement('script')
    script.src = '/tetris-easy-integration.js' // 确保这个路径正确
    script.async = true

    script.onload = () => {
      console.log('✅ 全局脚本加载完成')
      // 给脚本一些时间初始化
      setTimeout(() => {
        if (window.TetrisAccumulator) {
          resolve()
        } else {
          reject(new Error('脚本加载完成但TetrisAccumulator类未找到'))
        }
      }, 500)
    }

    script.onerror = (error) => {
      console.error('❌ 脚本加载失败:', error)
      reject(new Error('脚本加载失败'))
    }

    document.head.appendChild(script)

    // 超时保护
    setTimeout(() => {
      if (!window.TetrisAccumulator) {
        reject(new Error('脚本加载超时'))
      }
    }, 10000)
  })
}

// 👇 改进的触发活动方法
const triggerActivity = (reason) => {
  console.log('🎯 触发活动:', reason)

  if (useGlobalTetris.value && window.tetrisAccumulator) {
    try {
      window.tetrisAccumulator.addCustomPiece(reason)
      setTimeout(updateTetrisStats, 200) // 延迟更新统计
    } catch (error) {
      console.error('触发全局组件活动失败:', error)
    }
  } else if (!useGlobalTetris.value) {
    // Vue组件模式的简单逻辑
    console.log('Vue组件模式活动:', reason)
    pieceCount.value += 1
    if (!stats.value.activities) stats.value.activities = 0
    stats.value.activities += 1
  } else {
    console.warn('俄罗斯方块组件未准备就绪:', reason)
  }
}

// 👇 更新统计数据
const updateTetrisStats = () => {
  if (useGlobalTetris.value && window.tetrisAccumulator) {
    try {
      const currentStats = window.tetrisAccumulator.getStats()
      pieceCount.value = currentStats.pieces || 0
      stats.value = currentStats.stats || {}
      console.log('📊 统计更新:', { pieces: pieceCount.value, stats: stats.value })
    } catch (error) {
      console.error('更新统计失败:', error)
    }
  }
}

// 👇 检查全局组件状态
const checkGlobalComponentStatus = () => {
  const hasScript = !!window.TetrisAccumulator
  const hasInstance = !!window.tetrisAccumulator
  const hasDOM = !!document.getElementById('tetris-widget')

  console.log('🔍 全局组件状态检查:', { hasScript, hasInstance, hasDOM })

  return { hasScript, hasInstance, hasDOM }
}

// 👇 强制刷新全局组件
const forceRefreshGlobalComponent = () => {
  console.log('🔄 强制刷新全局组件')
  if (useGlobalTetris.value) {
    createGlobalTetrisComponent()
  }
}

// 以下方法保持不变...
const startDrag = (event) => {
  if (useGlobalTetris.value) return

  console.log('开始拖动', event.type)

  if (!tetrisWidget.value) {
    console.error('tetrisWidget 引用不存在')
    return
  }

  isDragging.value = true
  document.body.classList.add('dragging')

  const rect = tetrisWidget.value.getBoundingClientRect()
  const clientX = event.clientX || (event.touches && event.touches[0].clientX)
  const clientY = event.clientY || (event.touches && event.touches[0].clientY)

  const offsetX = clientX - rect.left
  const offsetY = clientY - rect.top

  const handleMove = (e) => {
    if (isDragging.value) {
      const moveX = e.clientX || (e.touches && e.touches[0].clientX)
      const moveY = e.clientY || (e.touches && e.touches[0].clientY)

      const newX = moveX - offsetX
      const newY = moveY - offsetY

      const maxX = window.innerWidth - 220
      const maxY = window.innerHeight - 120

      widgetPosition.value.x = Math.max(0, Math.min(newX, maxX))
      widgetPosition.value.y = Math.max(0, Math.min(newY, maxY))
    }
  }

  const handleEnd = () => {
    console.log('结束拖动')
    isDragging.value = false
    document.body.classList.remove('dragging')

    document.removeEventListener('mousemove', handleMove)
    document.removeEventListener('mouseup', handleEnd)
    document.removeEventListener('touchmove', handleMove)
    document.removeEventListener('touchend', handleEnd)

    try {
      localStorage.setItem('tetris_widget_position', JSON.stringify(widgetPosition.value))
    } catch (error) {
      console.error('保存组件位置失败:', error)
    }

    triggerActivity('拖动俄罗斯方块组件')
  }

  document.addEventListener('mousemove', handleMove)
  document.addEventListener('mouseup', handleEnd)
  document.addEventListener('touchmove', handleMove, {passive: false})
  document.addEventListener('touchend', handleEnd)

  event.preventDefault()
  event.stopPropagation()
}

const loadWidgetPosition = () => {
  if (useGlobalTetris.value) return

  try {
    const saved = localStorage.getItem('tetris_widget_position')
    if (saved) {
      const position = JSON.parse(saved)
      widgetPosition.value = {
        x: Math.max(0, Math.min(position.x, window.innerWidth - 220)),
        y: Math.max(0, Math.min(position.y, window.innerHeight - 120))
      }
    }
  } catch (error) {
    console.error('加载组件位置失败:', error)
  }
}

const resetWidgetPosition = () => {
  if (useGlobalTetris.value) return

  resetVueComponentPosition()
  try {
    localStorage.removeItem('tetris_widget_position')
    triggerActivity('重置俄罗斯方块组件位置')
  } catch (error) {
    console.error('重置组件位置失败:', error)
  }
}

// 计算属性
const todayCheckedIn = computed(() => {
  return checkedDates.value.has(todayDate.value)
})

const totalCheckedDays = computed(() => {
  return checkedDates.value.size
})

// 签到相关方法保持不变...
function handleDateClick(data) {
  const clickedDate = data.day
  selectedDate.value = clickedDate

  const clickedDateObj = new Date(clickedDate)
  const todayDateObj = new Date(todayDate.value)

  if (clickedDateObj > todayDateObj) {
    alert('🚫 不能签到未来的日期哦！')
    triggerActivity('尝试签到未来日期')
    return
  }

  if (!checkedDates.value.has(clickedDate)) {
    checkedDates.value.add(clickedDate)
    saveCheckinData()

    const isToday = clickedDate === todayDate.value
    const isPastDate = clickedDateObj < todayDateObj

    let message = ''
    if (isToday) {
      message = '🎉 今日签到成功！'
      triggerActivity('今日签到')
      setTimeout(() => triggerActivity('连续签到奖励'), 500)
    } else if (isPastDate) {
      message = `📝 补签${clickedDate}成功！`
      triggerActivity(`补签日期: ${clickedDate}`)
    }

    alert(message)
  } else {
    const isToday = clickedDate === todayDate.value
    const message = isToday ? '✅ 今日已签到' : `✅ ${clickedDate} 已签到`
    alert(message)
    triggerActivity('查看签到记录')
  }
}

function saveCheckinData() {
  try {
    const checkinData = {
      userId: user_id.value,
      checkedDates: Array.from(checkedDates.value),
      lastUpdate: new Date().toISOString()
    }
    localStorage.setItem('tetris_checkin_data', JSON.stringify(checkinData))
  } catch (error) {
    console.error('保存签到数据失败:', error)
  }
}

function loadCheckinData() {
  try {
    const saved = localStorage.getItem('tetris_checkin_data')
    if (saved) {
      const data = JSON.parse(saved)
      if (data.userId === user_id.value) {
        checkedDates.value = new Set(data.checkedDates || [])
      }
    }
  } catch (error) {
    console.error('加载签到数据失败:', error)
  }
}

function isDateChecked(dateStr) {
  return checkedDates.value.has(dateStr)
}

function getConsecutiveDays() {
  const today = new Date()
  let consecutive = 0

  for (let i = 0; i < 30; i++) {
    const checkDate = new Date(today)
    checkDate.setDate(today.getDate() - i)
    const dateStr = checkDate.toISOString().split('T')[0]

    if (checkedDates.value.has(dateStr)) {
      consecutive++
    } else if (i === 0) {
      break
    } else {
      break
    }
  }

  return consecutive
}

function switchUser() {
  saveCheckinData()
  const newUserId = user_id.value === 'lion' ? 'bear' : 'lion'
  user_id.value = newUserId
  loadCheckinData()
  triggerActivity(`切换到${user_id.value === 'lion' ? '狮子' : '熊'}模式`)
}

function showTetrisDetails() {
  if (useGlobalTetris.value && window.tetrisAccumulator) {
    window.tetrisAccumulator.showPieces()
  } else {
    const message = `🧩 俄罗斯方块详情

当前方块数: ${pieceCount.value}
页面访问: ${stats.value.visits || 0}
滚动次数: ${stats.value.scrolls || 0}
点击次数: ${stats.value.clicks || 0}
停留时间: ${stats.value.timeSpent || 0}分钟

💡 提示：继续浏览页面和签到可以获得更多方块！`

    alert(message)
  }

  triggerActivity('查看俄罗斯方块详情')
}

function startTetrisGame() {
  if (useGlobalTetris.value && window.tetrisAccumulator) {
    window.tetrisAccumulator.startGame()
  } else {
    if (pieceCount.value > 0) {
      const confirmed = confirm(`🎮 准备开始俄罗斯方块游戏！

你有 ${pieceCount.value} 个方块可以使用。

点击确定开始游戏`)

      if (confirmed) {
        triggerActivity('启动俄罗斯方块游戏')
        alert('🚧 游戏功能开发中，敬请期待！')
      }
    } else {
      alert('还没有方块可以游戏！继续浏览页面和签到来获取方块。')
    }
  }
}

function handleCalendarInteraction() {
  triggerActivity('与日历交互')
}

function claimDailyBonus() {
  if (todayCheckedIn.value) {
    triggerActivity('每日签到奖励')
    alert('🎉 获得每日签到奖励！')
  } else {
    alert('请先完成今日签到！')
  }
}

function showCheckinStats() {
  const consecutive = getConsecutiveDays()
  const total = totalCheckedDays.value
  const todayStatus = todayCheckedIn.value ? '已签到 ✅' : '未签到 ❌'

  const message = `📊 签到统计
今日状态: ${todayStatus}
连续签到: ${consecutive} 天
累计签到: ${total} 天

💡 温馨提示：
• 只能签到今天和过去的日期
• 未来日期显示为灰色，不可签到
• 今日签到有额外俄罗斯方块奖励`

  alert(message)
  triggerActivity('查看签到统计')
}

// 生命周期钩子
onMounted(() => {
  console.log('📅 日历组件已挂载')

  // 通用初始化
  loadCheckinData()

  if (useGlobalTetris.value) {
    console.log('🌐 初始化全局模式')

    // 检查现有状态
    const status = checkGlobalComponentStatus()

    if (status.hasScript && status.hasInstance && status.hasDOM) {
      console.log('✅ 发现现有的全局组件')
      updateTetrisStats()
      statsUpdateInterval = setInterval(updateTetrisStats, 2000)
    } else {
      console.log('🔧 需要创建全局组件')
      createGlobalTetrisComponent()
    }

    // 触发页面访问奖励
    setTimeout(() => {
      triggerActivity('访问日历页面')
      if (user_id.value === 'lion') {
        triggerActivity('狮子用户访问')
      } else if (user_id.value === 'bear') {
        triggerActivity('熊用户访问')
      }
    }, 2000)

  } else {
    console.log('🔧 初始化Vue模式')
    showVueTetris.value = true
    resetVueComponentPosition()

    setTimeout(() => {
      if (tetrisWidget.value) {
        console.log('tetrisWidget 引用已绑定')
      } else {
        console.error('tetrisWidget 引用未绑定')
      }
    }, 100)

    resizeHandler = () => {
      const maxX = window.innerWidth - 220
      const maxY = window.innerHeight - 120
      widgetPosition.value.x = Math.min(widgetPosition.value.x, maxX)
      widgetPosition.value.y = Math.min(widgetPosition.value.y, maxY)
    }

    window.addEventListener('resize', resizeHandler)
  }

  if (!todayCheckedIn.value) {
    setTimeout(() => {
      console.log('💡 提醒：今日还未签到哦！')
    }, 3000)
  }

  console.log('✅ 日历页面初始化完成')
})

onUnmounted(() => {
  console.log('🗑️ 组件卸载')

  if (resizeHandler) {
    window.removeEventListener('resize', resizeHandler)
  }

  if (statsUpdateInterval) {
    clearInterval(statsUpdateInterval)
  }

  saveCheckinData()
})

// 暴露给模板的调试方法
window.debugTetris = {
  checkStatus: checkGlobalComponentStatus,
  forceRefresh: forceRefreshGlobalComponent,
  switchMode: switchTetrisMode
}
</script>

<template>
  <div>
    <!-- 👇 签到和状态显示区域 -->
    <div class="status-container">
      <div class="main-status-bar">
        <div class="user-section">
          <span v-if="user_id === 'lion'" class="user-badge lion">🦁 {{ user_id.toUpperCase() }}</span>
          <span v-else-if="user_id === 'bear'" class="user-badge bear">🐻 {{ user_id.toUpperCase() }}</span>
          <span class="checkin-status" :class="{ 'checked-in': todayCheckedIn }">
            {{ todayCheckedIn ? '✅ 已签到' : '❌ 未签到' }}
          </span>
        </div>

        <div class="action-section">
          <button @click="switchUser" class="action-btn switch-btn">
            🔄 切换身份
          </button>
          <button @click="showCheckinStats" class="action-btn stats-btn">
            📊 签到统计
          </button>
          <!-- 👇 新增：组件模式切换按钮 -->
          <button @click="switchTetrisMode" class="action-btn tetris-mode-btn" title="切换俄罗斯方块组件模式">
            {{ useGlobalTetris ? '🌐 全局' : '🔧 Vue' }}
          </button>
          <!-- 👇 调试按钮（开发时使用） -->
          <button @click="debugTetrisComponent" class="action-btn debug-btn" title="调试俄罗斯方块组件">
            🔍 调试
          </button>
        </div>
      </div>
    </div>

    <!-- 👇 唯一的可拖动俄罗斯方块组件 -->
    <div
        v-if="showVueTetris && !useGlobalTetris"
        ref="tetrisWidget"
        class="draggable-tetris-widget"
        :class="{ dragging: isDragging }"
        :style="{ left: widgetPosition.x + 'px', top: widgetPosition.y + 'px' }"
    >
      <div
          class="tetris-header"
          @mousedown.prevent="startDrag"
          @touchstart.prevent="startDrag"
      >
        <span class="drag-handle">≡≡</span>
        <span class="tetris-icon">🧩</span>
        <span class="tetris-count">{{ pieceCount }}</span>
        <span class="tetris-label">方块</span>
        <button
            @click.stop="resetWidgetPosition"
            class="reset-position-btn"
            title="重置位置"
        >
          📍
        </button>
      </div>
      <div class="tetris-body" v-if="pieceCount > 0">
        <div class="tetris-actions">
          <button
              @click="startTetrisGame"
              class="tetris-btn primary"
          >
            🎮 游戏
          </button>
          <button @click="showTetrisDetails" class="tetris-btn secondary">
            📦 详情
          </button>
        </div>
      </div>
    </div>

    <!-- 👇 签到状态面板 -->
    <div class="checkin-panel">
      <div class="checkin-summary">
        <div class="summary-item">
          <span class="summary-label">今日状态</span>
          <span class="summary-value" :class="{ 'success': todayCheckedIn }">
            {{ todayCheckedIn ? '已签到 🎉' : '未签到 📝' }}
          </span>
        </div>
        <div class="summary-item">
          <span class="summary-label">连续签到</span>
          <span class="summary-value">{{ getConsecutiveDays() }} 天</span>
        </div>
        <div class="summary-item">
          <span class="summary-label">累计签到</span>
          <span class="summary-value">{{ totalCheckedDays }} 天</span>
        </div>
        <div class="summary-item">
          <span class="summary-label">俄罗斯方块</span>
          <span class="summary-value tetris-highlight">{{ pieceCount }} 个</span>
        </div>
      </div>

      <div class="daily-actions" v-if="todayCheckedIn">
        <button @click="claimDailyBonus" class="bonus-btn">
          🎁 领取每日奖励
        </button>
        <button @click="showTetrisDetails" class="details-btn">
          📦 查看方块
        </button>
      </div>
    </div>

    <!-- 👇 增强的日历组件，显示签到状态和日期限制 -->
    <el-calendar class="calendar" @click="handleCalendarInteraction">
      <template #date-cell="{ data }">
        <!-- 👇 狮子用户的日期显示 -->
        <div
            v-if="user_id === 'lion'"
            :class="[
            'date-cell',
            'lion-cell',
            {
              'is-selected': data.isSelected,
              'is-checked': isDateChecked(data.day),
              'is-today': data.day === todayDate,
              'is-future': new Date(data.day) > new Date(todayDate),
              'is-clickable': new Date(data.day) <= new Date(todayDate)
            }
          ]"
            @click.stop="handleDateClick(data)"
            :title="new Date(data.day) > new Date(todayDate) ? '未来日期不可签到' :
                  isDateChecked(data.day) ? '已签到' :
                  data.day === todayDate ? '点击今日签到' : '点击补签'"
        >
          <div class="date-content">
            <span class="date-number">{{ data.day.split('-').slice(1).join('-') }}</span>
            <div class="date-status">
              <span v-if="data.isSelected">✔️🦁</span>
              <span v-else-if="isDateChecked(data.day)" class="check-mark">✅</span>
              <span v-else-if="data.day === todayDate" class="today-mark">📅</span>
              <span v-else-if="new Date(data.day) > new Date(todayDate)" class="future-mark">🚫</span>
            </div>
          </div>
        </div>

        <!-- 👇 熊用户的日期显示 -->
        <div
            v-else-if="user_id === 'bear'"
            :class="[
            'date-cell',
            'bear-cell',
            {
              'is-selected': data.isSelected,
              'is-checked': isDateChecked(data.day),
              'is-today': data.day === todayDate,
              'is-future': new Date(data.day) > new Date(todayDate),
              'is-clickable': new Date(data.day) <= new Date(todayDate)
            }
          ]"
            @click.stop="handleDateClick(data)"
            :title="new Date(data.day) > new Date(todayDate) ? '未来日期不可签到' :
                  isDateChecked(data.day) ? '已签到' :
                  data.day === todayDate ? '点击今日签到' : '点击补签'"
        >
          <div class="date-content">
            <span class="date-number">{{ data.day.split('-').slice(1).join('-') }}</span>
            <div class="date-status">
              <span v-if="data.isSelected">✔️🐻</span>
              <span v-else-if="isDateChecked(data.day)" class="check-mark">✅</span>
              <span v-else-if="data.day === todayDate" class="today-mark">📅</span>
              <span v-else-if="new Date(data.day) > new Date(todayDate)" class="future-mark">🚫</span>
            </div>
          </div>
        </div>
      </template>
    </el-calendar>

    <!-- 👇 详细统计信息 -->
    <div class="tetris-stats">
      <h4>🎯 俄罗斯方块状态</h4>
      <div class="mode-indicator">
        <div class="mode-badge" :class="{ active: useGlobalTetris }">
          <span class="mode-icon">🌐</span>
          <div class="mode-info">
            <strong>全局可拖动组件</strong>
            <small>所有页面可用，支持拖动和位置保存</small>
          </div>
        </div>
        <div class="mode-badge" :class="{ active: !useGlobalTetris }">
          <span class="mode-icon">🔧</span>
          <div class="mode-info">
            <strong>Vue组件</strong>
            <small>仅当前页面，集成度更高</small>
          </div>
        </div>
      </div>
      <div class="stats-grid" v-if="stats && Object.keys(stats).length > 0">
        <div class="stat-item">
          <span class="stat-label">页面访问:</span>
          <span class="stat-value">{{ stats.visits || 0 }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">日期点击:</span>
          <span class="stat-value">{{ stats.clicks || 0 }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">滚动次数:</span>
          <span class="stat-value">{{ stats.scrolls || 0 }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">停留时间:</span>
          <span class="stat-value">{{ stats.timeSpent || 0 }}分钟</span>
        </div>
      </div>
    </div>

    <!-- 👇 使用说明 - 更新了拖动说明 -->
    <div class="user-guide">
      <h4>📖 使用说明</h4>
      <div class="guide-content">
        <div class="guide-item">
          <span class="guide-icon">📅</span>
          <div class="guide-text">
            <strong>今日签到：</strong>点击今天的日期完成签到，获得俄罗斯方块奖励
          </div>
        </div>
        <div class="guide-item">
          <span class="guide-icon">📝</span>
          <div class="guide-text">
            <strong>补签功能：</strong>可以补签过去的日期，但奖励相对较少
          </div>
        </div>
        <div class="guide-item">
          <span class="guide-icon">🎯</span>
          <div class="guide-text">
            <strong>可拖动组件：</strong>
            <span v-if="useGlobalTetris">右上角的俄罗斯方块组件可以拖动！拖动标题栏移动到任意位置，点击📍重置位置。全局组件在所有页面都可用。</span>
            <span v-else>当前使用Vue集成组件，仅在此页面可用。</span>
          </div>
        </div>
        <div class="guide-item">
          <span class="guide-icon">🔄</span>
          <div class="guide-text">
            <strong>组件模式：</strong>点击右上角的"{{ useGlobalTetris ? '🌐 全局' : '🔧 Vue' }}"按钮可以切换组件模式
          </div>
        </div>
        <div class="guide-item">
          <span class="guide-icon">🦁🐻</span>
          <div class="guide-text">
            <strong>身份切换：</strong>
            <span v-if="user_id === 'lion'" class="current-user">当前是狮子模式</span>
            <span v-else class="current-user">当前是熊模式</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 👇 在你的Vue组件的 <style scoped> 中添加这些样式 */
.debug-btn {
  background: linear-gradient(45deg, #ffc107, #ff9800) !important;
  font-size: 11px !important;
}

.debug-btn:hover {
  background: linear-gradient(45deg, #ffb300, #f57500) !important;
}

/* 调试模式指示器 */
.debug-info {
  position: fixed;
  top: 10px;
  left: 10px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 8px;
  border-radius: 4px;
  font-size: 12px;
  z-index: 9998;
  font-family: monospace;
  max-width: 200px;
  display: none;
}

.debug-mode .debug-info {
  display: block;
}

/* 添加一个开发模式的视觉指示器 */
.main-status-bar.debug-mode::before {
  content: "🔍 调试模式";
  position: absolute;
  top: -20px;
  right: 0;
  background: #ffc107;
  color: #000;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 10px;
  font-weight: bold;
}

/* 新增：组件模式切换按钮 */
.tetris-mode-btn {
  background: linear-gradient(45deg, #17a2b8, #007bff) !important;
  font-size: 11px !important;
  padding: 6px 10px !important;
}

.tetris-mode-btn:hover {
  background: linear-gradient(45deg, #138496, #0056b3) !important;
}

/* 模式指示器样式 */
.mode-indicator {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-bottom: 16px;
  padding: 16px;
  background: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #e9ecef;
}

.mode-badge {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  border-radius: 8px;
  border: 2px solid transparent;
  background: white;
  transition: all 0.3s ease;
  opacity: 0.6;
}

.mode-badge.active {
  opacity: 1;
  border-color: #667eea;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.2);
  background: linear-gradient(135deg, #f8f9ff, #ffffff);
}

.mode-icon {
  font-size: 24px;
  flex-shrink: 0;
}

.mode-info {
  flex: 1;
}

.mode-info strong {
  display: block;
  color: #333;
  font-size: 14px;
  margin-bottom: 4px;
}

.mode-info small {
  color: #666;
  font-size: 12px;
  line-height: 1.3;
}

/* 改进的统计区域 */
.tetris-stats h4 {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 20px;
}

/* 状态栏按钮组改进 */
.action-section {
  display: flex;
  gap: 6px;
  justify-self: end;
  flex-wrap: wrap;
}

.action-btn {
  font-size: 11px;
  padding: 6px 10px;
  white-space: nowrap;
  min-width: 60px;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .mode-indicator {
    grid-template-columns: 1fr;
    gap: 8px;
    padding: 12px;
  }

  .mode-badge {
    padding: 10px;
  }

  .mode-info strong {
    font-size: 13px;
  }

  .mode-info small {
    font-size: 11px;
  }

  .action-section {
    width: 100%;
    justify-content: center;
  }

  .action-btn {
    flex: 1;
    min-width: 50px;
    font-size: 10px;
    padding: 5px 8px;
  }

  .main-status-bar {
    grid-template-columns: 1fr;
    gap: 12px;
    text-align: center;
  }

  .user-section {
    justify-self: center;
  }
}

/* 全局组件存在时的提示 */
.global-tetris-notice {
  background: linear-gradient(135deg, #d1ecf1, #bee5eb);
  border: 1px solid #b6d4db;
  border-radius: 8px;
  padding: 12px 16px;
  margin: 12px 0;
  color: #0c5460;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.global-tetris-notice::before {
  content: "ℹ️";
  font-size: 16px;
}

/* 改进响应式布局 */
@media (max-width: 480px) {
  .checkin-summary {
    grid-template-columns: 1fr;
  }

  .summary-item {
    padding: 8px;
  }

  .summary-value {
    font-size: 14px;
  }

  .daily-actions {
    flex-direction: column;
  }

  .action-btn {
    font-size: 10px;
    padding: 4px 6px;
  }
}

/* 👇 可拖动俄罗斯方块组件样式 */
.draggable-tetris-widget {
  position: fixed;
  z-index: 9999; /* 降低z-index，避免遮挡重要内容 */
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px; /* 减少圆角 */
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2); /* 减少阴影 */
  color: white;
  font-family: 'Arial', sans-serif;
  min-width: 140px; /* 减少最小宽度 */
  max-width: 160px; /* 减少最大宽度 */
  user-select: none;
  transition: box-shadow 0.3s ease;
  will-change: transform; /* 优化动画性能 */
  pointer-events: auto; /* 确保可以接收鼠标事件 */
  opacity: 0.95; /* 稍微透明，减少视觉干扰 */
}

.draggable-tetris-widget:hover {
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.4);
}

.draggable-tetris-widget.dragging {
  box-shadow: 0 16px 48px rgba(0, 0, 0, 0.5);
  transform: rotate(2deg);
  z-index: 10001;
}

.widget-header {
  background: rgba(255, 255, 255, 0.1);
  padding: 6px 8px; /* 减少内边距 */
  border-radius: 8px 8px 0 0; /* 调整圆角 */
  display: flex;
  align-items: center;
  justify-content: space-between;
  cursor: move;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  user-select: none; /* 防止文本选择 */
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
}

.widget-header:hover {
  background: rgba(255, 255, 255, 0.15);
}

.drag-handle {
  font-size: 12px;
  opacity: 0.7;
  font-weight: bold;
  letter-spacing: -1px;
  cursor: move;
  padding: 2px;
  user-select: none;
}

.widget-title {
  font-size: 13px;
  font-weight: 600;
  flex: 1;
  text-align: center;
}

.reset-position-btn {
  background: none;
  border: none;
  color: white;
  cursor: pointer;
  font-size: 12px;
  padding: 2px 4px;
  border-radius: 4px;
  opacity: 0.7;
  transition: all 0.3s ease;
  z-index: 10; /* 确保按钮在上层 */
  position: relative;
}

.reset-position-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  opacity: 1;
  transform: scale(1.1);
}

.widget-body {
  padding: 12px;
}

.widget-stats {
  margin-bottom: 12px;
}

.stat-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 4px 0;
  font-size: 12px;
}

.stat-label {
  opacity: 0.9;
}

.stat-value {
  font-weight: bold;
  background: rgba(255, 255, 255, 0.2);
  padding: 1px 4px; /* 减少内边距 */
  border-radius: 6px; /* 减少圆角 */
  min-width: 16px; /* 减少最小宽度 */
  text-align: center;
  font-size: 10px; /* 减少字体大小 */
}

.widget-actions {
  display: flex;
  gap: 4px; /* 减少间距 */
}

.widget-btn {
  flex: 1;
  border: none;
  color: white;
  padding: 4px 6px; /* 减少内边距 */
  border-radius: 4px; /* 减少圆角 */
  cursor: pointer;
  font-size: 9px; /* 减少字体大小 */
  font-weight: 500;
  transition: all 0.3s ease;
}

.widget-btn.primary {
  background: rgba(40, 167, 69, 0.8);
  animation: pulseGlow 2s infinite;
}

.widget-btn.secondary {
  background: rgba(255, 255, 255, 0.2);
}

.widget-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.widget-btn.primary:hover {
  background: rgba(40, 167, 69, 1);
}

.widget-btn.secondary:hover {
  background: rgba(255, 255, 255, 0.3);
}

@keyframes pulseGlow {
  0%, 100% {
    box-shadow: 0 0 0 0 rgba(40, 167, 69, 0.7);
  }
  50% {
    box-shadow: 0 0 0 8px rgba(40, 167, 69, 0);
  }
}

/* 👇 拖动时的全局样式 */
body.dragging {
  cursor: move !important;
}

body.dragging * {
  cursor: move !important;
}

/* 👇 移动端适配 */
@media (max-width: 768px) {
  .draggable-tetris-widget {
    min-width: 160px;
    max-width: 180px;
  }

  .widget-header {
    padding: 10px 12px;
    /* 增加触摸区域 */
  }

  .drag-handle {
    font-size: 14px;
  }

  .widget-title {
    font-size: 12px;
  }

  .stat-row {
    font-size: 11px;
  }

  .widget-btn {
    font-size: 10px;
    padding: 5px 8px;
  }

  /* 移动端拖动时的特殊样式 */
  .draggable-tetris-widget.dragging {
    transform: rotate(0deg) scale(1.05);
    transition: none;
  }

  /* 触摸设备的拖动提示 */
  .widget-header::before {
    content: '↕️ 拖动';
    position: absolute;
    top: -25px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.8);
    color: white;
    padding: 2px 6px;
    border-radius: 4px;
    font-size: 10px;
    opacity: 0;
    transition: opacity 0.3s ease;
  }

  .widget-header:active::before {
    opacity: 1;
  }
}

.calendar {
  position: relative;
  width: 100%;
  height: 100%;
  cursor: pointer;
  padding: 0;
  margin-top: 140px; /* 👈 调整顶部间距 */
}

/* 👇 新的状态栏样式 - 避免遮挡 */
.status-container {
  margin-bottom: 20px;
}

.main-status-bar {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 16px 20px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  display: grid;
  grid-template-columns: 1fr 1fr; /* 改为两列布局 */
  gap: 20px;
  align-items: center;
  font-family: 'Arial', sans-serif;
  backdrop-filter: blur(10px);
}

.user-section {
  display: flex;
  align-items: center;
  gap: 12px;
  justify-self: start;
}

.action-section {
  display: flex;
  gap: 8px;
  justify-self: end;
}

.action-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
  padding: 8px 12px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 500;
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.3);
  white-space: nowrap;
}

.action-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.user-badge {
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: bold;
}

.user-badge.lion {
  background: linear-gradient(45deg, #ffa500, #ff8c00);
}

.user-badge.bear {
  background: linear-gradient(45deg, #8b4513, #a0522d);
}

.checkin-status {
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 13px;
  font-weight: 500;
  background: rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
}

.checkin-status.checked-in {
  background: #28a745;
  color: white;
  box-shadow: 0 2px 8px rgba(40, 167, 69, 0.3);
}

.action-buttons {
  display: flex;
  gap: 8px;
}

.action-buttons button {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
  padding: 8px 12px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 500;
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.action-buttons button:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

/* 👇 签到面板样式 - 改为正常文档流 */
.checkin-panel {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  padding: 16px;
  border: 1px solid #e0e6ed;
  margin-bottom: 20px;
}

.checkin-summary {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 12px;
  margin-bottom: 16px;
}

.summary-item {
  text-align: center;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.summary-item:hover {
  background: #e9ecef;
  transform: translateY(-2px);
}

.summary-label {
  display: block;
  font-size: 12px;
  color: #666;
  margin-bottom: 4px;
}

.summary-value {
  display: block;
  font-size: 16px;
  font-weight: bold;
  color: #333;
}

.summary-value.success {
  color: #28a745;
}

.summary-value.tetris-highlight {
  color: #667eea;
}

.daily-actions {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.bonus-btn, .details-btn {
  background: linear-gradient(45deg, #28a745, #20c997);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.bonus-btn:hover, .details-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(40, 167, 69, 0.3);
}

/* 👇 增强的日期单元格样式 */
.date-cell {
  cursor: pointer;
  padding: 8px;
  border-radius: 6px;
  transition: all 0.3s ease;
  font-weight: 500;
  text-align: center;
  position: relative;
  overflow: hidden;
  min-height: 60px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.date-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  width: 100%;
}

.date-number {
  font-size: 14px;
  font-weight: 600;
}

.date-status {
  font-size: 16px;
  min-height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.check-mark {
  animation: checkIn 0.5s ease-in-out;
}

.today-mark {
  animation: pulse 2s infinite;
}

@keyframes checkIn {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    transform: scale(1.2);
    opacity: 0.8;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

/* 👇 不同状态的日期样式 */
.date-cell.is-today {
  background: linear-gradient(135deg, #fff3cd, #ffeaa7);
  border: 2px solid #ffb347;
  font-weight: bold;
}

.date-cell.is-checked {
  background: linear-gradient(135deg, #d4edda, #c3e6cb);
  border: 2px solid #28a745;
  color: #155724;
}

.date-cell.is-selected {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  transform: scale(1.05);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.5);
  border: 2px solid #667eea;
}

.date-cell.is-future {
  background: #f8f9fa;
  color: #6c757d;
  cursor: not-allowed;
  opacity: 0.6;
  border: 2px solid #dee2e6;
}

.date-cell.is-future:hover {
  background: #f8f9fa !important;
  transform: none !important;
  box-shadow: none !important;
}

.date-cell.is-clickable {
  cursor: pointer;
}

.future-mark {
  color: #dc3545;
  opacity: 0.7;
  font-size: 14px;
}

/* 👇 添加禁用状态的视觉反馈 */
.date-cell.is-future::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: repeating-linear-gradient(
      45deg,
      transparent,
      transparent 4px,
      rgba(220, 53, 69, 0.1) 4px,
      rgba(220, 53, 69, 0.1) 8px
  );
  border-radius: 6px;
  pointer-events: none;
}

/* 👇 日期状态说明的工具提示样式增强 */
.date-cell[title]:hover::after {
  content: attr(title);
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0, 0, 0, 0.9);
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 11px;
  white-space: nowrap;
  z-index: 1000;
  opacity: 0;
  animation: fadeInTooltip 0.3s ease-in-out 0.5s forwards;
}

@keyframes fadeInTooltip {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(4px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

/* 👇 用户特定的悬停效果 - 只对可点击日期生效 */
.lion-cell.is-clickable:hover:not(.is-selected):not(.is-checked):not(.is-future) {
  background: linear-gradient(135deg, #ffe4b5, #ffd700);
  border: 2px solid #ffa500;
  transform: scale(1.02);
}

.bear-cell.is-clickable:hover:not(.is-selected):not(.is-checked):not(.is-future) {
  background: linear-gradient(135deg, #deb887, #d2b48c);
  border: 2px solid #8b4513;
  transform: scale(1.02);
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(102, 126, 234, 0.7);
  }
  50% {
    transform: scale(1.05);
    box-shadow: 0 0 0 10px rgba(102, 126, 234, 0);
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.switch-btn {
  background: linear-gradient(45deg, #ff6b6b, #ffa500) !important;
}

.stats-btn {
  background: linear-gradient(45deg, #4ecdc4, #44a08d) !important;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(102, 126, 234, 0.7);
  }
  50% {
    transform: scale(1.05);
    box-shadow: 0 0 0 10px rgba(102, 126, 234, 0);
  }
}

/* 👇 统计信息样式保持不变 */
.tetris-stats {
  margin: 20px 0;
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  border: 1px solid #e0e6ed;
}

.tetris-stats h4 {
  margin: 0 0 16px 0;
  color: #333;
  font-size: 18px;
  font-weight: 600;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 12px;
}

.stat-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  border-radius: 8px;
  border-left: 4px solid #667eea;
  transition: all 0.3s ease;
}

.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.stat-label {
  font-size: 14px;
  color: #666;
  font-weight: 500;
}

.stat-value {
  font-size: 18px;
  font-weight: bold;
  color: #667eea;
}

/* 👇 用户指南样式 */
.user-guide {
  background: white;
  border-radius: 12px;
  padding: 20px;
  margin: 20px 0;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  border-left: 4px solid #667eea;
}

.user-guide h4 {
  margin: 0 0 16px 0;
  color: #333;
  font-size: 18px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.guide-content {
  display: grid;
  gap: 12px;
}

.guide-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.guide-item:hover {
  background: #e9ecef;
  transform: translateX(4px);
}

.guide-icon {
  font-size: 20px;
  flex-shrink: 0;
  width: 24px;
  text-align: center;
}

.guide-text {
  flex: 1;
  font-size: 14px;
  line-height: 1.5;
}

.guide-text strong {
  color: #667eea;
  font-weight: 600;
}

.current-user {
  color: #28a745;
  font-weight: 600;
  background: rgba(40, 167, 69, 0.1);
  padding: 2px 6px;
  border-radius: 4px;
}

/* 👇 移动端完整适配 */
@media (max-width: 768px) {
  .main-status-bar {
    grid-template-columns: 1fr 1fr; /* 改为两列布局 */
    gap: 20px;
  }

  .user-section,
  .tetris-section,
  .action-section {
    justify-self: center;
  }

  .action-section {
    width: 100%;
    justify-content: center;
    flex-wrap: wrap;
  }

  .action-btn {
    flex: 1;
    min-width: 80px;
  }

  .checkin-panel {
    margin: 16px 0;
  }

  .checkin-summary {
    grid-template-columns: repeat(2, 1fr);
  }

  .calendar {
    margin-top: 20px;
  }

  .guide-content {
    gap: 8px;
  }

  .guide-item {
    padding: 10px;
  }

  .guide-text {
    font-size: 13px;
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }

  .user-badge {
    font-size: 12px;
    padding: 4px 8px;
  }

  .checkin-status {
    font-size: 11px;
    padding: 3px 8px;
  }

  .summary-value {
    font-size: 14px;
  }

  .summary-label {
    font-size: 11px;
  }

  .date-content {
    gap: 2px;
  }

  .date-number {
    font-size: 12px;
  }

  .date-status {
    font-size: 14px;
    min-height: 16px;
  }

  .tetris-section {
    background: rgba(255, 255, 255, 0.2);
    padding: 6px 12px;
  }

  .tetris-count {
    font-size: 16px;
    padding: 2px 8px;
  }

  .tetris-label {
    font-size: 12px;
  }

  /* 移动端时隐藏拖动组件上的某些元素 */
  .draggable-tetris-widget {
    z-index: 10000;
  }

  .widget-header {
    touch-action: none; /* 防止移动端滚动干扰拖动 */
  }
}

/* 👇 辅助样式 */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* 👇 打印样式 */
@media print {
  .status-container,
  .checkin-panel,
  .user-guide {
    display: none;
  }

  .calendar {
    margin-top: 0;
  }
}
</style>