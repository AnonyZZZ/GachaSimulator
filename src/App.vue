<template>
  <div class="app-container">
    <transition name="background-transition">
      <div
        v-if="showBackground"
        class="background-image"
        :class="{ 'background-corner': started }"
      />
    </transition>

    <!-- 启动页 -->
    <div v-if="!started" class="start-screen">
      <button class="start-btn" @click="startGacha">开始抽卡</button>
    </div>

    <!-- 抽卡页 -->
    <div v-else class="gacha-screen">
      <Gacha />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import Gacha from './components/Gacha.vue';

const started = ref(false);
const showBackground = ref(true);

const startGacha = () => {
  started.value = true;
};
</script>

<style scoped>
.app-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  position: relative;
  background: #000;
}

/* === 初始背景图样式 === */
.background-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: url('/background.png') center center / cover no-repeat;
  transition: all 1s ease;
  z-index: 1;
  border-radius: 0;
}

/* === 缩小后背景图样式 === */
.background-corner {
  width: 2500px;
  height: 1500px;
  bottom: -360px;
  left: -600px;
  top: auto;
  transform: scale(0.4);
  border-radius: 12px;
  box-shadow: 0 0 20px rgba(255,255,255,0.3);
}

/* === 启动界面 === */
.start-screen {
  position: absolute;
  bottom: 160px;
  right: 160px;
  z-index: 2;
}

.start-btn {
  width: 320px; /* 原来是自适应，现在固定宽 */
  height: 72px;
  font-size: 18px;
  padding: 12px 0;

  font-size: 18px;
  padding: 12px 24px;
  background: rgba(255, 238, 0, 0.9);
  border: none;
  border-radius: 12px;
  color: #222;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 0 12px rgba(255, 255, 255, 0.6);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.start-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.8);
}

/* === 抽卡界面 === */
.gacha-screen {
  z-index: 2;
  position: relative;
  padding-top: 40px;
}
</style>