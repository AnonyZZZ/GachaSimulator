<template>
  <div class="gacha">
  <div class="button-row">
    <button @contextmenu.prevent="handleRightClick" @click="pullTwelve" :disabled="isDrawing">十二连抽！</button>

  </div>

  <!-- 结果展示区不变 -->
  <div class="result-scroll" v-if="drawnCards.length">
    <transition-group name="slide-fade" tag="div" class="result-strip">
      <div class="card" v-for="(card, index) in drawnCards" :key="index">
        <div class="card-frame" :class="'r' + card.rarity">
          <img :src="card.image" :alt="card.name" />
          <div class="info">
            <p class="name">{{ card.name }}</p>
            <p class="rarity">[{{ card.rarity }}★]</p>
          </div>
        </div>
      </div>
    </transition-group>
  </div>
</div>
</template>

<script setup>
import { ref } from 'vue';
import gachaPool from '../data/gachaPool.js';

const drawnCards = ref([]);
const isDrawing = ref(false);

function drawSingle() {
  const rand = Math.random();
  let cumulative = 0;
  for (let group of gachaPool) {
    cumulative += group.rate;
    if (rand < cumulative) {
      const card = group.cards[Math.floor(Math.random() * group.cards.length)];
      return { rarity: group.rarity, ...card };
    }
  }
  return null;
}

async function pullTwelve() {
  if (isDrawing.value) return;
  isDrawing.value = true;
  drawnCards.value = [];

  await new Promise(resolve => setTimeout(resolve, 300));
  const pullResult = [];
  
  // 先抽 12 张卡存入临时结果
  for (let i = 0; i < 12; i++) {
    const card = drawSingle();
    pullResult.push(card);
  }

  // 检查是否含 U-Official，没有则保底替换
  const hasTarget = pullResult.some(card => card.name === 'U-Official');

  if (!hasTarget) {
    console.log('🎯 保底触发！注入 U-Official');

    const uoGroup = gachaPool.find(group =>
      group.cards.some(card => card.name === 'U-Official')
    );
    const uoCard = uoGroup.cards.find(card => card.name === 'U-Official');
    const uoFull = { rarity: uoGroup.rarity, ...uoCard };

    const replaceIndex = Math.floor(Math.random() * 12);
    pullResult[replaceIndex] = uoFull;
  }

  // 再逐张显示（动画）
  for (let i = 0; i < 12; i++) {
    drawnCards.value.push(pullResult[i]);
    await new Promise(resolve => setTimeout(resolve, 200)); // 保持每张卡缓入
  }

  isDrawing.value = false;
}

</script>

<style scoped>
.gacha {
  text-align: center;
  padding: 20px;
  background: #0e0e0e;
  color: white;
}

button {
  padding: 10px 20px;
  font-size: 18px;
  margin-bottom: 20px;
  background: #444;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.result-scroll {
  overflow-x: auto;
  padding: 10px;
  white-space: nowrap;
}

.result-strip {
  display: flex;
  gap: 16px;
  justify-content: flex-start;
}

.card {
  flex: 0 0 auto;
  animation: pop-in 0.3s ease;
}

.card-frame {
  background: #1c1c1c;
  border-radius: 10px;
  padding: 10px;
  width: 130px;
  text-align: center;
  border: 2px solid #999;
  transition: transform 0.3s;
}

.card-frame img {
  width: 100px;
  height: auto;
  border-radius: 6px;
}

.info {
  margin-top: 5px;
}
.name {
  font-weight: bold;
  font-size: 14px;
}
.rarity {
  font-size: 12px;
  opacity: 0.7;
}

.slide-fade-enter-active {
  transition: all 0.5s ease;
}
.slide-fade-enter-from {
  transform: translateX(50px);
  opacity: 0;
}

@keyframes pop-in {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

/* 稀有度颜色边框 + 动态光晕 */
.card-frame.r6 {
  border-color: rgb(255, 174, 0);
  box-shadow: 0 0 12px rgb(255, 174, 0);
}
.card-frame.r5 {
  border-color: gold;
  box-shadow: 0 0 12px gold;
}
.card-frame.r4 {
  border-color: violet;
}
.card-frame.r3 {
  border-color: dodgerblue;
}
.card-frame.r2 {
  border-color: #00ff33;
}
.card-frame.r1 {
  border-color: #ffffff;
}

/* 光晕动效 */
.card-frame.r6::after,
.card-frame.r5::after,
.card-frame.r4::after {
  content: "";
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  pointer-events: none;
  z-index: 1;
  border-radius: 50%;
  mix-blend-mode: screen;
  animation: halo-pulse 2.5s infinite;
}

.card-frame.r6::after {
  background: radial-gradient(circle, rgba(255,215,0,0.4), rgba(0,0,0,0));
}
.card-frame.r5::after {
  background: radial-gradient(circle, rgba(180,100,255,0.3), rgba(0,0,0,0));
}
.card-frame.r4::after {
  background: radial-gradient(circle, rgba(100,200,255,0.2), rgba(0,0,0,0));
}

/* 呼吸动效 */
@keyframes halo-pulse {
  0% { transform: scale(0.9); opacity: 0.4; }
  50% { transform: scale(1.1); opacity: 0.8; }
  100% { transform: scale(0.9); opacity: 0.4; }
}

.card-frame {
  position: relative;
  overflow: hidden;
  transition: transform 0.3s;
  z-index: 0;
}

/* === 按钮 Hover 特效 === */
button {
  padding: 10px 20px;
  font-size: 18px;
  margin-bottom: 20px;
  background: linear-gradient(145deg, #444, #333);
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
}

button:hover {
  background: linear-gradient(145deg, #666, #555);
  transform: scale(1.05);
  box-shadow: 0 0 12px rgba(255, 215, 0, 0.6); /* 类似金色光效 */
}
</style>
