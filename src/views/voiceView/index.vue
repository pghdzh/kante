<template>
  <section class="cantarella-voice">
    <!-- 背景轮播（两组用于桌面/移动） -->
    <div class="carousel carousel-desktop" aria-hidden="true">
      <transition-group name="bg-fade" tag="div" class="bg-layer">
        <img
          v-for="(src, idx) in randomFive"
          :key="`bg-${idx}`"
          :src="src"
          class="carousel-image"
          :class="{ active: idx === currentIndex }"
          alt=""
        />
      </transition-group>
    </div>
    <div class="carousel carousel-mobile" aria-hidden="true">
      <transition-group name="bg-fade" tag="div" class="bg-layer">
        <img
          v-for="(src, idx) in randomFive2"
          :key="`bg-m-${idx}`"
          :src="src"
          class="carousel-image"
          :class="{ active: idx === currentIndex }"
          alt=""
        />
      </transition-group>
    </div>

    <div class="voice-wrap">
      <!-- 头部：深海明珠 -->
      <header class="voice-header">
        <div class="pearl">
          <h1>翡萨烈·渊海呢喃</h1>
          <p class="subtitle">在深渊回响中，聆听往日的低语与誓言</p>
        </div>
        <div class="header-decoration">
          <span></span><span></span><span></span>
        </div>
      </header>

      <!-- 语音列表 -->
      <ul class="voice-list" role="list">
        <li
          v-for="id in allVoiceIds"
          :key="id"
          class="voice-item"
          :class="{ playing: id === currentId }"
        >
          <div class="item-inner">
            <div class="item-left">
              <div class="index-disc">{{ String(id).padStart(3, "0") }}</div>
              <div class="info">
                <div class="name">语音 {{ String(id).padStart(3, "0") }}</div>
              </div>
            </div>

            <div class="item-right">
              <button
                class="play-btn"
                @click="playEntry(id)"
                :title="currentId === id && isPlaying ? '暂停' : '播放'"
              >
                <span v-if="currentId === id && isPlaying">❚❚</span>
                <span v-else>▶</span>
              </button>

              <a
                :href="voicePath(id)"
                :download="`audio_${id}.mp3`"
                title="下载"
              >
                <el-button
                  type="primary"
                  :icon="Download"
                  circle
                  class="download-btn"
                />
              </a>
            </div>
            <div class="item-depth"></div>
          </div>
        </li>
      </ul>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";
import { Download } from "@element-plus/icons-vue";

/* ================== 配置 ================== */
const TOTAL_VOICES = 21; // 语音总数，按实际替换
const BG_INTERVAL_MS = 4500; // 背景切换间隔（毫秒）
const MOBILE_BREAKPOINT = 720; // 小于这个宽度视为移动端
/* ========================================= */

/* ========== 导入图片资源 ========== */
const modules: Record<string, any> = import.meta.glob(
  "@/assets/images1/*.{jpg,png,jpeg,webp}",
  { eager: true }
);
const allSrcs: string[] = Object.values(modules).map(
  (m: any) => m.default || m
);

const modules2: Record<string, any> = import.meta.glob(
  "@/assets/images2/*.{jpg,png,jpeg,webp}",
  { eager: true }
);
const allSrcs2: string[] = Object.values(modules2).map(
  (m: any) => m.default || m
);

function shuffle<T>(arr: T[]) {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

const randomFive = ref<string[]>(
  shuffle(allSrcs).slice(0, Math.min(5, allSrcs.length))
);
const randomFive2 = ref<string[]>(
  shuffle(allSrcs2).slice(0, Math.min(5, allSrcs2.length))
);

const currentIndex = ref(0);
let imgTimer: number | null = null;

const isMobile = ref(window.innerWidth < MOBILE_BREAKPOINT);
function handleResize() {
  const nowMobile = window.innerWidth < MOBILE_BREAKPOINT;
  if (nowMobile !== isMobile.value) {
    isMobile.value = nowMobile;
    currentIndex.value = 0;
  }
}

/* ========== 语音列表与播放逻辑 ========== */
const allIds = Array.from({ length: TOTAL_VOICES }, (_, i) => i);
const allVoiceIds = allIds;

let currentAudio: HTMLAudioElement | null = null;
const currentId = ref<number | null>(null);
const isPlaying = ref(false);

function createAudio(src?: string) {
  destroyAudio();
  currentAudio = new Audio();
  currentAudio.preload = "auto";
  if (src) currentAudio.src = src;
  currentAudio.addEventListener("ended", onEnded);
  currentAudio.addEventListener("error", onAudioError);
}
function destroyAudio() {
  if (!currentAudio) return;
  try {
    currentAudio.pause();
    currentAudio.removeEventListener("ended", onEnded);
    currentAudio.removeEventListener("error", onAudioError);
    currentAudio.src = "";
  } catch (e) {}
  currentAudio = null;
}
function onEnded() {
  isPlaying.value = false;
}
function onAudioError(e?: any) {
  console.error("audio error", e);
  isPlaying.value = false;
}

function voicePath(id: number) {
  return `/voice/audio (${id}).mp3`;
}

async function playEntry(id: number) {
  if (currentId.value === id && isPlaying.value) {
    currentAudio?.pause();
    isPlaying.value = false;
    return;
  }
  if (currentId.value === id && !isPlaying.value && currentAudio) {
    try {
      await currentAudio.play();
      isPlaying.value = true;
    } catch (e) {
      console.warn(e);
    }
    return;
  }

  currentId.value = id;
  createAudio(voicePath(id));
  try {
    await currentAudio!.play();
    isPlaying.value = true;
  } catch (e) {
    console.warn("播放被阻止或失败", e);
    isPlaying.value = false;
  }
}

/* ========== 背景轮播控制 ========== */
function startBgTimer() {
  stopBgTimer();
  imgTimer = window.setInterval(() => {
    const len = isMobile.value
      ? randomFive2.value.length
      : randomFive.value.length;
    currentIndex.value = (currentIndex.value + 1) % Math.max(1, len);
  }, BG_INTERVAL_MS);
}
function stopBgTimer() {
  if (imgTimer !== null) {
    clearInterval(imgTimer);
    imgTimer = null;
  }
}

onMounted(() => {
  window.addEventListener("resize", handleResize);
  startBgTimer();
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
  stopBgTimer();
  destroyAudio();
});

watch([isMobile, randomFive, randomFive2], () => {
  currentIndex.value = 0;
});
</script>

<style scoped lang="scss">
/* 坎特蕾拉色板 */
$deep-bg: #030614; // 极深海
$mid-bg: #14213d; // 深海中层
$accent-lavender: #b8a9ff; // 薰衣草紫
$accent-aqua: #7ae2ff; // 水母荧光
$accent-pink: #ffb3c6; // 毒药粉
$accent-deep-blue: #2a4a7a; // 深海蓝
$text-light: #f0f5fe;
$card-bg: rgba(255, 255, 255, 0.02);
$glass-edge: rgba($accent-aqua, 0.2);

.cantarella-voice {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(180deg, $deep-bg 0%, $mid-bg 100%);
  color: $text-light;
  overflow-x: hidden;
  padding: 80px 20px 60px;
  font-family: "PingFang SC", "Noto Sans SC", system-ui, sans-serif;

  /* 背景轮播 */
  .carousel {
    position: fixed;
    inset: 0;
    z-index: 0;
    pointer-events: none;

    &::after {
      content: "";
      position: absolute;
      inset: 0;
      background: linear-gradient(
        180deg,
        rgba($deep-bg, 0.6) 0%,
        rgba($mid-bg, 0.8) 100%
      );
      z-index: 1;
    }

    .bg-layer {
      position: absolute;
      inset: 0;
    }

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 2s ease-in-out;
     
      &.active {
        opacity: 1;
      }
    }
  }
  .carousel-mobile {
    display: none;
  }

  /* 主内容容器 */
  .voice-wrap {
    position: relative;
    z-index: 5;
    max-width: 960px;
    margin: 0 auto;
  }

  /* 头部：深海明珠 */
  .voice-header {
    text-align: center;
    margin-bottom: 40px;

    .pearl {
      display: inline-block;
      background: rgba(6, 10, 20, 0.6);
      backdrop-filter: blur(2px);
      border: 1px solid $glass-edge;
      border-radius: 120px;
      padding: 20px 48px;
      box-shadow: 0 0 50px rgba($accent-aqua, 0.2),
        0 20px 40px rgba(0, 0, 0, 0.4);
      position: relative;
      overflow: hidden;

      &::before {
        content: "";
        position: absolute;
        inset: 0;
        background: radial-gradient(
          circle at 30% 30%,
          rgba($accent-lavender, 0.2),
          transparent 70%
        );
        pointer-events: none;
      }

      h1 {
        margin: 0;
        font-size: 2rem;
        font-weight: 400;
        background: linear-gradient(
          135deg,
          #ffffff,
          $accent-lavender,
          $accent-aqua
        );
        -webkit-background-clip: text;
        background-clip: text;
        color: transparent;
        letter-spacing: 4px;
        font-family: "Times New Roman", serif;
      }

      .subtitle {
        margin: 8px 0 0;
        font-size: 1rem;
        color: rgba($text-light, 0.8);
        font-style: italic;
        font-weight: 300;
      }
    }

    .header-decoration {
      display: flex;
      justify-content: center;
      gap: 16px;
      margin-top: 12px;
      span {
        width: 2px;
        height: 20px;
        background: linear-gradient(to bottom, $accent-aqua, transparent);
        border-radius: 1px;
        animation: glowPulse 3s infinite alternate;
        &:nth-child(1) {
          height: 30px;
          animation-delay: 0s;
        }
        &:nth-child(2) {
          height: 20px;
          animation-delay: 0.4s;
          background: $accent-lavender;
        }
        &:nth-child(3) {
          height: 25px;
          animation-delay: 0.8s;
        }
      }
    }
  }

  /* 语音列表 */
  .voice-list {
    list-style: none;
    padding: 0;
    margin: 0;
    max-height: calc(100vh - 280px);
    overflow-y: auto;
    padding-right: 8px;

    &::-webkit-scrollbar {
      width: 6px;
    }
    &::-webkit-scrollbar-thumb {
      background: $accent-aqua;
      border-radius: 6px;
    }
  }

  .voice-item {
    margin-bottom: 16px;

    .item-inner {
      background: rgba(6, 10, 20, 0.5);
      backdrop-filter: blur(2px);
      border: 1px solid $glass-edge;
      border-radius: 40px 40px 30px 30px;
      padding: 16px 20px;
      position: relative;
      overflow: hidden;
      transition: all 0.3s;

      &::before {
        content: "";
        position: absolute;
        top: -10px;
        left: 10%;
        width: 80%;
        height: 30px;
        background: radial-gradient(
          ellipse at center,
          rgba($accent-aqua, 0.15),
          transparent 70%
        );
        filter: blur(10px);
        pointer-events: none;
      }

      &:hover {
        transform: translateY(-4px);
        border-color: rgba($accent-aqua, 0.4);
        box-shadow: 0 20px 30px rgba($accent-aqua, 0.2);
      }
    }

    &.playing .item-inner {
      border-color: $accent-aqua;
      box-shadow: 0 0 30px rgba($accent-aqua, 0.3);
    }

    .item-left {
      display: flex;
      gap: 16px;
      align-items: center;
      margin-bottom: 12px;

      .index-disc {
        width: 60px;
        height: 60px;
        border-radius: 40% 60% 40% 60%;
        background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 1.2rem;
        font-weight: 600;
        color: $deep-bg;
        box-shadow: 0 0 20px rgba($accent-aqua, 0.3);
      }

      .info .name {
        font-size: 1.1rem;
        font-weight: 400;
        color: $accent-aqua;
        letter-spacing: 0.5px;
      }
    }

    .item-right {
      display: flex;
      gap: 12px;
      align-items: center;
      justify-content: flex-end;

      .play-btn {
        width: 52px;
        height: 52px;
        border-radius: 40% 60% 40% 60%;
        background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
        border: none;
        color: $deep-bg;
        font-size: 1.4rem;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        box-shadow: 0 0 20px rgba($accent-aqua, 0.3);
        transition: all 0.2s;

        &:hover {
          transform: scale(1.1);
          box-shadow: 0 0 30px rgba($accent-aqua, 0.6);
        }
      }

      .download-btn {
        background: transparent !important;
        border: 1px solid $glass-edge !important;
        color: $accent-aqua !important;
        box-shadow: 0 0 15px rgba($accent-aqua, 0.2);
        transition: all 0.2s;

        &:hover {
          border-color: $accent-aqua !important;
          transform: translateY(-2px);
          box-shadow: 0 0 25px rgba($accent-aqua, 0.4);
        }
      }
    }

    .item-depth {
      height: 16px;
      background: linear-gradient(
        to bottom,
        transparent,
        rgba($accent-deep-blue, 0.2)
      );
      margin: 12px -20px -16px -20px;
      border-radius: 0 0 30px 30px;
    }
  }

  /* 动画 */
  @keyframes glowPulse {
    0% {
      opacity: 0.3;
      transform: scaleY(1);
    }
    100% {
      opacity: 0.9;
      transform: scaleY(1.3);
    }
  }

  .bg-fade-enter-active,
  .bg-fade-leave-active {
    transition: opacity 2s ease-in-out;
  }
  .bg-fade-enter-from,
  .bg-fade-leave-to {
    opacity: 0;
  }
  .bg-fade-enter-to,
  .bg-fade-leave-from {
    opacity: 0.5;
  }

  /* 移动端适配 */
  @media (max-width: 720px) {
    padding: 60px 12px 40px;

    .carousel-desktop {
      display: none;
    }
    .carousel-mobile {
      display: block;
    }

    .voice-header .pearl {
      padding: 16px 24px;
      h1 {
        font-size: 1.5rem;
      }
    }

    .voice-item .item-left {
      .index-disc {
        width: 48px;
        height: 48px;
        font-size: 1rem;
      }
      .info .name {
        font-size: 1rem;
      }
    }

    .item-right {
      .play-btn {
        width: 44px;
        height: 44px;
        font-size: 1.2rem;
      }
    }
  }
}
</style>
