<template>
  <div class="pacman-game">
    <canvas ref="gameCanvas" :width="canvasWidth" :height="canvasHeight"></canvas>
    <div class="score">Score: {{ score }}</div>
    <Ghost v-for="ghost in ghosts" 
          :key="ghost.type" 
          :type="ghost.type"
          :ctx="ctx"
          :pacman="pacman"
          :blinkyPos="getBlinkyPosition()"
          :frightened="powerMode"
          ref="ghostRefs" />
  </div>
</template>

<script>
import Ghost from './Ghost.vue'

export default {
  name: 'PacmanGame',
  components: {
    Ghost
  },
  data() {
    return {
      canvasWidth: 448,
      canvasHeight: 496,
      ctx: null,
      pacman: {
        x: 224,
        y: 384,
        direction: 0,
        nextDirection: 0,
        speed: 2,
        mouthOpen: 0
      },
      score: 0,
      gameLoop: null,
      powerMode: false,
      powerModeTimer: null,
      ghosts: [
        { type: 'blinky' },
        { type: 'pinky' },
        { type: 'inky' },
        { type: 'clyde' }
      ],
      gameIsOver: false,
      map: [
        [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
        [1,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,1],
        [1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,0,1],
        [1,2,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,2,1],
        [1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,0,1],
        [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
        [1,0,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,0,1],
        [1,0,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,0,1],
        [1,0,0,0,0,0,0,1,1,0,0,0,0,1,1,0,0,0,0,1,1,0,0,0,0,0,0,1],
        [1,1,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,1,1],
        [1,1,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,1,1],
        [1,1,1,1,1,1,0,1,1,0,0,0,0,0,0,0,0,0,0,1,1,0,1,1,1,1,1,1],
        [1,1,1,1,1,1,0,1,1,0,1,1,1,3,3,1,1,1,0,1,1,0,1,1,1,1,1,1],
        [1,1,1,1,1,1,0,1,1,0,1,0,0,0,0,0,0,1,0,1,1,0,1,1,1,1,1,1],
        [0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0],
        [1,1,1,1,1,1,0,1,1,0,1,0,0,0,0,0,0,1,0,1,1,0,1,1,1,1,1,1],
        [1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1],
        [1,1,1,1,1,1,0,1,1,0,0,0,0,0,0,0,0,0,0,1,1,0,1,1,1,1,1,1],
        [1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1],
        [1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1],
        [1,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,1],
        [1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,0,1],
        [1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,0,1],
        [1,2,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,2,1],
        [1,1,1,0,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,0,1,1,1],
        [1,1,1,0,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,0,1,1,1],
        [1,0,0,0,0,0,0,1,1,0,0,0,0,1,1,0,0,0,0,1,1,0,0,0,0,0,0,1],
        [1,0,1,1,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,1,1,0,1],
        [1,0,1,1,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,1,1,0,1],
        [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
        [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
      ]
    }
  },
  mounted() {
    this.ctx = this.gameCanvas.getContext('2d')
    this.setupGame()
    this.startGameLoop()
    window.addEventListener('keydown', this.handleKeyPress)
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeyPress)
    if (this.gameLoop) {
      cancelAnimationFrame(this.gameLoop)
    }
    if (this.powerModeTimer) {
      clearTimeout(this.powerModeTimer)
    }
  },
  methods: {
    setupGame() {
      this.drawMap()
      this.drawPacman()
    },
    startGameLoop() {
      const loop = () => {
        if (!this.gameIsOver) {
          this.update()
          this.draw()
          this.gameLoop = requestAnimationFrame(loop)
        }
      }
      this.gameLoop = requestAnimationFrame(loop)
    },
    update() {
      this.updatePacman()
      this.$refs.ghostRefs?.forEach(ghost => ghost.update(this.map))
      this.checkCollisions()
    },
    draw() {
      this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
      this.drawMap()
      this.drawPacman()
      this.$refs.ghostRefs?.forEach(ghost => ghost.draw())
    },
    drawMap() {
      for (let y = 0; y < this.map.length; y++) {
        for (let x = 0; x < this.map[y].length; x++) {
          if (this.map[y][x] === 1) {
            this.ctx.fillStyle = '#2121ff'
            this.ctx.fillRect(x * 16, y * 16, 16, 16)
          } else if (this.map[y][x] === 0) {
            this.ctx.fillStyle = '#ffb8ae'
            this.ctx.beginPath()
            this.ctx.arc(x * 16 + 8, y * 16 + 8, 2, 0, Math.PI * 2)
            this.ctx.fill()
          } else if (this.map[y][x] === 2) {
            this.ctx.fillStyle = '#ffb8ae'
            this.ctx.beginPath()
            this.ctx.arc(x * 16 + 8, y * 16 + 8, 6, 0, Math.PI * 2)
            this.ctx.fill()
          }
        }
      }
    },
    drawPacman() {
      this.ctx.save()
      this.ctx.translate(this.pacman.x, this.pacman.y)
      this.ctx.rotate((this.pacman.direction * 90) * Math.PI / 180)
      
      this.ctx.fillStyle = '#ffff00'
      this.ctx.beginPath()
      this.ctx.arc(0, 0, 13, (0.25 + this.pacman.mouthOpen * 0.25) * Math.PI, (1.75 - this.pacman.mouthOpen * 0.25) * Math.PI)
      this.ctx.lineTo(0, 0)
      this.ctx.fill()
      
      this.ctx.restore()
    },
    updatePacman() {
      const nextX = Math.floor((this.pacman.x + (this.pacman.direction === 1 ? 16 : this.pacman.direction === 3 ? -16 : 0)) / 16)
      const nextY = Math.floor((this.pacman.y + (this.pacman.direction === 2 ? 16 : this.pacman.direction === 0 ? -16 : 0)) / 16)
      
      if (this.canMove(nextX, nextY)) {
        switch(this.pacman.direction) {
          case 0: this.pacman.y -= this.pacman.speed; break
          case 1: this.pacman.x += this.pacman.speed; break
          case 2: this.pacman.y += this.pacman.speed; break
          case 3: this.pacman.x -= this.pacman.speed; break
        }
      }
      
      // Handle tunnel
      if (this.pacman.x < -10) this.pacman.x = this.canvasWidth + 10
      if (this.pacman.x > this.canvasWidth + 10) this.pacman.x = -10
      
      this.pacman.mouthOpen = Math.sin(Date.now() / 100) / 2 + 0.5
    },
    canMove(x, y) {
      return x >= 0 && x < 28 && y >= 0 && y < 31 && this.map[y][x] !== 1
    },
    checkCollisions() {
      const gridX = Math.floor(this.pacman.x / 16)
      const gridY = Math.floor(this.pacman.y / 16)
      
      if (this.map[gridY][gridX] === 0) {
        this.map[gridY][gridX] = 3
        this.score += 10
      } else if (this.map[gridY][gridX] === 2) {
        this.map[gridY][gridX] = 3
        this.score += 50
        this.activatePowerMode()
      }

      // Check ghost collisions with improved precision
      this.$refs.ghostRefs?.forEach(ghost => {
        const dx = this.pacman.x - ghost.x
        const dy = this.pacman.y - ghost.y
        const distance = Math.sqrt(dx * dx + dy * dy)
        
        if (distance < 16) { // Reduced collision distance for more precise detection
          if (this.powerMode) {
            this.score += 200
            ghost.resetPosition()
          } else {
            this.gameOver()
          }
        }
      })
    },
    handleKeyPress(e) {
      switch(e.key) {
        case 'ArrowUp': this.pacman.direction = 0; break
        case 'ArrowRight': this.pacman.direction = 1; break
        case 'ArrowDown': this.pacman.direction = 2; break
        case 'ArrowLeft': this.pacman.direction = 3; break
      }
    },
    activatePowerMode() {
      this.powerMode = true
      if (this.powerModeTimer) clearTimeout(this.powerModeTimer)
      this.powerModeTimer = setTimeout(() => {
        this.powerMode = false
      }, 10000)
    },
    gameOver() {
      this.gameIsOver = true
      cancelAnimationFrame(this.gameLoop)
      this.ctx.fillStyle = 'rgba(0, 0, 0, 0.75)'
      this.ctx.fillRect(0, 0, this.canvasWidth, this.canvasHeight)
      this.ctx.fillStyle = 'white'
      this.ctx.font = '48px Arial'
      this.ctx.textAlign = 'center'
      this.ctx.fillText('Game Over', this.canvasWidth / 2, this.canvasHeight / 2)
    },
    getBlinkyPosition() {
      const blinkyRef = this.$refs.ghostRefs?.find(ghost => ghost.type === 'blinky')
      return blinkyRef ? { x: blinkyRef.x, y: blinkyRef.y } : null
    }
  },
  computed: {
    gameCanvas() {
      return this.$refs.gameCanvas
    }
  }
}
</script>

<style scoped>
.pacman-game {
  position: relative;
}
.score {
  position: absolute;
  top: -40px;
  left: 0;
  color: white;
  font-family: Arial, sans-serif;
  font-size: 24px;
}
</style>