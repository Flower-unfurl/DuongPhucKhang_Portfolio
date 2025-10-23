<template>
  <div id="console">
    <!-- bolts -->
    <img id="corner" src="/icons/console/bolt-up-left.svg" alt="" class="absolute top-2 left-2 opacity-70">
    <img id="corner" src="/icons/console/bolt-up-right.svg" alt="" class="absolute top-2 right-2 opacity-70">
    <img id="corner" src="/icons/console/bolt-down-left.svg" alt="" class="absolute bottom-2 left-2 opacity-70">
    <img id="corner" src="/icons/console/bolt-down-right.svg" alt="" class="absolute bottom-2 right-2 opacity-70">

    <!-- Game Screen -->
    <div id="game-screen-container" class="relative">
      <canvas 
        ref="gameCanvas" 
        :width="canvasWidth" 
        :height="canvasHeight"
        id="game-canvas"
      />
      
      <!-- Mode Selection -->
      <div v-if="!gameStarted && !gameOver" id="mode-selection">
        <div class="px-8 py-12">
          <span class="font-roboto_mono_regular text-greenfy text-lg mb-6 block text-center">
            SELECT MODE
          </span>
          <button 
            class="mode-button font-roboto_mono_regular mb-3" 
            @click="startGame('auto')"
          >
            🤖 auto-play
          </button>
          <button 
            class="mode-button font-roboto_mono_regular" 
            @click="startGame('player')"
          >
            🎮 player-vs-ai
          </button>
        </div>
      </div>

      <!-- Game Over -->
      <div v-if="gameOver" id="game-over">
        <span class="font-roboto_mono_regular text-greenfy bg-bluefy-dark h-12 flex items-center justify-center">
          {{ mode === 'auto' 
            ? (winner === 'player' ? 'AI-1 WINS!' : 'AI-2 WINS!') 
            : (winner === 'player' ? 'YOU WIN!' : 'AI WINS!') 
          }}
        </span>
        <div class="font-roboto_mono_regular text-menu-text text-sm py-4 text-center">
          <p class="text-white text-lg mb-2">{{ playerScore }} - {{ aiScore }}</p>
          <p class="text-xs">{{ mode === 'auto' ? 'Auto-play completed' : 'Match finished' }}</p>
        </div>
        <button 
          class="font-roboto_mono_regular text-menu-text text-sm flex items-center justify-center w-full py-4 hover:text-white" 
          @click="restart"
        >
          play-again
        </button>
      </div>
    </div>

    <!-- Console Menu -->
    <div id="console-menu" class="h-full flex flex-col items-end justify-between">
      <div class="w-full">
        <!-- Instructions -->
        <div id="instructions" class="font-roboto_mono_regular text-sm text-white">
          <p v-if="mode === 'player' && gameStarted">// use arrow keys</p>
          <p v-if="mode === 'player' && gameStarted">// ↑↓ to move paddle</p>
          <p v-if="mode === 'auto' && gameStarted">// watching AI play</p>
          <p v-if="!gameStarted">// choose game mode</p>

          <div v-if="mode === 'player' && gameStarted" id="buttons" class="w-full flex flex-col items-center gap-1 pt-3">
            <button 
              id="console-button" 
              class="button-up"
              @mousedown="startMovingPaddle('up')"
              @mouseup="stopMovingPaddle"
              @mouseleave="stopMovingPaddle"
            >
              <img src="/icons/console/arrow-button.svg" alt="move up">
            </button>

            <button 
              id="console-button" 
              class="button-down"
              @mousedown="startMovingPaddle('down')"
              @mouseup="stopMovingPaddle"
              @mouseleave="stopMovingPaddle"
            >
              <img src="/icons/console/arrow-button.svg" alt="move down" class="rotate-180">
            </button>
          </div>
        </div>

        <!-- Game Info -->
        <div class="font-roboto_mono_regular text-white pt-3 px-3">
          <p class="text-xs text-menu-text">// current mode</p>
          <p class="text-sm text-greenfy font-roboto_mono_bold pt-1">
            {{ mode === 'auto' ? 'AUTO-PLAY' : mode === 'player' ? 'PLAYER-VS-AI' : 'NOT STARTED' }}
          </p>
        </div>
      </div>

      <!-- Score Dots -->
      <div id="score-dots" class="w-full flex flex-col px-5 pt-3">
        <p class="font-roboto_mono_regular text-white text-xs text-menu-text pb-2">// points</p>
        <div class="grid grid-cols-5 gap-2 justify-items-center">
          <div 
            v-for="i in 5" 
            :key="i" 
            class="score-dot"
            :class="{ active: i <= playerScore }"
          ></div>
        </div>
        <div class="grid grid-cols-5 gap-2 justify-items-center pt-1">
          <div 
            v-for="i in 5" 
            :key="i" 
            class="score-dot"
            :class="{ active: i <= aiScore }"
          ></div>
        </div>
      </div>

      <!-- Score Board -->
      <div id="score-board" class="w-full flex flex-col pl-5 pb-3">
        <p class="font-roboto_mono_regular text-white pt-3 text-xs text-menu-text">// score</p>
        <div class="grid grid-cols-3 gap-2 items-center pt-2">
          <div class="text-center">
            <p class="text-xs text-menu-text mb-1">{{ mode === 'player' ? 'YOU' : 'AI-1' }}</p>
            <p class="text-2xl text-greenfy font-roboto_mono_bold">{{ playerScore }}</p>
          </div>
          <div class="text-center">
            <p class="text-white text-xl">:</p>
          </div>
          <div class="text-center">
            <p class="text-xs text-menu-text mb-1">{{ mode === 'auto' ? 'AI-2' : 'AI' }}</p>
            <p class="text-2xl text-greenfy font-roboto_mono_bold">{{ aiScore }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Skip Button -->
    <NuxtLink 
      id="skip-btn" 
      to="/about-me" 
      class="font-roboto_mono_regular flex hover:bg-white/20"
    >
      skip
    </NuxtLink>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

// Canvas dimensions - expanded for better gameplay
const canvasWidth = ref(280)
const canvasHeight = ref(420)

// Game state
const gameCanvas = ref<HTMLCanvasElement | null>(null)
let ctx: CanvasRenderingContext2D | null = null
let animationFrameId: number | null = null

const gameStarted = ref(false)
const gameOver = ref(false)
const mode = ref<'auto' | 'player' | null>(null)
const winner = ref<'player' | 'ai' | null>(null)

// Scores
const playerScore = ref(0)
const aiScore = ref(0)
const maxScore = 5

// Game objects
interface Paddle {
  x: number
  y: number
  width: number
  height: number
  speed: number
}

interface Ball {
  x: number
  y: number
  radius: number
  velocityX: number
  velocityY: number
  speed: number
}

const paddleWidth = 4
const paddleHeight = 50
const ballRadius = 4

const playerPaddle = ref<Paddle>({
  x: 10,
  y: canvasHeight.value / 2 - paddleHeight / 2,
  width: paddleWidth,
  height: paddleHeight,
  speed: 5
})

const aiPaddle = ref<Paddle>({
  x: canvasWidth.value - 10 - paddleWidth,
  y: canvasHeight.value / 2 - paddleHeight / 2,
  width: paddleWidth,
  height: paddleHeight,
  speed: 3
})

const ball = ref<Ball>({
  x: canvasWidth.value / 2,
  y: canvasHeight.value / 2,
  radius: ballRadius,
  velocityX: 2.5,
  velocityY: 2.5,
  speed: 2.5
})

// Player controls
let paddleDirection: 'up' | 'down' | null = null
let movingPaddle = false

onMounted(() => {
  if (gameCanvas.value) {
    ctx = gameCanvas.value.getContext('2d')
    window.addEventListener('keydown', handleKeyDown)
    window.addEventListener('keyup', handleKeyUp)
  }
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
  window.removeEventListener('keyup', handleKeyUp)
  if (animationFrameId) cancelAnimationFrame(animationFrameId)
})

// Keyboard controls
const handleKeyDown = (e: KeyboardEvent) => {
  if (mode.value !== 'player' || !gameStarted.value) return
  
  if (e.code === 'ArrowUp') {
    e.preventDefault()
    paddleDirection = 'up'
  } else if (e.code === 'ArrowDown') {
    e.preventDefault()
    paddleDirection = 'down'
  }
}

const handleKeyUp = (e: KeyboardEvent) => {
  if (e.code === 'ArrowUp' || e.code === 'ArrowDown') {
    paddleDirection = null
  }
}

// Touch/Mouse controls
const startMovingPaddle = (direction: 'up' | 'down') => {
  if (mode.value !== 'player' || !gameStarted.value) return
  paddleDirection = direction
  movingPaddle = true
}

const stopMovingPaddle = () => {
  paddleDirection = null
  movingPaddle = false
}

// Start game with selected mode
const startGame = (selectedMode: 'auto' | 'player') => {
  mode.value = selectedMode
  gameStarted.value = true
  gameOver.value = false
  winner.value = null
  playerScore.value = 0
  aiScore.value = 0
  
  resetBall()
  
  playerPaddle.value.y = canvasHeight.value / 2 - paddleHeight / 2
  aiPaddle.value.y = canvasHeight.value / 2 - paddleHeight / 2
  
  gameLoop()
}

// Restart game
const restart = () => {
  mode.value = null
  gameStarted.value = false
  gameOver.value = false
  winner.value = null
  playerScore.value = 0
  aiScore.value = 0
  paddleDirection = null
  movingPaddle = false
}

// Reset ball position
const resetBall = () => {
  ball.value.x = canvasWidth.value / 2
  ball.value.y = canvasHeight.value / 2
  // Slower ball speed for better gameplay
  ball.value.speed = mode.value === 'auto' ? 3 : 2.5
  
  // Random direction
  const angle = (Math.random() * Math.PI / 4) - Math.PI / 8
  const direction = Math.random() > 0.5 ? 1 : -1
  
  ball.value.velocityX = direction * ball.value.speed * Math.cos(angle)
  ball.value.velocityY = ball.value.speed * Math.sin(angle)
}

// Main game loop
const gameLoop = () => {
  if (!ctx || !gameStarted.value || gameOver.value) return
  
  update()
  draw()
  
  animationFrameId = requestAnimationFrame(gameLoop)
}

// Update game state
const update = () => {
  // Move player paddle
  if (mode.value === 'player') {
    if (paddleDirection === 'up') {
      playerPaddle.value.y = Math.max(0, playerPaddle.value.y - playerPaddle.value.speed)
    } else if (paddleDirection === 'down') {
      playerPaddle.value.y = Math.min(
        canvasHeight.value - playerPaddle.value.height,
        playerPaddle.value.y + playerPaddle.value.speed
      )
    }
  } else if (mode.value === 'auto') {
    // AI controls left paddle in auto mode with higher intelligence
    const leftPaddleCenter = playerPaddle.value.y + playerPaddle.value.height / 2
    const predictedY = ball.value.y + ball.value.velocityY * 15
    
    if (predictedY < leftPaddleCenter - 3) {
      playerPaddle.value.y = Math.max(0, playerPaddle.value.y - playerPaddle.value.speed)
    } else if (predictedY > leftPaddleCenter + 3) {
      playerPaddle.value.y = Math.min(
        canvasHeight.value - playerPaddle.value.height,
        playerPaddle.value.y + playerPaddle.value.speed
      )
    }
  }
  
  // AI paddle movement (right side)
  const aiPaddleCenter = aiPaddle.value.y + aiPaddle.value.height / 2
  
  if (mode.value === 'auto') {
    // In auto mode, AI is smarter with prediction
    const predictedY = ball.value.y + ball.value.velocityY * 15
    
    if (predictedY < aiPaddleCenter - 3) {
      aiPaddle.value.y = Math.max(0, aiPaddle.value.y - aiPaddle.value.speed)
    } else if (predictedY > aiPaddleCenter + 3) {
      aiPaddle.value.y = Math.min(
        canvasHeight.value - aiPaddle.value.height,
        aiPaddle.value.y + aiPaddle.value.speed
      )
    }
  } else {
    // In player mode, AI is easier to beat - less accurate prediction
    const predictedY = ball.value.y + ball.value.velocityY * 5
    
    if (predictedY < aiPaddleCenter - 15) {
      aiPaddle.value.y = Math.max(0, aiPaddle.value.y - aiPaddle.value.speed)
    } else if (predictedY > aiPaddleCenter + 15) {
      aiPaddle.value.y = Math.min(
        canvasHeight.value - aiPaddle.value.height,
        aiPaddle.value.y + aiPaddle.value.speed
      )
    }
  }
  
  // Move ball
  ball.value.x += ball.value.velocityX
  ball.value.y += ball.value.velocityY
  
  // Ball collision with top/bottom walls
  if (ball.value.y - ball.value.radius <= 0 || ball.value.y + ball.value.radius >= canvasHeight.value) {
    ball.value.velocityY = -ball.value.velocityY
  }
  
  // Ball collision with paddles
  // Left paddle
  if (
    ball.value.x - ball.value.radius <= playerPaddle.value.x + playerPaddle.value.width &&
    ball.value.x + ball.value.radius >= playerPaddle.value.x &&
    ball.value.y >= playerPaddle.value.y &&
    ball.value.y <= playerPaddle.value.y + playerPaddle.value.height
  ) {
    const hitPos = (ball.value.y - (playerPaddle.value.y + playerPaddle.value.height / 2)) / (playerPaddle.value.height / 2)
    const angle = hitPos * (Math.PI / 4)
    
    ball.value.speed = Math.min(ball.value.speed + 0.08, 4.5)
    ball.value.velocityX = ball.value.speed * Math.cos(angle)
    ball.value.velocityY = ball.value.speed * Math.sin(angle)
    
    if (ball.value.velocityX < 0) ball.value.velocityX = -ball.value.velocityX
  }
  
  // Right paddle
  if (
    ball.value.x + ball.value.radius >= aiPaddle.value.x &&
    ball.value.x - ball.value.radius <= aiPaddle.value.x + aiPaddle.value.width &&
    ball.value.y >= aiPaddle.value.y &&
    ball.value.y <= aiPaddle.value.y + aiPaddle.value.height
  ) {
    const hitPos = (ball.value.y - (aiPaddle.value.y + aiPaddle.value.height / 2)) / (aiPaddle.value.height / 2)
    const angle = hitPos * (Math.PI / 4)
    
    ball.value.speed = Math.min(ball.value.speed + 0.08, 4.5)
    ball.value.velocityX = -ball.value.speed * Math.cos(angle)
    ball.value.velocityY = ball.value.speed * Math.sin(angle)
    
    if (ball.value.velocityX > 0) ball.value.velocityX = -ball.value.velocityX
  }
  
  // Score detection
  if (ball.value.x - ball.value.radius <= 0) {
    // AI scores
    aiScore.value++
    if (aiScore.value >= maxScore) {
      endGame('ai')
    } else {
      resetBall()
    }
  } else if (ball.value.x + ball.value.radius >= canvasWidth.value) {
    // Player scores
    playerScore.value++
    if (playerScore.value >= maxScore) {
      endGame('player')
    } else {
      resetBall()
    }
  }
}

// End game
const endGame = (winnerSide: 'player' | 'ai') => {
  gameOver.value = true
  gameStarted.value = false
  winner.value = winnerSide
  
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
    animationFrameId = null
  }
}

// Draw everything
const draw = () => {
  if (!ctx) return
  
  // Clear canvas with gradient background
  const gradient = ctx.createLinearGradient(0, 0, 0, canvasHeight.value)
  gradient.addColorStop(0, 'rgba(1, 22, 39, 0.5)')
  gradient.addColorStop(0.5, 'rgba(17, 47, 65, 0.4)')
  gradient.addColorStop(1, 'rgba(1, 22, 39, 0.5)')
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, canvasWidth.value, canvasHeight.value)
  
  // Draw center line
  ctx.strokeStyle = 'rgba(67, 217, 173, 0.3)'
  ctx.lineWidth = 2
  ctx.setLineDash([5, 5])
  ctx.beginPath()
  ctx.moveTo(canvasWidth.value / 2, 0)
  ctx.lineTo(canvasWidth.value / 2, canvasHeight.value)
  ctx.stroke()
  ctx.setLineDash([])
  
  // Draw paddles
  ctx.fillStyle = '#43D9AD'
  ctx.shadowColor = '#43D9AD'
  ctx.shadowBlur = 10
  
  // Left paddle
  ctx.fillRect(
    playerPaddle.value.x,
    playerPaddle.value.y,
    playerPaddle.value.width,
    playerPaddle.value.height
  )
  
  // Right paddle
  ctx.fillRect(
    aiPaddle.value.x,
    aiPaddle.value.y,
    aiPaddle.value.width,
    aiPaddle.value.height
  )
  
  // Draw ball
  ctx.beginPath()
  ctx.arc(ball.value.x, ball.value.y, ball.value.radius, 0, Math.PI * 2)
  ctx.fillStyle = '#43D9AD'
  ctx.fill()
  ctx.closePath()
  
  // Reset shadow
  ctx.shadowBlur = 0
}
</script>

<style scoped>
#console {
  width: 600px;
  height: 540px;
  border: 2px solid rgba(67, 217, 173, 0.3);
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: linear-gradient(135deg, 
    rgba(1, 22, 39, 0.4) 0%,
    rgba(35, 123, 109, 0.3) 50%,
    rgba(67, 217, 173, 0.2) 100%);
  backdrop-filter: blur(20px);
  border-radius: 16px;
  padding: 30px;
  position: relative;
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

#console:hover {
  transform: translateY(-5px);
  border-color: rgba(67, 217, 173, 0.6);
  box-shadow: 
    0 12px 48px rgba(67, 217, 173, 0.3),
    0 0 80px rgba(67, 217, 173, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  background: linear-gradient(135deg, 
    rgba(1, 22, 39, 0.5) 0%,
    rgba(35, 123, 109, 0.4) 50%,
    rgba(67, 217, 173, 0.25) 100%);
}

#game-screen-container {
  position: relative;
  width: 280px;
  height: 420px;
}

#game-canvas {
  width: 280px;
  height: 420px;
  border-radius: 12px;
  background: linear-gradient(135deg, rgba(1, 22, 39, 0.7), rgba(17, 47, 65, 0.6));
  backdrop-filter: blur(10px);
  box-shadow: 
    inset 0 0 20px rgba(0, 0, 0, 0.5),
    0 0 30px rgba(67, 217, 173, 0.3),
    0 0 50px rgba(67, 217, 173, 0.2);
  border: 2px solid rgba(67, 217, 173, 0.4);
  transition: all 0.3s ease;
}

#game-canvas:hover {
  box-shadow: 
    inset 0 0 20px rgba(0, 0, 0, 0.5),
    0 0 40px rgba(67, 217, 173, 0.5),
    0 0 70px rgba(67, 217, 173, 0.3);
  border-color: rgba(67, 217, 173, 0.7);
}

#mode-selection {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 280px;
  height: 420px;
  padding: 0;
  background: linear-gradient(135deg, rgba(1, 22, 39, 0.95), rgba(17, 47, 65, 0.9));
  backdrop-filter: blur(20px);
  border-radius: 12px;
  border: 2px solid rgba(67, 217, 173, 0.6);
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.5),
    0 0 40px rgba(67, 217, 173, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  z-index: 10;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.mode-button {
  width: 100%;
  padding: 12px 16px;
  border-radius: 8px;
  border: 1px solid rgba(67, 217, 173, 0.5);
  background: linear-gradient(135deg, rgba(67, 217, 173, 0.1), rgba(67, 217, 173, 0.05));
  backdrop-filter: blur(10px);
  color: #43D9AD;
  cursor: pointer;
  font-size: 0.875rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.mode-button:hover {
  background: linear-gradient(135deg, rgba(67, 217, 173, 0.25), rgba(67, 217, 173, 0.15));
  border-color: rgba(67, 217, 173, 0.8);
  box-shadow: 
    0 0 25px rgba(67, 217, 173, 0.6),
    0 8px 20px rgba(0, 0, 0, 0.3);
  transform: translateY(-3px);
}

#console-menu{
  height: 420px;
}

#console-button {
  background: linear-gradient(135deg, rgba(1, 12, 21, 0.8), rgba(17, 47, 65, 0.6));
  backdrop-filter: blur(10px);
  border: 1px solid rgba(67, 217, 173, 0.3);
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 45px;
  height: 28px;
  transition: all 0.3s ease;
}

#console-button:hover {
  background: linear-gradient(135deg, rgba(1, 12, 21, 0.9), rgba(17, 47, 65, 0.8));
  border-color: rgba(67, 217, 173, 0.6);
  box-shadow: 0 0 20px rgba(67, 217, 173, 0.5);
  transform: translateY(-2px);
}

#instructions {
  background: linear-gradient(135deg, rgba(1, 20, 35, 0.3), rgba(17, 47, 65, 0.2));
  backdrop-filter: blur(10px);
  border: 1px solid rgba(67, 217, 173, 0.2);
  border-radius: 8px;
  padding: 8px;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

#game-over {
  position: absolute;
  bottom: 12%;
  left: 0;
  color: #43D9AD;
  width: 280px;
  background: linear-gradient(135deg, rgba(1, 22, 39, 0.95), rgba(17, 47, 65, 0.9));
  backdrop-filter: blur(20px);
  border-radius: 12px;
  border: 2px solid rgba(67, 217, 173, 0.6);
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.5),
    0 0 40px rgba(67, 217, 173, 0.3);
  overflow: hidden;
  z-index: 10;
}

#corner {
  width: 24px;
  height: 24px;
}

.score-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: rgba(67, 217, 173, 0.2);
  border: 1px solid rgba(67, 217, 173, 0.3);
  transition: all 0.3s ease;
}

.score-dot.active {
  background-color: #43D9AD;
  border-color: #43D9AD;
  box-shadow: 
    0 0 10px rgba(67, 217, 173, 0.8),
    0 0 20px rgba(67, 217, 173, 0.4);
}

#skip-btn {
  font-size: 14px;
  color: white;
  padding-inline: 16px;
  padding-block: 8px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.05), rgba(255, 255, 255, 0.02));
  backdrop-filter: blur(10px);
  border-radius: 0.5rem;
  position: absolute;
  bottom: 20px;
  right: 30px;
  transition: all 0.3s ease;
}

#skip-btn:hover {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.15), rgba(255, 255, 255, 0.08));
  border-color: rgba(255, 255, 255, 0.6);
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

/* Responsive styles */
@media (min-width: 1024px) and (max-width: 1536px) {
  #console {
    width: 480px;
    height: 430px;
    padding: 24px;
  }

  #game-screen-container {
    width: 224px;
    height: 336px;
  }

  #game-canvas {
    width: 224px;
    height: 336px;
  }

  #console-menu {
    height: 320px;
  }

  #instructions {
    font-size: 12px;
  }

  #console-button {
    width: 40px;
    height: 25px;
    border-radius: 6px;
  }

  #mode-selection {
    width: 224px;
    height: 336px;
    padding: 0;
  }

  .mode-button {
    padding: 10px 12px;
    font-size: 0.75rem;
  }

  #game-over {
    width: 224px;
  }

  .score-dot {
    width: 8px;
    height: 8px;
  }

  #corner {
    width: 20px;
    height: 20px;
  }

  #skip-btn {
    font-size: 12px;
    padding-inline: 12px;
    padding-block: 6px;
  }
}
</style>
