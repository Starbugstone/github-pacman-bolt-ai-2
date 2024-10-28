<template>
  <canvas ref="ghostCanvas" :width="32" :height="32" style="display: none;"></canvas>
</template>

<script>
export default {
  name: 'Ghost',
  props: {
    type: {
      type: String,
      required: true
    },
    ctx: {
      required: true
    },
    pacman: {
      type: Object,
      required: true
    },
    blinkyPos: {
      type: Object,
      default: null
    },
    frightened: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      x: 224,
      y: 232,
      direction: 2,
      speed: 1.5,
      ghostCanvas: null,
      ghostCtx: null,
      scatter: false,
      scatterTimer: null,
      colors: {
        blinky: '#FF0000',
        pinky: '#FFB8FF',
        inky: '#00FFFF',
        clyde: '#FFB851'
      },
      scatterTargets: {
        blinky: { x: 27, y: 0 },
        pinky: { x: 0, y: 0 },
        inky: { x: 27, y: 31 },
        clyde: { x: 0, y: 31 }
      },
      startPosition: { x: 224, y: 232 }
    }
  },
  mounted() {
    this.ghostCanvas = this.$refs.ghostCanvas
    this.ghostCtx = this.ghostCanvas.getContext('2d')
    this.startScatterMode()
    this.setInitialPosition()
  },
  methods: {
    setInitialPosition() {
      // Set different starting positions for each ghost
      switch(this.type) {
        case 'blinky':
          this.startPosition = { x: 224, y: 232 }
          break
        case 'pinky':
          this.startPosition = { x: 224, y: 232 }
          break
        case 'inky':
          this.startPosition = { x: 192, y: 232 }
          break
        case 'clyde':
          this.startPosition = { x: 256, y: 232 }
          break
      }
      this.resetPosition()
    },
    resetPosition() {
      this.x = this.startPosition.x
      this.y = this.startPosition.y
      this.direction = 2
    },
    startScatterMode() {
      this.scatterTimer = setInterval(() => {
        this.scatter = !this.scatter
      }, 7000)
    },
    update(map) {
      if (this.frightened) {
        this.updateFrightened(map)
        return
      }

      const target = this.getTarget()
      const possibleDirections = this.getPossibleDirections(map)
      
      if (possibleDirections.length > 0) {
        this.direction = this.getBestDirection(possibleDirections, target)
        this.move()
      }
    },
    updateFrightened(map) {
      const possibleDirections = this.getPossibleDirections(map)
      if (possibleDirections.length === 0) return
      
      // In frightened mode, choose a random valid direction
      const randomIndex = Math.floor(Math.random() * possibleDirections.length)
      this.direction = possibleDirections[randomIndex]
      
      // Move at reduced speed when frightened
      this.move(0.5)
    },
    getTarget() {
      if (this.scatter) {
        return this.scatterTargets[this.type]
      }

      const pacmanX = Math.floor(this.pacman.x / 16)
      const pacmanY = Math.floor(this.pacman.y / 16)

      switch (this.type) {
        case 'blinky':
          return { x: pacmanX, y: pacmanY }
        case 'pinky':
          return this.getPinkyTarget()
        case 'inky':
          return this.getInkyTarget()
        case 'clyde':
          return this.getClydeTarget()
        default:
          return { x: pacmanX, y: pacmanY }
      }
    },
    getPinkyTarget() {
      const pacmanX = Math.floor(this.pacman.x / 16)
      const pacmanY = Math.floor(this.pacman.y / 16)
      const offset = 4
      
      let targetX = pacmanX
      let targetY = pacmanY

      switch(this.pacman.direction) {
        case 0: // Up
          targetY -= offset
          targetX -= offset // Add the original Pac-Man bug
          break
        case 1: // Right
          targetX += offset
          break
        case 2: // Down
          targetY += offset
          break
        case 3: // Left
          targetX -= offset
          break
      }

      return { x: targetX, y: targetY }
    },
    getInkyTarget() {
      if (!this.blinkyPos) return this.getPinkyTarget()
      
      const pacmanX = Math.floor(this.pacman.x / 16)
      const pacmanY = Math.floor(this.pacman.y / 16)
      const blinkyX = Math.floor(this.blinkyPos.x / 16)
      const blinkyY = Math.floor(this.blinkyPos.y / 16)
      
      // Get position 2 tiles ahead of Pacman
      let pivotX = pacmanX
      let pivotY = pacmanY
      
      switch(this.pacman.direction) {
        case 0: // Up
          pivotY -= 2
          pivotX -= 2 // Add the original Pac-Man bug
          break
        case 1: // Right
          pivotX += 2
          break
        case 2: // Down
          pivotY += 2
          break
        case 3: // Left
          pivotX -= 2
          break
      }
      
      // Double the vector from Blinky to pivot point
      return {
        x: pivotX + (pivotX - blinkyX),
        y: pivotY + (pivotY - blinkyY)
      }
    },
    getClydeTarget() {
      const pacmanX = Math.floor(this.pacman.x / 16)
      const pacmanY = Math.floor(this.pacman.y / 16)
      const ghostX = Math.floor(this.x / 16)
      const ghostY = Math.floor(this.y / 16)
      
      const distance = Math.sqrt(
        Math.pow(ghostX - pacmanX, 2) + 
        Math.pow(ghostY - pacmanY, 2)
      )
      
      // If Clyde is more than 8 tiles away from Pacman, target Pacman
      // Otherwise, target his scatter corner
      return distance > 8 ? 
        { x: pacmanX, y: pacmanY } : 
        this.scatterTargets.clyde
    },
    getPossibleDirections(map) {
      const directions = []
      const x = Math.floor(this.x / 16)
      const y = Math.floor(this.y / 16)
      
      // Prevent reversing direction unless no other option
      const oppositeDirection = (this.direction + 2) % 4
      
      // Check each direction
      if (y > 0 && map[y-1][x] !== 1) directions.push(0)  // Up
      if (x < map[0].length-1 && map[y][x+1] !== 1) directions.push(1)  // Right
      if (y < map.length-1 && map[y+1][x] !== 1) directions.push(2)  // Down
      if (x > 0 && map[y][x-1] !== 1) directions.push(3)  // Left
      
      // If there are options other than reversing, remove the opposite direction
      if (directions.length > 1) {
        const index = directions.indexOf(oppositeDirection)
        if (index !== -1) {
          directions.splice(index, 1)
        }
      }
      
      return directions
    },
    getBestDirection(possibleDirections, target) {
      let bestDirection = this.direction
      let shortestDistance = Infinity
      
      possibleDirections.forEach(direction => {
        const testX = Math.floor(this.x / 16)
        const testY = Math.floor(this.y / 16)
        
        let nextX = testX
        let nextY = testY
        
        switch(direction) {
          case 0: nextY--; break
          case 1: nextX++; break
          case 2: nextY++; break
          case 3: nextX--; break
        }
        
        const distance = Math.sqrt(
          Math.pow(nextX - target.x, 2) + 
          Math.pow(nextY - target.y, 2)
        )
        
        if (distance < shortestDistance) {
          shortestDistance = distance
          bestDirection = direction
        }
      })
      
      return bestDirection
    },
    move(speedMultiplier = 1) {
      const currentSpeed = this.speed * speedMultiplier
      switch(this.direction) {
        case 0: this.y -= currentSpeed; break
        case 1: this.x += currentSpeed; break
        case 2: this.y += currentSpeed; break
        case 3: this.x -= currentSpeed; break
      }
      
      // Handle tunnel wrapping
      if (this.x < -10) this.x = 448 + 10
      if (this.x > 448 + 10) this.x = -10
    },
    draw() {
      const color = this.frightened ? '#2121ff' : this.colors[this.type]
      
      // Draw body
      this.ctx.fillStyle = color
      this.ctx.beginPath()
      this.ctx.arc(this.x, this.y, 13, Math.PI, 0, false)
      this.ctx.lineTo(this.x + 13, this.y + 13)
      
      // Draw zigzag bottom
      for(let i = 12; i >= -12; i -= 4) {
        this.ctx.lineTo(this.x + i, this.y + (i % 8 === 0 ? 8 : 13))
      }
      
      this.ctx.fill()
      
      // Draw eyes
      const eyeX = this.x + (this.direction === 3 ? -4 : this.direction === 1 ? 4 : 0)
      const eyeY = this.y + (this.direction === 0 ? -4 : this.direction === 2 ? 4 : 0)
      
      this.ctx.fillStyle = 'white'
      this.ctx.beginPath()
      this.ctx.arc(eyeX - 3, this.y - 3, 3, 0, Math.PI * 2)
      this.ctx.arc(eyeX + 3, this.y - 3, 3, 0, Math.PI * 2)
      this.ctx.fill()
      
      this.ctx.fillStyle = 'black'
      this.ctx.beginPath()
      this.ctx.arc(eyeX - 3 + (this.direction === 3 ? -1 : this.direction === 1 ? 1 : 0),
                   this.y - 3 + (this.direction === 0 ? -1 : this.direction === 2 ? 1 : 0),
                   1.5, 0, Math.PI * 2)
      this.ctx.arc(eyeX + 3 + (this.direction === 3 ? -1 : this.direction === 1 ? 1 : 0),
                   this.y - 3 + (this.direction === 0 ? -1 : this.direction === 2 ? 1 : 0),
                   1.5, 0, Math.PI * 2)
      this.ctx.fill()
    }
  },
  beforeUnmount() {
    if (this.scatterTimer) {
      clearInterval(this.scatterTimer)
    }
  }
}
</script>