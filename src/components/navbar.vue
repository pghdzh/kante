<template>
  <header class="cantarella-header">
    <!-- 背景的幻梦水母层 (装饰用) -->
    <div class="jellyfish-bg">
      <span class="jelly"></span><span class="jelly"></span
      ><span class="jelly"></span>
    </div>

    <!-- 左侧：珊瑚骨洋伞与家主之名 -->
    <div class="brand">
      <!-- 伞icon: 象征她的武器“珊瑚骨洋伞” [citation:4] -->
      <span class="umbrella-icon">☂︎</span>
      <h1 class="title">
        <span class="title-main">坎特蕾拉</span>
        <span class="title-sub">·电子设定集·</span>
      </h1>
    </div>

    <!-- 中央/右侧导航 (桌面) -->
    <nav class="nav-links" :class="{ 'mobile-open': mobileNavOpen }">
      <RouterLink
        v-for="item in navItems"
        :key="item.name"
        :to="item.path"
        class="nav-item"
        active-class="active-link"
        @click="mobileNavOpen = false"
      >
        <!-- 毒液滴点缀 + 水母触须效果在css里 -->
        <span class="item-text">{{ item.name }}</span>
        <span class="venom-drop"></span>
      </RouterLink>

      <!-- 外链：霜落映界 (保留) -->
      <a
        href="http://slty.site/#/redirector"
        target="_blank"
        rel="noopener"
        class="nav-item external"
        @click="mobileNavOpen = false"
      >
        <span class="item-text">霜落映界</span>
        <span class="venom-drop"></span>
      </a>

      <!-- 在线人数 - 嵌入在导航旁，像一个秘药瓶 [citation:4] -->
      <div class="online-phial" v-if="onlineCount !== null">
        <span class="phial-glow"></span>
        <span class="label">梦海共潮</span>
        <span class="count">{{ onlineCount }}</span>
        <span class="unit">人</span>
      </div>
      <!-- 音乐播放按钮（秘药瓶 + 水母） -->
      <button
        class="music-phial"
        @click="togglePlay"
        :disabled="audioLoading"
        :title="
          audioLoading ? '幻梦加载中' : isPlaying ? '暂停幻梦' : '奏响幻梦'
        "
      >
        <!-- 瓶身光晕 -->
        <span class="phial-glow"></span>
        <!-- 水母伞盖（装饰） -->
        <span class="jelly-umbrella"></span>
        <!-- 主图标：根据状态变化 -->
        <span
          class="phial-icon"
          :class="{ loading: audioLoading, playing: isPlaying }"
        >
          <span v-if="audioLoading" class="loader"></span>
          <span v-else-if="isPlaying">❚❚</span>
          <span v-else>♪</span>
        </span>
        <!-- 状态文字 -->
        <span class="status">{{
          audioLoading ? "入梦" : isPlaying ? "梦醒" : "入梦"
        }}</span>
        <!-- 漂浮水母触须（动态） -->
        <span class="jelly-tentacle" v-for="n in 3" :key="n"></span>
        <!-- 底部触须光晕 -->
        <span class="tentacle-glow"></span>
      </button>
    </nav>

    <!-- 移动端汉堡菜单 (水母伞造型) -->
    <button class="hamburger" @click="toggleMobileNav" aria-label="toggle menu">
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
      <!-- 伞顶小水母 -->
      <span class="jelly-tassel"></span>
    </button>
  </header>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import { io } from "socket.io-client";

const navItems = [
  { name: "翡萨烈幻梦", path: "/" },
  { name: "潮痕纪事", path: "/timeLine" },
  { name: "涟漪心语", path: "/message" },
  { name: "幻梦画廊", path: "/gallery" },
  { name: "渊海问答", path: "/talk" },
  { name: "秘药典籍", path: "/resources" },
  { name: "深梦呢喃", path: "/voice" },
  { name: "海潮乐章", path: "/music" },
  { name: "残章手札", path: "/wiki" },
];

const mobileNavOpen = ref(false);
function toggleMobileNav() {
  mobileNavOpen.value = !mobileNavOpen.value;
}

const siteId = "kante";
const onlineCount = ref<number | null>(null);
const socket: any = io("http://36.150.237.25:3000", {
  query: { siteId },
});
// 音乐播放控制（增强版）
const audio = ref<HTMLAudioElement | null>(null);
const isPlaying = ref(false);
const audioLoading = ref(false); // 加载状态

// 三首候选曲目（坎特蕾拉风格）
const musicList = [
  "http://36.150.237.25:3000/music/悠忽舞于梦中.mp3",
  "http://36.150.237.25:3000/music/沉沦幻海（伴奏）.mp3",
  "http://36.150.237.25:3000/music/Hush.mp3",
];
const currentMusic = ref(
  musicList[Math.floor(Math.random() * musicList.length)]
);

function togglePlay() {
  if (audioLoading.value) return;

  if (!audio.value) {
    // 首次点击：创建 audio 并加载
    audio.value = new Audio(currentMusic.value);
    audio.value.loop = false;
    audio.value.preload = "auto";

    // 监听加载事件
    audio.value.addEventListener("canplaythrough", () => {
      audioLoading.value = false;
    });
    audio.value.addEventListener("error", () => {
      audioLoading.value = false;
      alert("幻梦加载失败，请稍后重试");
    });

    audioLoading.value = true;
  }

  if (isPlaying.value) {
    audio.value.pause();
    isPlaying.value = false;
  } else {
    audio.value
      .play()
      .then(() => {
        isPlaying.value = true;
        audioLoading.value = false; // 确保播放后加载状态关闭
      })
      .catch((e) => {
        console.warn("播放被阻止", e);
        audioLoading.value = false;
      });
  }
}

onMounted(() => {
  socket.on("onlineCount", (count: number) => {
    onlineCount.value = count;
  });
});

onBeforeUnmount(() => {
  socket.disconnect();
  audio.value?.pause();
  audio.value = null;
});
</script>

<style scoped lang="scss">
// 引入高贵衬线字体，符合“神秘、高贵、剧毒”气质 [citation:4]
@import url("https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,600;1,500&family=ZCOOL+QingKe+HuangYou&display=swap");

.cantarella-header {
  // ---------- 坎特蕾拉主题色：深海紫罗兰 + 毒药粉 + 水母荧光 [citation:1][citation:4] ----------
  --deep-violet: #1f0e3c; // 深渊底色
  --venom-pink: #ff6b9d; // 毒药/幻觉毒色泽 [citation:1]
  --jelly-blue: #5ac8fa; // 水母触手荧光
  --abyss-glow: #7d4ec7; // 紫罗兰高光
  --pearl-mist: #f0eaf2; // 泡沫白
  --shadow-abyss: rgba(7, 2, 19, 0.9);
  --glass-effect: rgba(210, 180, 255, 0.06);

  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  height: 72px;
  padding: 0 48px;
  display: flex;
  align-items: center;
  justify-content: space-between;

  // 背景：深海漩涡 + 微光
  background: radial-gradient(circle at 30% 30%, #2a144a, #0c0520 80%);
  backdrop-filter: blur(10px) saturate(180%);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.7),
    0 0 0 1px rgba(90, 200, 250, 0.1) inset;
  border-bottom: 1px solid rgba(255, 107, 157, 0.2);

  // 浮动水母群背景 (纯装饰，不影响点击)
  .jellyfish-bg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    overflow: hidden;
    .jelly {
      position: absolute;
      width: 40px;
      height: 60px;
      background: radial-gradient(
        circle at 30% 40%,
        rgba(90, 200, 250, 0.15),
        transparent 70%
      );
      border-radius: 60% 40% 50% 30% / 40% 50% 40% 50%;
      filter: blur(8px);
      animation: floatJelly 18s infinite alternate ease-in-out;
      &:nth-child(1) {
        top: 5%;
        left: 15%;
        width: 70px;
        height: 90px;
        background: rgba(200, 150, 255, 0.08);
        animation-duration: 22s;
      }
      &:nth-child(2) {
        bottom: -10%;
        right: 20%;
        width: 120px;
        height: 160px;
        background: rgba(255, 107, 157, 0.06);
        animation-duration: 28s;
        animation-delay: -5s;
      }
      &:nth-child(3) {
        top: 40%;
        right: 5%;
        width: 50px;
        height: 70px;
        background: rgba(90, 200, 250, 0.1);
        animation-duration: 14s;
      }
    }
  }

  // 品牌/标题区域 —— 她的洋伞与名字 [citation:4]
  .brand {
    display: flex;
    align-items: center;
    gap: 8px;
    position: relative;
    z-index: 5;
    .umbrella-icon {
      font-size: 2.4rem;
      line-height: 1;
      color: var(--jelly-blue);
      text-shadow: 0 0 15px var(--venom-pink), 0 0 30px #a57cff;
      transform: rotate(-10deg);
      filter: drop-shadow(0 4px 6px #00000060);
      transition: transform 0.3s ease;
    }
    .title {
      font-family: "Playfair Display", "Cinzel", serif;
      font-weight: 600;
      letter-spacing: 0.06em;
      .title-main {
        font-size: 1.8rem;
        background: linear-gradient(
          135deg,
          #fff6f0,
          var(--jelly-blue),
          var(--venom-pink)
        );
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        background-clip: text;
        text-shadow: 0 0 30px rgba(255, 107, 157, 0.5);
        margin-right: 6px;
      }
      .title-sub {
        font-size: 0.9rem;
        font-style: italic;
        color: rgba(255, 255, 255, 0.65);
        text-shadow: 0 0 10px var(--jelly-blue);
        letter-spacing: 0.2rem;
      }
    }
    &:hover .umbrella-icon {
      transform: rotate(0deg) scale(1.05);
    }
  }

  // 导航区域 + 秘药瓶在线显示
  .nav-links {
    display: flex;
    align-items: center;
    gap: 12px;
    position: relative;
    z-index: 5;

    .nav-item {
      position: relative;
      padding: 8px 14px;
      color: var(--pearl-mist);
      text-decoration: none;
      font-family: "Playfair Display", serif;
      font-weight: 500;
      font-size: 1.1rem;
      letter-spacing: 0.02em;
      transition: all 0.25s;
      border-radius: 30px;
      white-space: nowrap;

      // 毒液滴 (坎特蕾拉的幻觉毒) [citation:1]
      .venom-drop {
        position: absolute;
        right: -2px;
        top: -4px;
        width: 10px;
        height: 10px;
        border-radius: 50%;
        background: var(--venom-pink);
        filter: blur(2px);
        opacity: 0;
        transition: opacity 0.3s, transform 0.3s;
        box-shadow: 0 0 12px #ff3b7a;
        &::after {
          content: "";
          position: absolute;
          top: 5px;
          left: 2px;
          width: 4px;
          height: 8px;
          background: var(--venom-pink);
          border-radius: 50%;
          filter: blur(3px);
        }
      }

      // 水母触须下划线 (幻觉与毒的交织)
      &::after {
        content: "";
        position: absolute;
        left: 50%;
        bottom: 0;
        width: 0;
        height: 2px;
        background: linear-gradient(
          90deg,
          transparent,
          var(--jelly-blue),
          var(--venom-pink),
          transparent
        );
        transform: translateX(-50%);
        transition: width 0.3s cubic-bezier(0.2, 1, 0.3, 1);
        border-radius: 2px;
        filter: drop-shadow(0 0 6px #5ac8fa);
      }

      &:hover {
        color: white;
        transform: translateY(-2px);
        text-shadow: 0 0 14px var(--jelly-blue), 0 0 30px var(--venom-pink);
        .venom-drop {
          opacity: 0.9;
          transform: scale(1.2);
        }
        &::after {
          width: 80%;
        }
      }

      // 激活项：毒素弥漫 + 音符飘浮 (结合梦中低语) [citation:3]
      &.active-link {
        color: white;
        font-weight: 600;
        background: rgba(255, 107, 157, 0.08);
        box-shadow: 0 0 20px rgba(90, 200, 250, 0.2);

        .venom-drop {
          opacity: 1;
          animation: dropPulse 2s infinite;
        }
        &::before {
          content: "♫";
          position: absolute;
          left: -4px;
          top: -2px;
          font-size: 14px;
          color: var(--jelly-blue);
          filter: drop-shadow(0 0 5px hotpink);
          animation: floatNote 3s infinite;
        }
        &::after {
          width: 85%;
          background: linear-gradient(
            90deg,
            transparent,
            var(--venom-pink),
            var(--jelly-blue),
            transparent
          );
          height: 3px;
          filter: blur(1px);
        }
      }
    }

    // 秘药瓶在线人数 [citation:4]  ———— 翡萨烈家主的秘药瓶
    .online-phial {
      margin-left: 20px;
      display: flex;
      align-items: baseline;
      gap: 6px;
      background: linear-gradient(145deg, #23153c, #130b24);
      padding: 6px 18px 6px 16px;
      border-radius: 40px;
      border: 1px solid rgba(90, 200, 250, 0.3);
      box-shadow: 0 0 20px rgba(255, 107, 157, 0.3), inset 0 0 10px #2f1b4e;
      position: relative;
      backdrop-filter: blur(4px);
      .phial-glow {
        position: absolute;
        inset: 0;
        border-radius: 40px;
        background: radial-gradient(
          circle at 30% 50%,
          rgba(255, 200, 230, 0.2),
          transparent 70%
        );
        pointer-events: none;
      }
      .label {
        font-size: 0.85rem;
        color: #b9b0d4;
        font-style: italic;
        letter-spacing: 0.1rem;
      }
      .count {
        font-size: 1.5rem;
        font-weight: 700;
        background: linear-gradient(135deg, #ffe6f0, var(--jelly-blue));
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        line-height: 1;
        text-shadow: 0 0 15px cyan;
      }
      .unit {
        font-size: 0.9rem;
        color: var(--venom-pink);
        opacity: 0.9;
      }
    }
    /* 音乐播放按钮（秘药瓶 + 水母）- 替换原有样式 */
    .music-phial {
      position: relative;
      display: flex;
      align-items: center;
      gap: 8px;
      background: linear-gradient(145deg, #1f2b4a, #0e1a2f);
      padding: 6px 18px 6px 16px;
      border-radius: 40px;
      border: 1px solid rgba(90, 200, 250, 0.4);
      box-shadow: 0 0 20px rgba(255, 107, 157, 0.3), inset 0 0 10px #2a3a5a;
      backdrop-filter: blur(4px);
      cursor: pointer;
      transition: all 0.3s ease;
      margin-left: 8px;
      overflow: visible;

      &:disabled {
        opacity: 0.6;
        cursor: wait;
        filter: grayscale(0.5);
      }

      /* 瓶身光晕 */
      .phial-glow {
        position: absolute;
        inset: 0;
        border-radius: 40px;
        background: radial-gradient(
          circle at 30% 50%,
          rgba(255, 200, 230, 0.2),
          transparent 70%
        );
        pointer-events: none;
        transition: opacity 0.3s;
      }

      /* 水母伞盖（悬浮在瓶口） */
      .jelly-umbrella {
        position: absolute;
        top: -14px;
        left: 10px;
        width: 20px;
        height: 20px;
        background: radial-gradient(
          circle at 30% 30%,
          rgba(90, 200, 250, 0.6),
          transparent 70%
        );
        border-radius: 50% 50% 30% 30%;
        filter: blur(3px);
        opacity: 0.7;
        transition: all 0.4s;
        animation: floatUmbrella 3s infinite alternate;
      }

      /* 主图标区域 */
      .phial-icon {
        font-size: 1.3rem;
        color: var(--jelly-blue);
        filter: drop-shadow(0 0 8px #ffb3c6);
        transition: transform 0.3s;
        display: inline-flex;
        align-items: center;
        justify-content: center;
        width: 1.8rem;
        height: 1.8rem;

        &.loading {
          transform: scale(1.1);
        }

        .loader {
          display: inline-block;
          width: 1.2rem;
          height: 1.2rem;
          border: 2px solid rgba(255, 255, 255, 0.3);
          border-top-color: var(--jelly-blue);
          border-radius: 50%;
          animation: spin 1s linear infinite;
        }

        &.playing {
          color: var(--venom-pink);
          filter: drop-shadow(0 0 12px hotpink);
        }
      }

      .status {
        font-size: 0.85rem;
        color: #d0d5f0;
        font-style: italic;
        letter-spacing: 0.05rem;
        position: relative;
        z-index: 2;
      }

      /* 漂浮的水母触须（多条） */
      .jelly-tentacle {
        position: absolute;
        bottom: -6px;
        width: 8px;
        height: 15px;
        background: linear-gradient(
          to top,
          rgba(90, 200, 250, 0.6),
          transparent
        );
        border-radius: 2px;
        filter: blur(2px);
        animation: tentacleSway 2s infinite alternate;
        pointer-events: none;

        &:nth-child(4) {
          left: 20%;
          animation-delay: 0s;
        }
        &:nth-child(5) {
          left: 40%;
          animation-delay: 0.3s;
          height: 20px;
          background: rgba(255, 107, 157, 0.4);
        }
        &:nth-child(6) {
          left: 60%;
          animation-delay: 0.6s;
          height: 12px;
        }
      }

      /* 底部触须光晕 */
      .tentacle-glow {
        position: absolute;
        bottom: -10px;
        left: 10%;
        width: 80%;
        height: 15px;
        background: radial-gradient(
          ellipse at center,
          rgba(90, 200, 250, 0.4),
          transparent 70%
        );
        filter: blur(5px);
        border-radius: 50%;
        transition: height 0.3s;
      }

      &:hover {
        transform: translateY(-3px);
        border-color: var(--jelly-blue);
        box-shadow: 0 0 30px rgba(90, 200, 250, 0.5);

        .jelly-umbrella {
          top: -18px;
          opacity: 1;
          filter: blur(2px);
        }
        .jelly-tentacle {
          height: 20px;
          filter: blur(1px);
        }
        .tentacle-glow {
          height: 20px;
          background: radial-gradient(
            ellipse at center,
            rgba(90, 200, 250, 0.6),
            transparent 70%
          );
        }
      }

      // 移动端下适当缩小
      @media (max-width: 768px) {
        margin-left: 0;
        margin-top: 8px;
        padding: 4px 14px;
        .phial-icon {
          font-size: 1.1rem;
        }
        .status {
          font-size: 0.75rem;
        }
      }
    }

    /* 动画定义 */
    @keyframes spin {
      to {
        transform: rotate(360deg);
      }
    }

    @keyframes floatUmbrella {
      0% {
        transform: translateY(0) rotate(-5deg);
        opacity: 0.5;
      }
      100% {
        transform: translateY(-4px) rotate(5deg);
        opacity: 0.9;
      }
    }

    @keyframes tentacleSway {
      0% {
        transform: translateY(0) scaleY(1);
        opacity: 0.5;
      }
      100% {
        transform: translateY(-4px) scaleY(1.2);
        opacity: 1;
      }
    } 
  }

  // 移动端汉堡菜单 (水母伞造型)
  .hamburger {
    display: none;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 42px;
    height: 42px;
    background: rgba(45, 20, 60, 0.6);
    border: 1px solid var(--jelly-blue);
    border-radius: 50%;
    backdrop-filter: blur(8px);
    position: relative;
    z-index: 20;
    cursor: pointer;
    box-shadow: 0 0 20px #ff6b9d80;
    span {
      display: block;
      width: 22px;
      height: 2px;
      background: var(--jelly-blue);
      margin: 3px 0;
      border-radius: 4px;
      transition: 0.2s;
      box-shadow: 0 0 8px var(--venom-pink);
    }
    .jelly-tassel {
      position: absolute;
      top: -5px;
      right: -2px;
      width: 14px;
      height: 14px;
      background: var(--venom-pink);
      border-radius: 50% 30% 50% 30%;
      filter: blur(4px);
      opacity: 0.8;
    }
    span.open:nth-child(1) {
      transform: translateY(8px) rotate(45deg);
      background: var(--venom-pink);
    }
    span.open:nth-child(2) {
      opacity: 0;
    }
    span.open:nth-child(3) {
      transform: translateY(-8px) rotate(-45deg);
      background: var(--venom-pink);
    }
  }

  // 移动端响应式
  @media (max-width: 920px) {
    padding: 0 24px;
    .brand .title-sub {
      display: none;
    }
  }
  @media (max-width: 768px) {
    .hamburger {
      display: flex;
    }
    .nav-links {
      position: absolute;
      top: 72px;
      left: 0;
      right: 0;
      flex-direction: column;
      background: rgba(8, 3, 20, 0.98);
      backdrop-filter: blur(20px);
      border-top: 1px solid var(--venom-pink);
      padding: 0 10px;
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.4s ease;
      gap: 8px;
      &.mobile-open {
        max-height: 780px;
      }
      .nav-item {
        width: 100%;
        text-align: center;
        padding: 14px 0;
      }
      .online-phial {
        margin: 16px 0 0 0;
        justify-content: center;
        width: fit-content;
      }
    }
    .brand .title-main {
      font-size: 1.5rem;
    }
  }
}

// 动画
@keyframes floatJelly {
  0% {
    transform: translate(0, 0) rotate(0deg) scale(1);
  }
  100% {
    transform: translate(20px, -15px) rotate(8deg) scale(1.05);
  }
}
@keyframes dropPulse {
  0% {
    opacity: 0.6;
    transform: scale(1);
  }
  50% {
    opacity: 1;
    transform: scale(1.5);
  }
  100% {
    opacity: 0.6;
    transform: scale(1);
  }
}
@keyframes floatNote {
  0% {
    transform: translateY(0) rotate(-5deg);
  }
  50% {
    transform: translateY(-6px) rotate(5deg);
  }
  100% {
    transform: translateY(0) rotate(-5deg);
  }
}
</style>
