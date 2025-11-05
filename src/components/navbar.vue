<template>
  <header class="app-header">
    <h1 class="title">坎特蕾拉电子设定集</h1>
    <!-- 在线人数展示 -->
    <div class="online-count" v-if="onlineCount !== null">
      当前在线：<span class="count">{{ onlineCount }}人</span>
    </div>
    <!-- 移动端汉堡按钮 -->
    <button
      class="hamburger"
      @click="toggleMobileNav"
      aria-label="Toggle navigation"
    >
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
    </button>

    <!-- 普通导航 & 移动端下拉导航 -->
    <nav :class="['nav-links', { 'mobile-open': mobileNavOpen }]">
      <RouterLink
        v-for="item in navItems"
        :key="item.name"
        :to="item.path"
        class="nav-item"
        active-class="active-link"
        @click="mobileNavOpen = false"
      >
        {{ item.name }}
      </RouterLink>

      <a
        href="http://slty.site/#/redirector"
        target="_blank"
        rel="noopener"
        class="nav-item"
        active-class="active-link"
        @click="mobileNavOpen = false"
      >
        霜落映界
      </a>
    </nav>
  </header>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import { io } from "socket.io-client";

const navItems = [
  { name: "深海梦域", path: "/" }, // 首页 - 坎特蕾拉的深海梦境领域
  { name: "时潮韵律", path: "/timeLine" }, // 年谱 - 时间如潮汐般起伏的韵律
  { name: "涟漪心语", path: "/message" }, // 留言板 - 如水面涟漪般扩散的心声
  { name: "蜃景残像", path: "/gallery" }, // 图集 - 海市蜃楼中定格的残像
  { name: "渊海秘录", path: "/resources" }, // 资料库 - 深渊之海中的秘密记录
  { name: "梦语低吟", path: "/voice" }, // 语音馆 - 梦中传来的温柔低语
  { name: "韵律回廊", path: "/music" }, // 歌曲库 - 充满韵律的梦境回廊
];

const mobileNavOpen = ref(false);
function toggleMobileNav() {
  mobileNavOpen.value = !mobileNavOpen.value;
}

const siteId = "kante";

const onlineCount = ref<number | null>(null);

// 连接时带上 query.siteId
const socket: any = io("http://36.150.237.25:3000", {
  query: { siteId },
});

onMounted(() => {
  socket.on("onlineCount", (count: number) => {
    onlineCount.value = count;
  });
});

onBeforeUnmount(() => {
  socket.disconnect();
});
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Cinzel:ital,wght@0,400;1,700&family=ZCOOL+QingKe+HuangYou&display=swap");

.app-header {
  /* 基于坎特蕾拉：水母 / 紫蓝海感 + 毒药高光 */
  --deep-bg: rgba(4, 6, 12, 0.9); // 更深夜海底
  --glass-accent: rgba(58, 34, 120, 0.06); // 暗紫蓝玻璃感
  --accent: #7a5fe6; // 暗紫（主光）
  --accent-2: #5fe0ff; // 冷海蓝（辅助高光）
  --venom: #ff6b85; // 毒色点缀（保留作对比）
  --muted-text: #efeef2; // 微冷象牙
  --wet-sheen: rgba(255, 255, 255, 0.035); // 湿光高光覆盖
  --bubble: rgba(95, 224, 255, 0.06); // 泡沫微光

  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  height: 64px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 40px;
  background: radial-gradient(
      600px 120px at 10% 10%,
      rgba(95, 224, 255, 0.02),
      transparent 8%
    ),
    linear-gradient(180deg, rgba(6, 8, 14, 0.95), rgba(8, 6, 16, 0.94));
  backdrop-filter: blur(6px) saturate(0.98);
  box-shadow: 0 8px 36px rgba(3, 4, 6, 0.6), 0 0 18px var(--glass-accent) inset;
  border-bottom: 1px solid rgba(122, 95, 230, 0.035);
  animation: fadeInDown 0.6s ease-out both;
  overflow: visible;

  /* 轻微的表面“湿感”覆盖（伪元素） */
  &::after {
    content: "";
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    height: 100%;
    pointer-events: none;
    background: linear-gradient(180deg, var(--wet-sheen), transparent 20%);
    mix-blend-mode: overlay;
  }

  .title {
    position: relative;
    font-family: "Cinzel", serif;
    font-style: italic;
    font-size: 26px;
    font-weight: 700;
    color: var(--muted-text);
    background: linear-gradient(90deg, var(--accent), var(--accent-2));
    background-clip: text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    letter-spacing: 0.4px;
    text-shadow: 0 6px 22px rgba(6, 6, 10, 0.55), 0 1px 0 rgba(0, 0, 0, 0.28);
    transition: transform 0.28s ease, text-shadow 0.28s ease;
    animation: float-slow 10s ease-in-out infinite;

    /* 文字上加薄薄“水波”动感遮罩（伪元素）*/
    &::before {
      content: "";
      position: absolute;
      left: -6%;
      top: -18%;
      width: 120%;
      height: 140%;
      background: radial-gradient(
        circle at 50% 20%,
        rgba(255, 255, 255, 0.02),
        transparent 18%
      );
      transform: translateY(0);
      animation: shimmer 6s linear infinite;
      pointer-events: none;
      mix-blend-mode: screen;
    }

    &:hover {
      transform: translateY(-2px) scale(1.03);
      text-shadow: 0 10px 36px rgba(122, 95, 230, 0.14),
        0 2px 0 rgba(0, 0, 0, 0.26);
    }
  }

  .online-count {
    position: relative;
    margin-left: 16px;
    padding: 6px 14px;
    font-family: "Cinzel Decorative", serif;
    font-size: 1rem;
    color: var(--muted-text);
    background: linear-gradient(
      135deg,
      rgba(12, 8, 20, 0.26),
      rgba(8, 6, 18, 0.22)
    );
    border: 1px solid rgba(95, 224, 255, 0.04);
    border-radius: 24px;
    backdrop-filter: blur(8px);
    box-shadow: 0 8px 22px rgba(3, 4, 6, 0.5),
      0 0 12px rgba(58, 34, 120, 0.03) inset;
    overflow: hidden;
    cursor: default;
    transition: transform 0.22s ease, box-shadow 0.22s ease;

    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 14px 36px rgba(3, 4, 6, 0.56), 0 0 36px var(--bubble);
    }

    .count {
      display: inline-block;
      margin-left: 18px;
      font-weight: 700;
      color: var(--accent-2);
      background: linear-gradient(90deg, var(--accent), var(--accent-2));
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      text-shadow: 0 0 10px rgba(95, 224, 255, 0.06);
    }
  }

  .nav-links {
    display: flex;
    gap: 22px;
    align-items: center;

    .nav-item {
      position: relative;
      color: var(--muted-text);
      font-weight: 500;
      text-decoration: none;
      transition: color 0.22s ease, transform 0.16s ease;
      padding: 6px 4px;
      border-radius: 6px;
      overflow: visible;
      font-family: "STKaiti", "KaiTi", "Noto Serif SC", serif;
      font-style: italic;

      &::after {
        content: "";
        position: absolute;
        left: 50%;
        bottom: -8px;
        width: 0;
        height: 6px;
        border-radius: 6px;
        background: linear-gradient(
          90deg,
          rgba(0, 0, 0, 0),
          rgba(122, 95, 230, 0.9),
          rgba(95, 224, 255, 0.85),
          rgba(0, 0, 0, 0)
        );
        transform: translateX(-50%);
        opacity: 0.95;
        filter: blur(0.8px) contrast(1.03);
        transition: width 0.36s cubic-bezier(0.2, 0.9, 0.2, 1), left 0.36s,
          opacity 0.24s;
        background-size: 200% 100%;
        animation: flow-wave 6.5s linear infinite;
      }

      /* 悬停增加“毒液滴”效果（微交互） */
      &::before {
        content: "";
        position: absolute;
        right: 10%;
        top: -6px;
        width: 6px;
        height: 6px;
        border-radius: 50%;
        background: linear-gradient(180deg, var(--accent), var(--accent-2));
        opacity: 0;
        transform: translateY(-6px);
        transition: opacity 0.26s, transform 0.36s;
        box-shadow: 0 4px 8px rgba(255, 107, 133, 0.08);
      }

      &:hover {
        color: var(--accent-2);
        transform: translateY(-1.8px);
        text-shadow: 0 0 8px rgba(95, 224, 255, 0.04);
      }

      &:hover::after {
        width: 72%;
        left: 50%;
        opacity: 1;
      }

      &:hover::before {
        opacity: 1;
        transform: translateY(0);
      }
    }

    .active-link {
      color: var(--accent);
      font-weight: 600;

      &::before {
        content: "♪";
        position: absolute;
        right: -6px;
        top: 50%;
        transform: translateY(-50%);
        font-size: 12px;
        color: var(--accent);
        text-shadow: 0 2px 8px rgba(122, 95, 230, 0.12);
        animation: note-float 3.6s ease-in-out infinite;
        opacity: 0.95;
      }

      &::after {
        width: 92%;
        opacity: 1;
        box-shadow: 0 6px 22px rgba(122, 95, 230, 0.08);
      }
    }
  }

  .hamburger {
    display: none;
    flex-direction: column;
    justify-content: space-around;
    width: 28px;
    height: 24px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;

    span {
      display: block;
      width: 100%;
      height: 3px;
      background: rgba(239, 233, 236, 0.92);
      border-radius: 2px;
      transition: transform 0.28s ease, opacity 0.28s ease, background 0.28s;
      box-shadow: 0 2px 6px rgba(8, 6, 10, 0.18),
        0 0 8px rgba(58, 34, 120, 0.02);
    }

    span.open:nth-child(1) {
      transform: translateY(10px) rotate(45deg);
      background: linear-gradient(90deg, var(--accent), var(--accent-2));
    }

    span.open:nth-child(2) {
      opacity: 0;
    }

    span.open:nth-child(3) {
      transform: translateY(-10px) rotate(-45deg);
      background: linear-gradient(90deg, var(--accent), var(--accent-2));
    }
  }

  @media (max-width: 768px) {
    padding: 0 20px;

    .title {
      display: none;
    }
    .hamburger {
      display: flex;
    }

    .nav-links {
      position: absolute;
      top: 64px;
      left: 0;
      right: 0;
      flex-direction: column;
      background: linear-gradient(
        180deg,
        rgba(8, 6, 12, 0.98),
        rgba(6, 4, 10, 0.995)
      );
      backdrop-filter: blur(12px);
      gap: 0;
      overflow: hidden;
      max-height: 0;
      transition: max-height 0.34s ease;
      border-top: 1px solid rgba(95, 224, 255, 0.03);

      &.mobile-open {
        max-height: 520px;
      }

      .nav-item {
        padding: 14px 20px;
        border-bottom: 1px solid rgba(255, 255, 255, 0.03);
      }
    }
  }
}

/* 动效关键帧 */
@keyframes flow-wave {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

@keyframes float-slow {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-3.5px);
  }
  100% {
    transform: translateY(0);
  }
}

@keyframes fadeInDown {
  0% {
    opacity: 0;
    transform: translateY(-8px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes note-float {
  0% {
    transform: translateY(-6%) rotate(-6deg);
    opacity: 0.85;
  }
  50% {
    transform: translateY(6%) rotate(2deg);
    opacity: 1;
  }
  100% {
    transform: translateY(-6%) rotate(-6deg);
    opacity: 0.85;
  }
}

/* 额外：气泡上升 & 文字光泽 */
@keyframes bubble-rise {
  0% {
    transform: translateY(6px) translateX(0);
    opacity: 0.08;
  }
  50% {
    opacity: 0.16;
    transform: translateY(-6px) translateX(8px);
  }
  100% {
    transform: translateY(-22px) translateX(0);
    opacity: 0.02;
  }
}

@keyframes shimmer {
  0% {
    transform: translateY(0) rotate(0deg);
    opacity: 0.6;
  }
  50% {
    transform: translateY(-6px) rotate(1deg);
    opacity: 1;
  }
  100% {
    transform: translateY(0) rotate(0deg);
    opacity: 0.6;
  }
}
</style>
