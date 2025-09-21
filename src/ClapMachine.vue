<template>
  <div class="clap-container" @click="handleClap">
    <div class="info-box">
      <h1>👏 Infinite Clap</h1>
      <p>
        Every click is a clap.<br />
        Every clap is eternal.<br />
        Will you stop? We won’t.<br />
        <em>Clap until the void claps back.</em>
      </p>
      <p class="slogan">💥 {{ randomSlogan }}</p>
      <div class="counter">👏 {{ count.toLocaleString() }}</div>
    </div>

    <div
        class="clap-emoji"
        v-for="clap in claps"
        :key="clap.id"
        :style="{ top: `${clap.y}px`, left: `${clap.x}px` }"
    >
      👏
    </div>

    <audio ref="audioRef" src="/clap.mp3" preload="auto" />
  </div>
</template>

<script lang="ts" setup>
import {onMounted, ref} from 'vue'
type Clap = {
  id: number
  x: number
  y: number
  rotate: number
  scale: number
  color?: string
}

const count = ref(0)
const claps = ref<Clap[]>([])
const audioRef = ref<HTMLAudioElement | null>(null)


const audioPool = Array.from({ length: 5 }, () => new Audio('/clapSound.wav'))
const specialAudio = new Audio('/clapMore.wav')
let poolIndex = 0

const slogans = [
  "Clap like nobody's watching.",
  "Clap therapy: just one more.",
  "Infinite? Try it.",
  "You thought it would end?",
  "Each clap fuels the chaos.",
  "Clap to charge your soul.",
  "The void is clapping back.",
  "Your wrist will hate this.",
  "This is fine. Keep clapping.",
  "404: End not found.",
]

const randomSlogan = ref("")

let lastSpecial = -1
onMounted(() => {
  randomSlogan.value = slogans[Math.floor(Math.random() * slogans.length)]
})

function handleClap(e: MouseEvent) {
  // 1. 박수 소리 재생 (Audio Pool)
  const audio = audioPool[poolIndex]
  audio.currentTime = 0
  audio.play()
  poolIndex = (poolIndex + 1) % audioPool.length

  // 2. 이펙트 표시
  const id = Date.now() + Math.random()
  const clap = { id, x: e.clientX, y: e.clientY }
  claps.value.push(clap)
  setTimeout(() => {
    claps.value = claps.value.filter((c) => c.id !== id)
  }, 600)

  // 3. 카운트 증가
  count.value++

  // ✅ 4. 100의 배수일 때 특별 사운드 재생
  if (count.value % 100 === 0 && count.value !== lastSpecial) {
    specialAudio.currentTime = 0
    specialAudio.play()
    lastSpecial = count.value
  }
}
</script>

<style lang="scss" scoped>
.clap-container {
  height: 100vh;
  width: 100%;
  background: linear-gradient(45deg, #ffcece, #ffeaa7, #dfe6e9);
  background-size: 400% 400%;
  animation: bgMove 10s infinite alternate;
  position: relative;
  cursor: url('https://emojicursor.app/cursors/clapping-hands.png'), auto;
  overflow: hidden;
  user-select: none;
  font-family: 'Comic Sans MS', 'DungGeunMo', sans-serif;
}

.counter {
  position: fixed;
  top: 30px;
  right: 30px;
  font-size: 40px;
  font-weight: 900;
  color: #222;
  background: #fdffb6;
  padding: 20px;
  border-radius: 20px;
  border: 4px dashed #ff7675;
  animation: wiggle 0.3s infinite;
}

.clap-emoji {
  position: absolute;
  font-size: 40px;
  animation: floatPop 0.6s ease-out forwards;
  pointer-events: none;
  transform: rotate(0deg);
  animation-delay: 0s;
}

.info-box h1 {
  font-size: 36px;
  font-weight: bold;
  color: #d63031;
  margin-bottom: 10px;
  font-family: 'Bangers', cursive; /* 해외 병맛 느낌 폰트 */
}

.info-box p {
  font-size: 18px;
  color: #2d3436;
  margin-bottom: 20px;
  line-height: 1.8;
  font-family: 'Gmarket Sans', sans-serif;
}

.info-box em {
  display: inline-block;
  margin-top: 10px;
  color: #0984e3;
  font-weight: bold;
  font-style: normal;
  font-family: 'Comic Sans MS', cursive;
}

.slogan {
  font-size: 16px;
  font-style: italic;
  color: #6c5ce7;
  margin-top: 10px;
  font-family: 'Comic Sans MS', cursive;
  animation: fadeIn 1s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes floatPop {
  0% {
    transform: scale(1) translateY(0) rotate(0deg);
    opacity: 1;
  }
  100% {
    transform: scale(2) translateY(-80px) rotate(720deg);
    opacity: 0;
  }
}

@keyframes wiggle {
  0%, 100% {
    transform: rotate(-2deg);
  }
  50% {
    transform: rotate(2deg);
  }
}

@keyframes bgMove {
  0% { background-position: 0% 50%; }
  100% { background-position: 100% 50%; }
}
</style>