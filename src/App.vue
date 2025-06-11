<template>
  <div id="main-container">
    <div class="container">
      <div class="header-section">
        <div class="title-wrapper">
          <div class="main-title">動的要素デモサイト</div>
          <div class="sub-title">{{ currentTime }}</div>
        </div>
      </div>

      <div class="control-panel">
        <div class="button-group">
          <div class="control-button" @click="addRandomBox">ボックス追加</div>
          <div class="control-button" @click="shuffleColors">色をシャッフル</div>
          <div class="control-button" @click="toggleAnimation">アニメーション切替</div>
          <div class="control-button" @click="generateRandomData">データ生成</div>
        </div>
      </div>

      <div class="dynamic-section">
        <div class="dynamic-grid">
          <div v-for="(box, index) in colorBoxes" :key="`box-${index}-${box.id}`"
               class="item-box color-box"
               :style="{ backgroundColor: box.color }"
               @click="handleBoxClick(index)">
            <div class="box-content">
              <div class="box-number">{{ box.value }}</div>
              <div class="box-label">Box {{ index + 1 }}</div>
            </div>
          </div>
        </div>
      </div>

      <div class="data-section">
        <div class="data-grid">
          <div v-for="item in randomData" :key="`data-${item.id}`" class="data-item">
            <div class="data-header">
              <div class="data-title">{{ item.title }}</div>
              <div class="data-value">{{ item.value }}</div>
            </div>
            <div class="data-body">
              <div v-for="(tag, tagIndex) in item.tags" :key="`tag-${tagIndex}`" class="random-item">
                {{ tag }}
              </div>
            </div>
            <div class="data-footer">
              <div class="timestamp">{{ item.timestamp }}</div>
            </div>
          </div>
        </div>
      </div>

      <div class="interactive-area">
        <div class="click-zone" @mousemove="handleMouseMove">
          <div v-for="particle in particles" :key="`particle-${particle.id}`"
               class="particle"
               :style="{
                 left: particle.x + 'px',
                 top: particle.y + 'px',
                 backgroundColor: particle.color,
                 width: particle.size + 'px',
                 height: particle.size + 'px'
               }">
          </div>
        </div>
      </div>

      <div class="status-section">
        <div class="status-grid">
          <div v-for="n in 12" :key="`status-${n}`" class="status-item">
            <div class="status-indicator" :class="{ active: activeStatuses.includes(n) }"></div>
            <div class="status-label">Status {{ n }}</div>
          </div>
        </div>
      </div>

      <div v-if="showLoader" class="loader-overlay">
        <div class="loader"></div>
        <div class="loader"></div>
        <div class="loader"></div>
      </div>
    </div>

    <div v-for="(element, index) in floatingElements" :key="`float-${index}`"
         class="floating-element"
         :style="{
           top: element.y + 'px',
           left: element.x + 'px',
           backgroundColor: element.color
         }"
         @click="removeFloatingElement(index)">
      {{ element.text }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      currentTime: '',
      colorBoxes: [],
      randomData: [],
      particles: [],
      floatingElements: [],
      activeStatuses: [],
      showLoader: false,
      animationActive: true,
      colors: ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7', '#DDA0DD', '#F4A460', '#98D8C8'],
      mouseX: 0,
      mouseY: 0
    }
  },
  mounted() {
    this.initializeApp()
    this.startTimeUpdate()
    this.startRandomUpdates()
  },
  methods: {
    initializeApp() {
      this.generateColorBoxes(8)
      this.generateRandomData()
      this.updateActiveStatuses()
    },
    generateColorBoxes(count) {
      this.colorBoxes = Array.from({ length: count }, (_, i) => ({
        id: Math.random(),
        color: this.colors[i % this.colors.length],
        value: Math.floor(Math.random() * 100)
      }))
    },
    addRandomBox() {
      const newBox = {
        id: Math.random(),
        color: this.colors[Math.floor(Math.random() * this.colors.length)],
        value: Math.floor(Math.random() * 100)
      }
      this.colorBoxes.push(newBox)
      this.createFloatingElement('新しいボックス追加！')
    },
    shuffleColors() {
      this.colorBoxes = this.colorBoxes.map(box => ({
        ...box,
        color: this.colors[Math.floor(Math.random() * this.colors.length)]
      }))
    },
    toggleAnimation() {
      this.animationActive = !this.animationActive
      this.showLoader = true
      setTimeout(() => {
        this.showLoader = false
      }, 1000)
    },
    generateRandomData() {
      const titles = ['アイテム', 'データ', '要素', 'コンテンツ', 'オブジェクト']
      const tags = ['タグA', 'タグB', 'タグC', 'タグD', 'タグE', 'タグF']
      
      this.randomData = Array.from({ length: 6 }, () => ({
        id: Math.random(),
        title: titles[Math.floor(Math.random() * titles.length)] + Math.floor(Math.random() * 100),
        value: Math.floor(Math.random() * 1000),
        tags: Array.from({ length: Math.floor(Math.random() * 4) + 1 }, () => 
          tags[Math.floor(Math.random() * tags.length)]
        ),
        timestamp: new Date().toLocaleTimeString()
      }))
    },
    handleBoxClick(index) {
      const box = this.colorBoxes[index]
      box.value = Math.floor(Math.random() * 100)
      this.createParticles(event.clientX, event.clientY)
    },
    handleMouseMove(event) {
      this.mouseX = event.clientX
      this.mouseY = event.clientY
      
      if (Math.random() > 0.95 && this.animationActive) {
        this.createParticles(event.clientX, event.clientY)
      }
    },
    createParticles(x, y) {
      const newParticles = Array.from({ length: 5 }, () => ({
        id: Math.random(),
        x: x + (Math.random() - 0.5) * 50,
        y: y + (Math.random() - 0.5) * 50,
        color: this.colors[Math.floor(Math.random() * this.colors.length)],
        size: Math.random() * 20 + 5
      }))
      
      this.particles.push(...newParticles)
      
      setTimeout(() => {
        this.particles = this.particles.slice(-20)
      }, 1000)
    },
    createFloatingElement(text) {
      const element = {
        text: text || `要素 ${this.floatingElements.length + 1}`,
        x: Math.random() * (window.innerWidth - 200),
        y: Math.random() * (window.innerHeight - 100),
        color: this.colors[Math.floor(Math.random() * this.colors.length)]
      }
      this.floatingElements.push(element)
    },
    removeFloatingElement(index) {
      this.floatingElements.splice(index, 1)
    },
    updateActiveStatuses() {
      const count = Math.floor(Math.random() * 6) + 1
      this.activeStatuses = Array.from({ length: count }, () => 
        Math.floor(Math.random() * 12) + 1
      )
    },
    startTimeUpdate() {
      setInterval(() => {
        this.currentTime = new Date().toLocaleTimeString()
      }, 1000)
    },
    startRandomUpdates() {
      setInterval(() => {
        if (this.animationActive) {
          this.updateActiveStatuses()
          
          if (Math.random() > 0.7) {
            const randomIndex = Math.floor(Math.random() * this.colorBoxes.length)
            if (this.colorBoxes[randomIndex]) {
              this.colorBoxes[randomIndex].value = Math.floor(Math.random() * 100)
            }
          }
        }
      }, 2000)
    }
  }
}
</script>

<style scoped>
#main-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.header-section {
  text-align: center;
  padding: 40px 0;
}

.title-wrapper {
  display: inline-block;
}

.main-title {
  font-size: 36px;
  font-weight: bold;
  margin-bottom: 10px;
}

.sub-title {
  font-size: 18px;
  color: #666;
}

.control-panel {
  margin: 30px 0;
}

.button-group {
  display: flex;
  justify-content: center;
  gap: 15px;
  flex-wrap: wrap;
}

.control-button {
  padding: 10px 20px;
  background: #3498db;
  color: white;
  border-radius: 5px;
  cursor: pointer;
  transition: background 0.3s;
}

.control-button:hover {
  background: #2980b9;
}

.data-section {
  margin: 40px 0;
}

.data-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.data-item {
  background: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.data-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.data-title {
  font-weight: bold;
  font-size: 18px;
}

.data-value {
  color: #3498db;
  font-size: 24px;
  font-weight: bold;
}

.data-body {
  margin: 15px 0;
}

.data-footer {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #eee;
}

.timestamp {
  font-size: 12px;
  color: #999;
}

.interactive-area {
  position: relative;
  height: 300px;
  background: rgba(255,255,255,0.5);
  border-radius: 10px;
  margin: 30px 0;
  overflow: hidden;
}

.click-zone {
  width: 100%;
  height: 100%;
  position: relative;
}

.particle {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  animation: fadeOut 1s ease-out forwards;
}

@keyframes fadeOut {
  to {
    opacity: 0;
    transform: scale(0);
  }
}

.status-section {
  margin: 40px 0;
}

.status-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 15px;
}

.status-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: white;
  border-radius: 5px;
}

.status-indicator {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #ddd;
  transition: background 0.3s;
}

.status-indicator.active {
  background: #2ecc71;
  box-shadow: 0 0 10px rgba(46,204,113,0.5);
}

.status-label {
  font-size: 14px;
}

.loader-overlay {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  gap: 10px;
  background: rgba(255,255,255,0.9);
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 5px 20px rgba(0,0,0,0.2);
}

.box-content {
  text-align: center;
}

.box-number {
  font-size: 24px;
  margin-bottom: 5px;
}

.box-label {
  font-size: 14px;
}
</style>