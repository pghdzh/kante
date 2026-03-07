<template>
  <div id="app">
    <transition name="fade" v-if="showIntro">
      <!-- 坎特蕾拉 · 深海幻梦开屏 -->
      <div class="intro-container" @click="showIntro = false">
        <!-- 背景视频层 (调高亮度) -->
        <video
          class="video-background"
          :src="videoSrc"
          autoplay
          muted
          loop
          playsinline
        ></video>

        <!-- 光束效果：海面洒下的光柱 (新增) -->
        <div class="light-beams" aria-hidden="true">
          <span v-for="n in 5" :key="n" class="beam"></span>
        </div>

        <!-- 浮游水母群 (动态增强) -->
        <div class="jellyfish-swarm" aria-hidden="true">
          <span v-for="n in 8" :key="n" class="jelly"></span>
        </div>

        <!-- 角色标志性元素：珊瑚骨洋伞 (左侧) + 秘药瓶 (右侧) (新增) -->
        <div class="symbol-left" aria-hidden="true">
          <span class="umbrella"><img src="@/assets/violet.png" alt="" /></span>
          <span class="tentacles"></span>
          <span class="label">珊瑚骨洋伞</span>
        </div>
        <div class="symbol-right" aria-hidden="true">
          <span class="phial"><img src="@/assets/media.png" alt="" /></span>
          <span class="glow"></span>
          <span class="label">秘药·紫绒梦</span>
        </div>

        <!-- 主视觉内容 -->
        <div class="intro-content" aria-live="polite">
          <div class="intro-inner">
            <!-- 打字机：坎特蕾拉语录 -->
            <div class="typewriter-wrap">
              <p class="typewriter">
                {{ displayText }}
              </p>
            </div>
          </div>

          <!-- 角色短签 (增加蓝宝石元素) -->
          <div class="insignia">
            <span class="badge"> <i class="gem">💎</i> 翡萨烈家主 </span>
            <span class="badge venom">「毒药」坎特蕾拉</span>
            <span class="badge dream">「幻梦」编织者</span>
          </div>

          <!-- 底部渐隐光带 (新增) -->
          <div class="light-bar"></div>
        </div>
      </div>
    </transition>

    <!-- 主应用 -->
    <div v-else>
      <navbar />
      <main class="main-content">
        <RouterView />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import { RouterView } from "vue-router";
import navbar from "@/components/navbar.vue";

const showIntro = ref(true);
const videoSrc = ref("");

// 坎特蕾拉专属文案 (增加更多角色语录)
const lines = [
  "“如此宁静祥和的日常，便是你创造的奇迹”",
  "“是毒药，还是解药，全凭你的用法哦。”",
  "“水母群从伞下涌出，翩飞成一片幻影之海。”",
  "“用自己的灵魂熬制一锅独一无二的秘药。”",
  "“我喜欢你凝视我的样子，和你在一起的时候，我不再是「毒药」，而是我自己。”",
  "“微光穿透深海，映照出隐现于汹涌浪潮之间的温柔呢喃……”",
  "“睡吧，睡吧，在昏昏沉沉中坠向深处。”",
  "“如今，镶嵌着蓝宝石的秘药瓶中盛放着难得的真心与柔情。”",
] as const;

const displayText = ref("");
let typingTimer: number | null = null;
const typingSpeed = 130;

function pickRandomLine(): string {
  const idx = Math.floor(Math.random() * lines.length);
  return lines[idx];
}

function startOnceType(line: string) {
  const reduce =
    window.matchMedia &&
    window.matchMedia("(prefers-reduced-motion: reduce)").matches;
  if (reduce) {
    displayText.value = line;
    return;
  }

  let i = 0;
  typingTimer = window.setInterval(() => {
    i++;
    displayText.value = line.slice(0, i);
    if (i >= line.length) {
      if (typingTimer) {
        clearInterval(typingTimer);
        typingTimer = null;
      }
    }
  }, typingSpeed);
}

onMounted(() => {
  // 背景视频
  const isMobile = window.innerWidth <= 768;
  const folder = isMobile ? "/mp2" : "/mp1";
  const index = Math.floor(Math.random() * 4) + 1;
  videoSrc.value = `${folder}/1 (${index}).mp4`;

  // 10秒后自动关闭 (稍长一点让用户欣赏)
  setTimeout(() => {
    showIntro.value = false;
  }, 6000);

  const line = pickRandomLine();
  setTimeout(() => startOnceType(line), 280);
});

onBeforeUnmount(() => {
  if (typingTimer) clearInterval(typingTimer);
});
</script>

<style scoped lang="scss">
#app {
  position: relative;
  min-height: 100vh;
  animation: cursorAnimation 1s infinite step-start;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 1.2s cubic-bezier(0.45, 0, 0.2, 1);
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* ===== 坎特蕾拉风格 V2：深海幻梦 · 光与水的诗 ===== */
$deep-start: #061020; // 深海起始 (稍提亮)
$deep-end: #142845; // 深海终点 (加入更多蓝)
$light-aqua: #7ae2ff; // 水母荧光 (提亮)
$accent-violet: #b39eff; // 淡紫罗兰 (提亮)
$accent-blue: #4ba0e8; // 蓝宝石色 (新增)
$gem-blue: #3478d6; // 深邃蓝宝石
$text-light: #f0f5fe;

.intro-container {
  position: fixed;
  inset: 0;
  background: radial-gradient(ellipse at 30% 30%, #1d3557 0%, $deep-start 80%);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;

  /* 背景视频 (调高亮度、降低对比度，让光束更明显) */
  .video-background {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    opacity: 0.5; // 从0.3→0.5 提高可见度
    z-index: 1;
    filter: hue-rotate(5deg) saturate(1.1) brightness(0.7); // 提高亮度
    pointer-events: none;
  }

  /* 新增：海面洒下的光束 */
  .light-beams {
    position: absolute;
    inset: 0;
    z-index: 2;
    pointer-events: none;
    .beam {
      position: absolute;
      top: -10%;
      width: 60px;
      height: 120%;
      background: linear-gradient(
        180deg,
        rgba(255, 255, 255, 0.15) 0%,
        rgba(122, 226, 255, 0.1) 40%,
        transparent 90%
      );
      transform: rotate(10deg);
      filter: blur(30px);
      animation: beamMove 12s infinite alternate ease-in-out;

      &:nth-child(1) {
        left: 10%;
        animation-duration: 14s;
        width: 80px;
      }
      &:nth-child(2) {
        left: 30%;
        animation-duration: 16s;
        width: 100px;
        opacity: 0.6;
      }
      &:nth-child(3) {
        left: 50%;
        animation-duration: 18s;
        width: 70px;
      }
      &:nth-child(4) {
        left: 70%;
        animation-duration: 15s;
        width: 90px;
        opacity: 0.5;
      }
      &:nth-child(5) {
        left: 90%;
        animation-duration: 17s;
        width: 60px;
      }
    }
  }

  /* 水母群 (提亮) */
  .jellyfish-swarm {
    position: absolute;
    inset: 0;
    z-index: 3;
    pointer-events: none;
    .jelly {
      position: absolute;
      width: 40px;
      height: 60px;
      background: radial-gradient(
        circle at 50% 30%,
        rgba(179, 158, 255, 0.4),
        rgba(122, 226, 255, 0.2) 60%,
        transparent 80%
      );
      border-radius: 60% 60% 40% 40%;
      filter: blur(5px);
      box-shadow: 0 0 30px rgba($light-aqua, 0.6);
      animation: floatJelly 16s infinite alternate ease-in-out;

      &:nth-child(1) {
        top: 15%;
        left: 8%;
        animation-duration: 18s;
        width: 50px;
      }
      &:nth-child(2) {
        bottom: 25%;
        right: 12%;
        animation-duration: 20s;
        width: 70px;
        height: 90px;
      }
      &:nth-child(3) {
        top: 45%;
        right: 20%;
        animation-duration: 22s;
        opacity: 0.7;
      }
      &:nth-child(4) {
        bottom: 35%;
        left: 25%;
        animation-duration: 16s;
        width: 45px;
      }
      &:nth-child(5) {
        top: 70%;
        left: 15%;
        animation-duration: 19s;
        opacity: 0.6;
      }
      &:nth-child(6) {
        top: 20%;
        right: 35%;
        animation-duration: 24s;
        width: 60px;
      }
      &:nth-child(7) {
        bottom: 15%;
        left: 40%;
        animation-duration: 21s;
        width: 55px;
      }
      &:nth-child(8) {
        top: 60%;
        right: 5%;
        animation-duration: 17s;
        width: 65px;
      }
    }
  }

  /* 新增：左侧珊瑚骨洋伞 (角色标志性物品) */
  .symbol-left {
    position: absolute;
    left: 5%;
    bottom: 15%;
    z-index: 5;
    display: flex;
    flex-direction: column;
    align-items: center;
    color: $light-aqua;
    opacity: 0.7;
    filter: drop-shadow(0 0 20px rgba($light-aqua, 0.8));
    animation: symbolFloat 6s infinite ease-in-out;

    .umbrella {
      font-size: 5rem;
      transform: rotate(-10deg);
      line-height: 1;
      filter: drop-shadow(0 10px 15px rgba(0, 40, 80, 0.5));
      img {
        width: 80px;
      }
    }
    .tentacles {
      font-size: 3rem;
      letter-spacing: 8px;
      margin-top: -20px;
      animation: sway 3s infinite;
    }
    .label {
      margin-top: 10px;
      font-size: 0.8rem;
      background: rgba(10, 20, 40, 0.5);
      backdrop-filter: blur(4px);
      padding: 4px 12px;
      border-radius: 30px;
      border: 1px solid rgba($light-aqua, 0.4);
      color: rgba(255, 255, 255, 0.9);
      letter-spacing: 1px;
    }
  }

  /* 新增：右侧秘药瓶 (蓝宝石镶嵌) */
  .symbol-right {
    position: absolute;
    right: 5%;
    bottom: 15%;
    z-index: 5;
    display: flex;
    flex-direction: column;
    align-items: center;
    animation: symbolFloat 7s infinite ease-in-out reverse;

    .phial {
      font-size: 4.5rem;
      line-height: 1;
      filter: drop-shadow(0 0 25px $gem-blue);
      position: relative;
      img {
        width: 80px;
      }
    }
    .glow {
      position: absolute;
      width: 60px;
      height: 60px;
      background: radial-gradient(
        circle,
        rgba(52, 120, 214, 0.6) 0%,
        transparent 70%
      );
      border-radius: 50%;
      top: 20px;
      z-index: -1;
      animation: pulseGlow 3s infinite;
    }
    .label {
      margin-top: 10px;
      font-size: 0.8rem;
      background: rgba(10, 20, 40, 0.5);
      backdrop-filter: blur(4px);
      padding: 4px 12px;
      border-radius: 30px;
      border: 1px solid rgba($gem-blue, 0.5);
      color: rgba(255, 255, 255, 0.9);
      letter-spacing: 1px;
    }
  }

  /* 主内容区 (提高层级) */
  .intro-content {
    position: relative;
    z-index: 10;
    width: 100%;
    padding: 20px 30px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 40px;

    .intro-inner {
      display: flex;
      align-items: center;
      justify-content: center;
      max-width: 1000px;
      margin-top: -5vh;

      .typewriter-wrap {
        position: relative;
        display: inline-flex;
        align-items: center;
        max-width: 900px;
        padding: 20px 40px;
        background: rgba(6, 16, 32, 0.3);
        backdrop-filter: blur(10px);
        border-radius: 60px;
        border: 1px solid rgba($light-aqua, 0.3);
        box-shadow: 0 0 60px rgba($light-aqua, 0.3);

        .typewriter {
          margin: 0;
          font-weight: 600;
          font-size: clamp(1.8rem, 5vw, 3rem);
          line-height: 1.4;
          background: linear-gradient(
            135deg,
            #ffffff,
            $light-aqua,
            $accent-violet
          );
          -webkit-background-clip: text;
          background-clip: text;
          color: transparent;
          text-shadow: 0 0 30px rgba($light-aqua, 0.5);
          font-family: "Dancing Script", "Segoe Script", "Brush Script MT",
            cursive;
          white-space: pre-wrap;
          word-break: break-word;
        }
      }
    }

    /* 角色短签 (带蓝宝石图标) */
    .insignia {
      display: flex;
      flex-wrap: wrap;
      gap: 15px 25px;
      justify-content: center;
      margin-top: 20px;

      .badge {
        display: flex;
        align-items: center;
        gap: 6px;
        background: rgba(8, 18, 32, 0.6);
        backdrop-filter: blur(8px);
        border: 1px solid rgba($light-aqua, 0.3);
        border-radius: 40px;
        padding: 8px 28px;
        font-size: 1.1rem;
        font-weight: 500;
        color: rgba($text-light, 1);
        letter-spacing: 1px;
        box-shadow: 0 0 25px rgba($accent-violet, 0.3);

        .gem {
          font-size: 1.3rem;
          filter: drop-shadow(0 0 12px $gem-blue);
        }

        &.venom {
          border-color: rgba(255, 120, 180, 0.5);
          box-shadow: 0 0 25px rgba(255, 120, 180, 0.3);
        }
        &.dream {
          border-color: rgba($light-aqua, 0.6);
        }
      }
    }

    /* 新增：底部光带 */
    .light-bar {
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 100px;
      background: linear-gradient(
        180deg,
        transparent 0%,
        rgba(75, 160, 232, 0.2) 50%,
        rgba(122, 226, 255, 0.3) 100%
      );
      filter: blur(30px);
      z-index: -1;
      pointer-events: none;
    }
  }
}

/* 动画定义 */
@keyframes floatJelly {
  0% {
    transform: translate(0, 0) scale(1);
    opacity: 0.4;
  }
  50% {
    transform: translate(-25px, -35px) scale(1.2);
    opacity: 0.8;
  }
  100% {
    transform: translate(25px, 25px) scale(0.9);
    opacity: 0.4;
  }
}

@keyframes beamMove {
  0% {
    transform: rotate(8deg) translateX(-10px);
    opacity: 0.2;
  }
  50% {
    transform: rotate(12deg) translateX(10px);
    opacity: 0.5;
  }
  100% {
    transform: rotate(8deg) translateX(-5px);
    opacity: 0.3;
  }
}

@keyframes sway {
  0% {
    transform: translateX(0) rotate(0deg);
  }
  50% {
    transform: translateX(8px) rotate(3deg);
  }
  100% {
    transform: translateX(0) rotate(0deg);
  }
}

@keyframes symbolFloat {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-15px);
  }
  100% {
    transform: translateY(0);
  }
}

@keyframes pulseGlow {
  0% {
    opacity: 0.3;
    transform: scale(0.8);
  }
  50% {
    opacity: 0.8;
    transform: scale(1.2);
  }
  100% {
    opacity: 0.3;
    transform: scale(0.8);
  }
}

/* 移动端适配 */
@media (max-width: 720px) {
  .intro-container {
    .light-beams .beam {
      width: 30px;
      filter: blur(20px);
    }

    .symbol-left,
    .symbol-right {
      bottom: 8%;
      .umbrella {
        font-size: 3.5rem;
      }
      .phial {
        font-size: 3rem;
      }
      .label {
        font-size: 0.7rem;
        padding: 2px 8px;
      }
    }

    .intro-inner {
      .typewriter-wrap {
        padding: 15px 20px;
        .typewriter {
          font-size: 1.5rem;
        }
      }
    }

    .insignia .badge {
      padding: 5px 16px;
      font-size: 0.9rem;
    }
  }
}
</style>
