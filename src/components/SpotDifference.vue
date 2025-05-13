<template>
  <div class="spot-difference-container">
    <el-card class="game-header">
      <h1>全民找茬游戏</h1>
      <div class="score-timer">
        <el-tag size="large" type="success">得分: {{ score }}</el-tag>
        <el-tag size="large" type="warning">时间: {{ formatTime(timer) }}</el-tag>
        <el-tag size="large" type="info">差异数量: {{ currentLevel.differences.length }}</el-tag>
      </div>
    </el-card>

    <div class="game-container">
      <div class="image-container">
        <div class="image left-image" ref="leftImageRef" @click="handleClick($event, 'left')">
          <el-image :src="currentLevel.leftImage" fit="contain" />
          <div v-for="(mark, index) in leftMarks" 
               :key="'left-' + index" 
               class="mark" 
               :style="{ left: mark.x + 'px', top: mark.y + 'px' }">
            <el-icon><CircleCheck /></el-icon>
          </div>
        </div>
        <div class="image right-image" ref="rightImageRef" @click="handleClick($event, 'right')">
          <el-image :src="currentLevel.rightImage" fit="contain" />
          <div v-for="(mark, index) in rightMarks" 
               :key="'right-' + index" 
               class="mark" 
               :style="{ left: mark.x + 'px', top: mark.y + 'px' }">
            <el-icon><CircleCheck /></el-icon>
          </div>
        </div>
      </div>
      <div class="control-buttons">
        <el-button type="primary" @click="switchImages" :disabled="isSwitching">
          切换图片
        </el-button>
        <el-button type="warning" @click="showAnswer" :disabled="isShowingAnswer">
          显示答案
        </el-button>
      </div>
    </div>

    <el-dialog
      v-model="gameOverDialogVisible"
      title="游戏结束"
      width="30%"
      center>
      <span>恭喜你完成本关！最终得分：{{ score }}</span>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="restartGame">重新开始</el-button>
          <el-button type="primary" @click="nextLevel">
            下一关
          </el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { ElMessage } from 'element-plus'
import { CircleCheck } from '@element-plus/icons-vue'

// 游戏数据
const levels = [
  {
    id: 1,
    leftImage: '/images/scene1a.svg',
    rightImage: '/images/scene1b.svg',
    differences: [
      { x1: 240, y1: 220, x2: 240, y2: 220, radius: 30 },
      { x1: 110, y1: 150, x2: 110, y2: 150, radius: 30 },
      { x1: 500, y1: 80, x2: 480, y2: 80, radius: 30 }
    ]
  },
  {
    id: 2,
    leftImage: '/images/scene2a.svg',
    rightImage: '/images/scene2b.svg',
    differences: [
      { x1: 110, y1: 200, x2: 110, y2: 200, radius: 30 },
      { x1: 500, y1: 80, x2: 500, y2: 80, radius: 30 },
      { x1: 432, y1: 250, x2: 437, y2: 250, radius: 30 },
      { x1: 200, y1: 260, x2: 200, y2: 260, radius: 30 }
    ]
  },
  {
    id: 3,
    leftImage: '/images/scene3a.svg',
    rightImage: '/images/scene3b.svg',
    differences: [
      { x1: 120, y1: 320, x2: 120, y2: 320, radius: 30 },
      { x1: 150, y1: 280, x2: 150, y2: 280, radius: 30 },
      { x1: 415, y1: 120, x2: 415, y2: 120, radius: 30 },
      { x1: 240, y1: 320, x2: 240, y2: 320, radius: 30 },
      { x1: 520, y1: 150, x2: 520, y2: 150, radius: 30 }
    ]
  }
]

// 响应式状态
const currentLevelIndex = ref(0)
const currentLevel = ref(levels[0])
const score = ref(0)
const timer = ref(0)
const gameOverDialogVisible = ref(false)
const leftMarks = ref([])
const rightMarks = ref([])
const leftImageRef = ref(null)
const rightImageRef = ref(null)
const isSwitching = ref(false)
const isShowingAnswer = ref(false)

let timerInterval

// 开始计时器
const startTimer = () => {
  timerInterval = setInterval(() => {
    timer.value++
  }, 1000)
}

// 格式化时间
const formatTime = (seconds) => {
  return `${seconds}s`
}

// 处理点击事件
const handleClick = (event, side) => {
  if (gameOverDialogVisible.value || isShowingAnswer.value) return

  const rect = event.target.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top

  // 检查是否已经找到过这个差异点
  const alreadyFound = leftMarks.value.some(mark => 
    Math.sqrt(Math.pow(mark.x - x, 2) + Math.pow(mark.y - y, 2)) <= 30
  )

  if (alreadyFound) {
    ElMessage.warning('这个差异点已经被找到了！')
    return
  }

  // 检查是否点击到差异点
  const differenceIndex = currentLevel.value.differences.findIndex(diff => {
    const compareX = side === 'left' ? diff.x1 : diff.x2
    const compareY = side === 'left' ? diff.y1 : diff.y2
    
    return Math.sqrt(
      Math.pow(x - compareX, 2) + 
      Math.pow(y - compareY, 2)
    ) <= 30
  })

  if (differenceIndex !== -1) {
    const difference = currentLevel.value.differences[differenceIndex]
    // 标记找到的差异点
    if (side === 'left') {
      leftMarks.value.push({ x: difference.x1, y: difference.y1 })
      rightMarks.value.push({ x: difference.x2, y: difference.y2 })
    } else {
      leftMarks.value.push({ x: difference.x1, y: difference.y1 })
      rightMarks.value.push({ x: difference.x2, y: difference.y2 })
    }
    
    score.value += 10
    const remainingDifferences = currentLevel.value.differences.length - leftMarks.value.length
    ElMessage.success(`找到一处差异！还剩${remainingDifferences}处`)
    
    // 检查是否完成所有差异点
    if (leftMarks.value.length === currentLevel.value.differences.length) {
      clearInterval(timerInterval)
      gameOverDialogVisible.value = true
    }
  } else {
    score.value = Math.max(0, score.value - 5)
    ElMessage.error('这里没有差异哦！')
  }
}

// 重新开始游戏
const restartGame = () => {
  score.value = 0
  timer.value = 0
  leftMarks.value = []
  rightMarks.value = []
  gameOverDialogVisible.value = false
  clearInterval(timerInterval)
  startTimer()
}

// 下一关
const nextLevel = () => {
  if (currentLevelIndex.value < levels.length - 1) {
    currentLevelIndex.value++
    currentLevel.value = levels[currentLevelIndex.value]
    leftMarks.value = []
    rightMarks.value = []
    gameOverDialogVisible.value = false
    clearInterval(timerInterval)
    startTimer()
  } else {
    ElMessage.success('恭喜你完成所有关卡！')
    gameOverDialogVisible.value = false
  }
}

// 切换图片
const switchImages = () => {
  if (isSwitching.value) return
  
  isSwitching.value = true
  currentLevelIndex.value = (currentLevelIndex.value + 1) % levels.length
  currentLevel.value = levels[currentLevelIndex.value]
  leftMarks.value = []
  rightMarks.value = []
  score.value = 0
  timer.value = 0
  clearInterval(timerInterval)
  startTimer()
  
  // 添加一个短暂的延迟，防止快速点击
  setTimeout(() => {
    isSwitching.value = false
  }, 500)
}

// 显示答案
const showAnswer = () => {
  if (isShowingAnswer.value) return
  
  isShowingAnswer.value = true
  // 清除现有的标记
  leftMarks.value = []
  rightMarks.value = []
  
  // 显示所有差异点
  currentLevel.value.differences.forEach(diff => {
    leftMarks.value.push({ x: diff.x1, y: diff.y1 })
    rightMarks.value.push({ x: diff.x2, y: diff.y2 })
  })
  
  // 添加一个短暂的延迟，防止快速点击
  setTimeout(() => {
    isShowingAnswer.value = false
  }, 500)
}

// 生命周期钩子
onMounted(() => {
  startTimer()
})

onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval)
  }
})
</script>

<style scoped>
.spot-difference-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.game-header {
  margin-bottom: 20px;
  text-align: center;
}

.score-timer {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-top: 10px;
}

.game-container {
  background: #f5f7fa;
  border-radius: 8px;
  padding: 20px;
}

.image-container {
  display: flex;
  justify-content: space-between;
  gap: 20px;
}

.image {
  position: relative;
  width: 48%;
  height: 400px;
  border: 2px solid #dcdfe6;
  border-radius: 4px;
  overflow: hidden;
  cursor: pointer;
  background: #fff;
}

.image :deep(.el-image) {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.image:hover {
  border-color: #409eff;
}

.mark {
  position: absolute;
  transform: translate(-50%, -50%);
  color: #67c23a;
  font-size: 24px;
  pointer-events: none;
  z-index: 1;
}

.dialog-footer {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 20px;
}

.control-buttons {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  gap: 10px;
}
</style> 