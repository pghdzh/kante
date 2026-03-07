<template>
  <main class="cantarella-hall">
    <!-- ===== 深海幻境背景层 ===== -->
    <div class="abyss-backdrop">
      <!-- 幽海漩涡 -->
      <div class="swirl"></div>
      <!-- 浮游水母群 (动态触须) -->
      <div class="jellyfish-swarm">
        <span class="jelly"></span><span class="jelly"></span
        ><span class="jelly"></span> <span class="jelly"></span
        ><span class="jelly"></span>
      </div>
      <!-- 上升气泡群 -->
      <div class="bubble-field">
        <span class="bubble"></span><span class="bubble"></span
        ><span class="bubble"></span> <span class="bubble"></span
        ><span class="bubble"></span>
      </div>
      <!-- 极淡的“毒雾”弥散 -->
      <div class="venom-mist"></div>
    </div>

    <!-- ===== 画布：水母洋伞粒子 ===== -->
    <canvas ref="canvasEl" class="jelly-canvas" aria-hidden="true"></canvas>

    <!-- ===== 背景轮播 (两组用于响应式裁切) ===== -->
    <div class="carousel carousel-deep" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <div class="carousel carousel-shallows" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive2"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>

    <!-- ===== 中央圣所：家主絮语 ===== -->
    <section class="sanctum">
      <!-- 珊瑚骨洋伞 装饰 (动态旋转) -->
      <div class="umbrella-decoration">
        <span class="umbrella-icon"
          ><img src="@/assets/violet.png" alt=""
        /></span>
        <span class="tentacles"></span>
      </div>

      <h1 class="title">
        <span class="title-line">翡萨烈家主</span>
        <span class="title-glow">坎特蕾拉</span>
      </h1>

      <!-- 幻梦絮语·打字机 (取自角色台词/背景) -->
      <div class="dream-words" aria-live="polite">
        <span class="typed">{{ typed }}</span>
        <span class="cursor" aria-hidden="true"></span>
      </div>

      <!-- 低语波纹装饰 -->
      <div class="whisper-ripple"></div>
    </section>

    <!-- ===== 梦涯页脚 ===== -->
    <footer class="dreamshore-footer">
      <div class="footer-inner">
        <div class="footer-slogan">以梦为丝，以海为镜，编织真实的幻境</div>
        <div class="footer-meta">
          © {{ year }} 坎特蕾拉电子设定集 · 深海秘藏
        </div>

        <!-- 优化后的群号容器：秘药瓶样式 -->
        <a
          href="http://qm.qq.com/cgi-bin/qm/qr?_wv=1027&k=RTN2gv0Rt17exZfET-WJ-8fRBJN-Atfg&authKey=PrOTvT5PfxE54V4qsgRKukiTeMl2nX3ZonMOoJDokmQ1d3U8CoWNB3rx972nAZry&noverify=0&group_code=629760313"
          target="_blank"
        >
          <div class="footer-group">
            <span class="group-icon">⚱️</span>
            <span class="group-text">点击跳转家主同好QQ群:</span>
            <span class="group-number">629760313</span>
            <!-- 瓶身光晕装饰 -->
            <span class="group-glow"></span>
            <!-- 瓶底触须装饰 -->
            <span class="group-tentacles"></span></div
        ></a>
      </div>

      <!-- 底部水母触须（增强） -->
      <div class="footer-tentacles"></div>
    </footer>
  </main>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";

const year = new Date().getFullYear();

// ========== Canvas 水母粒子 (替代原玫瑰粒子) ==========
const canvasEl = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D;
let animationId = 0;
let lastTime = 0;
let elapsed = 0;

interface JellyParticle {
  baseX: number;
  y: number;
  size: number; // 伞盖大小
  speed: number;
  swayAmp: number; // 水平摆动幅度
  swayFreq: number;
  phase: number;
  angle: number; // 自身旋转
  angularSpeed: number;
  tentaclePhase: number; // 触须动画相位
  type: "jelly" | "droplet"; // 水母 或 毒液滴
}

const particles: JellyParticle[] = [];
const PARTICLE_COUNT_DESKTOP = 24;
const PARTICLE_COUNT_MOBILE = 10;

// 绘制水母 (简易风格)
function drawJelly(
  ctx: CanvasRenderingContext2D,
  x: number,
  y: number,
  size: number,
  angle: number,
  phase: number,
  alpha: number
) {
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(angle);
  ctx.globalAlpha = alpha * 0.7;

  // 伞盖 (半透明椭圆, 紫蓝渐变)
  const gradient = ctx.createRadialGradient(
    0,
    -size * 0.1,
    0,
    0,
    0,
    size * 0.6
  );
  gradient.addColorStop(0, "rgba(200, 180, 255, 0.9)");
  gradient.addColorStop(0.5, "rgba(90, 200, 250, 0.5)");
  gradient.addColorStop(1, "rgba(40, 20, 70, 0.2)");
  ctx.fillStyle = gradient;
  ctx.beginPath();
  ctx.ellipse(0, -size * 0.1, size * 0.4, size * 0.25, 0, 0, Math.PI * 2);
  ctx.fill();

  // 触须 (多条曲线)
  ctx.strokeStyle = "rgba(200, 220, 255, 0.5)";
  ctx.lineWidth = 1.5;
  for (let i = 0; i < 5; i++) {
    const offset = (i - 2) * size * 0.15;
    const t = phase + i * 0.8;
    ctx.beginPath();
    ctx.moveTo(offset, size * 0.1);
    ctx.quadraticCurveTo(
      offset + Math.sin(t) * size * 0.2,
      size * 0.4,
      offset + Math.sin(t * 1.3) * size * 0.3,
      size * 0.8
    );
    ctx.strokeStyle = `rgba(160, 200, 255, ${0.3 + Math.sin(t) * 0.1})`;
    ctx.stroke();
  }
  ctx.restore();
}

// 绘制毒液滴 (幻觉毒)
function drawDroplet(
  ctx: CanvasRenderingContext2D,
  x: number,
  y: number,
  size: number,
  alpha: number
) {
  ctx.save();
  ctx.translate(x, y);
  ctx.globalAlpha = alpha * 0.5;
  const grad = ctx.createRadialGradient(0, 0, 0, 0, 0, size);
  grad.addColorStop(0, "#ff6b9d");
  grad.addColorStop(0.7, "#9d4edd");
  grad.addColorStop(1, "rgba(0,0,0,0)");
  ctx.fillStyle = grad;
  ctx.beginPath();
  ctx.arc(0, 0, size * 0.5, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();
}

function initParticles(count: number) {
  particles.length = 0;
  const canvas = canvasEl.value!;
  const w = canvas.width / (window.devicePixelRatio || 1);
  const h = canvas.height / (window.devicePixelRatio || 1);

  for (let i = 0; i < count; i++) {
    const type = Math.random() > 0.7 ? "droplet" : "jelly"; // 约30%为毒液滴
    particles.push({
      baseX: Math.random() * w,
      y: Math.random() * -h,
      size: type === "jelly" ? 20 + Math.random() * 50 : 8 + Math.random() * 18,
      speed: 8 + Math.random() * 30,
      swayAmp:
        type === "jelly" ? 15 + Math.random() * 30 : 5 + Math.random() * 10,
      swayFreq: 0.1 + Math.random() * 0.6,
      phase: Math.random() * Math.PI * 2,
      angle: Math.random() * Math.PI * 2,
      angularSpeed: (Math.random() - 0.5) * 1.2,
      tentaclePhase: Math.random() * 10,
      type,
    });
  }
  elapsed = 0;
}

let resizeTimer: number;
function resizeCanvas() {
  window.clearTimeout(resizeTimer);
  resizeTimer = window.setTimeout(() => {
    cancelAnimationFrame(animationId);
    const canvas = canvasEl.value!;
    const parent = canvas.parentElement!;
    const dpr = window.devicePixelRatio || 1;
    const w = parent.clientWidth;
    const h = Math.max(parent.clientHeight, 500);

    canvas.style.width = w + "px";
    canvas.style.height = h + "px";
    canvas.width = w * dpr;
    canvas.height = h * dpr;

    ctx.setTransform(1, 0, 0, 1, 0, 0);
    ctx.scale(dpr, dpr);

    const isMobile = w < 768;
    initParticles(isMobile ? PARTICLE_COUNT_MOBILE : PARTICLE_COUNT_DESKTOP);
    lastTime = 0;
    animationId = requestAnimationFrame(tickCanvas);
  }, 160);
}

function tickCanvas(now: number) {
  if (!lastTime) lastTime = now;
  const dt = (now - lastTime) / 1000;
  lastTime = now;
  elapsed += dt;

  const canvas = canvasEl.value!;
  const w = canvas.clientWidth;
  const h = canvas.clientHeight;

  ctx.clearRect(0, 0, w, h);

  // 极淡的深海底色 (保持通透)
  ctx.fillStyle = "rgba(4, 6, 18, 0.1)";
  ctx.fillRect(0, 0, w, h);

  particles.forEach((p) => {
    p.y += p.speed * dt;
    const sway = p.swayAmp * Math.sin(p.phase + elapsed * p.swayFreq);
    const x = p.baseX + sway;
    p.angle += p.angularSpeed * dt;
    p.tentaclePhase += dt;

    if (p.y > h + p.size * 2) {
      p.y = -p.size * 0.8;
      p.baseX = Math.random() * w;
      p.phase = Math.random() * Math.PI * 2;
    }

    const alpha = Math.max(0, Math.min(1, 1 - (p.y / h) * 0.7 + 0.2));

    if (p.type === "jelly") {
      drawJelly(ctx, x, p.y, p.size, p.angle, p.tentaclePhase, alpha);
    } else {
      drawDroplet(ctx, x, p.y, p.size, alpha);
    }
  });

  animationId = requestAnimationFrame(tickCanvas);
}

// ========== 打字机文案 (取自角色故事/台词) ==========
const lines = [
  "神秘、高贵、美艳、剧毒……",
  "我居于山巅的冠冕，而山崖间流淌着的，都是由我编织的幻梦。",
  "是「毒」，还是「药」，取决于你的用法哦……",
  "当你进入我的海域，周身浮现的波纹，已经将一切都告诉了我。",
  "虚假的幻术，也可以促成美妙的真实",
  "花园里那些可爱的小东西们，由我亲手采摘、筛选、调配……",
  "酿成一壶壶甜蜜的汁液，温柔的香气唤来美梦。",
  "我喜欢你凝视我的样子。原本深邃的幽海，也变得清澈见底。",
  "微光穿透深海，映照出隐现于汹涌浪潮之间的温柔呢喃……",
];

const typed = ref("");
let lineIndex = 0;
let charIndex = 0;
let deleting = false;
let timer: number | null = null;

const TYPING = 100;
const DELETING = 40;
const PAUSE = 2000;

function tick() {
  const cur = lines[lineIndex];
  if (!deleting) {
    typed.value = cur.slice(0, charIndex + 1);
    charIndex++;
    if (charIndex >= cur.length) {
      timer = window.setTimeout(() => {
        deleting = true;
        tick();
      }, PAUSE);
      return;
    }
    timer = window.setTimeout(tick, TYPING);
  } else {
    typed.value = cur.slice(0, charIndex - 1);
    charIndex--;
    if (charIndex <= 0) {
      deleting = false;
      lineIndex = (lineIndex + 1) % lines.length;
      timer = window.setTimeout(tick, 360);
      return;
    }
    timer = window.setTimeout(tick, DELETING);
  }
}

// ========== 背景图片导入与轮播 ==========
const modules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs: string[] = Object.values(modules).map((mod: any) => mod.default);

const modules2 = import.meta.glob("@/assets/images2/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs2: string[] = Object.values(modules2).map(
  (mod: any) => mod.default
);

function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
const randomFive = ref<string[]>(shuffle(allSrcs).slice(0, 5));
const randomFive2 = ref<string[]>(shuffle(allSrcs2).slice(0, 5));
const currentIndex = ref(0);
let imgTimer: number | undefined;

onMounted(() => {
  timer = window.setTimeout(tick, 420);
  imgTimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5800);

  const canvas = canvasEl.value!;
  ctx = canvas.getContext("2d")!;
  resizeCanvas();
  window.addEventListener("resize", resizeCanvas);
});

onBeforeUnmount(() => {
  if (imgTimer) clearInterval(imgTimer);
  if (timer) window.clearTimeout(timer);
  cancelAnimationFrame(animationId);
  window.removeEventListener("resize", resizeCanvas);
});
</script>

<style scoped lang="scss">
// 字体：优雅衬线 + 手写点缀
@import url("https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@400;500;600;700&family=Playfair+Display:wght@400;600;700&family=Qwitcher+Grypen:wght@400;700&display=swap");

.cantarella-hall {
  --deep-violet: #0f0722;
  --venom-pink: #ff6b9d;
  --jelly-blue: #5ac8fa;
  --abyss-glow: #8a6de9;
  --pearl-mist: #f0eaf8;
  --mist-transparent: rgba(90, 200, 250, 0.03);

  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background: radial-gradient(ellipse at 30% 40%, #1d1238, #030014 90%);
  position: relative;
  overflow: hidden;
  color: var(--pearl-mist);
  font-family: "Cormorant Garamond", "Playfair Display", serif;

  // 深海背景层 (所有动态装饰)
  .abyss-backdrop {
    position: absolute;
    inset: 0;
    pointer-events: none;
    z-index: 1;

    .swirl {
      position: absolute;
      top: -20%;
      left: -10%;
      width: 70%;
      height: 140%;
      background: radial-gradient(
        circle at 30% 50%,
        rgba(90, 200, 250, 0.06) 0%,
        transparent 60%
      );
      filter: blur(50px);
      animation: swirlMove 25s infinite alternate;
    }

    .jellyfish-swarm {
      .jelly {
        position: absolute;
        background: radial-gradient(
          circle at 30% 40%,
          rgba(200, 150, 255, 0.1),
          transparent 70%
        );
        border-radius: 60% 40% 50% 30% / 40% 50% 40% 50%;
        filter: blur(16px);
        animation: floatJelly 22s infinite alternate;
        &:nth-child(1) {
          top: 15%;
          left: 5%;
          width: 150px;
          height: 180px;
          background: rgba(255, 107, 157, 0.08);
          animation-duration: 28s;
        }
        &:nth-child(2) {
          bottom: 5%;
          right: 2%;
          width: 220px;
          height: 260px;
          background: rgba(90, 200, 250, 0.06);
          animation-duration: 32s;
        }
        &:nth-child(3) {
          top: 40%;
          right: 15%;
          width: 100px;
          height: 120px;
          background: rgba(200, 130, 250, 0.1);
          animation-duration: 18s;
        }
        &:nth-child(4) {
          bottom: 20%;
          left: 20%;
          width: 80px;
          height: 100px;
          background: rgba(255, 180, 200, 0.07);
          animation-duration: 20s;
        }
        &:nth-child(5) {
          top: 70%;
          left: 40%;
          width: 60px;
          height: 80px;
          background: rgba(140, 220, 255, 0.1);
          animation-duration: 15s;
        }
      }
    }

    .bubble-field {
      .bubble {
        position: absolute;
        bottom: -20px;
        width: 10px;
        height: 10px;
        background: rgba(255, 255, 255, 0.15);
        border-radius: 50%;
        filter: blur(3px);
        animation: bubbleUp 10s infinite;
        &:nth-child(1) {
          left: 15%;
          width: 18px;
          height: 18px;
          animation-duration: 9s;
        }
        &:nth-child(2) {
          left: 35%;
          width: 12px;
          height: 12px;
          animation-duration: 12s;
          animation-delay: 2s;
        }
        &:nth-child(3) {
          left: 55%;
          width: 22px;
          height: 22px;
          animation-duration: 7s;
          animation-delay: 1s;
        }
        &:nth-child(4) {
          left: 75%;
          width: 14px;
          height: 14px;
          animation-duration: 11s;
          animation-delay: 3s;
        }
        &:nth-child(5) {
          left: 90%;
          width: 16px;
          height: 16px;
          animation-duration: 8s;
          animation-delay: 0.5s;
        }
      }
    }

    .venom-mist {
      position: absolute;
      inset: 0;
      background: radial-gradient(
        circle at 70% 30%,
        rgba(255, 107, 157, 0.02),
        transparent 50%
      );
      mix-blend-mode: screen;
      pointer-events: none;
    }
  }

  // Canvas 粒子层
  .jelly-canvas {
    position: absolute;
    inset: 0;
    z-index: 2;
    pointer-events: none;
  }

  // 轮播背景
  .carousel {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 1.8s ease;
      filter: saturate(1) brightness(0.6) contrast(1.1);
      &.active {
        opacity: 1;
      }
    }
  }
  .carousel-shallows {
    display: none;
  }

  // 中央圣所
  .sanctum {
    position: relative;
    z-index: 5;
    flex: 1 0 auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 40px 24px;
    gap: 24px;

    .umbrella-decoration {
      position: relative;
      margin-bottom: 10px;
      .umbrella-icon {
        font-size: 5rem;
        line-height: 1;
        color: var(--jelly-blue);
        text-shadow: 0 0 30px var(--venom-pink), 0 0 60px #b282ff;
        filter: drop-shadow(0 10px 20px #00000080);
        transform: rotate(-8deg);
        animation: umbrellaFloat 6s ease-in-out infinite;
        display: inline-block;
        img {
          width: 120px;
        }
      }
      .tentacles {
        position: absolute;
        bottom: -20px;
        left: 50%;
        transform: translateX(-50%);
        width: 80px;
        height: 60px;
        background: radial-gradient(
          ellipse at 50% 0%,
          rgba(90, 200, 250, 0.3),
          transparent 70%
        );
        filter: blur(12px);
        animation: tentacleWave 4s infinite alternate;
      }
    }

    .title {
      font-family: "Playfair Display", serif;
      font-weight: 700;
      margin: 0;
      .title-line {
        font-size: 2rem;
        letter-spacing: 0.3em;
        color: rgba(255, 255, 255, 0.7);
        display: block;
        font-style: italic;
        text-shadow: 0 0 20px var(--jelly-blue);
      }
      .title-glow {
        font-size: 5rem;
        background: linear-gradient(
          145deg,
          #fff,
          var(--jelly-blue),
          var(--venom-pink)
        );
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        background-clip: text;
        text-shadow: 0 0 40px rgba(255, 107, 157, 0.5);
        line-height: 1.2;
      }
    }

    .dream-words {
      font-size: 2.2rem;
      min-height: 3rem;
      color: rgba(255, 255, 255, 0.9);
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 12px;
      font-family: "Qwitcher Grypen", cursive;
      font-weight: 500;
      max-width: 800px;
      text-shadow: 0 0 30px var(--jelly-blue);
      .typed {
        display: inline-block;
      }
      .cursor {
        width: 4px;
        height: 2.4rem;
        background: linear-gradient(
          180deg,
          var(--jelly-blue),
          var(--venom-pink)
        );
        border-radius: 2px;
        animation: blink 1s steps(1) infinite;
        filter: drop-shadow(0 0 16px cyan);
      }
    }

    .whisper-ripple {
      width: 300px;
      height: 60px;
      background: radial-gradient(
        ellipse at 50% 0%,
        rgba(90, 200, 250, 0.1),
        transparent 70%
      );
      filter: blur(20px);
      margin-top: 20px;
    }
  }

  // 页脚
  .dreamshore-footer {
    position: relative;
    z-index: 5;
    background: linear-gradient(
      180deg,
      rgba(10, 5, 20, 0.7),
      rgba(2, 0, 8, 0.9)
    );
    border-top: 1px solid rgba(255, 107, 157, 0.2);
    padding: 24px 0 20px;
    backdrop-filter: blur(4px);
    overflow: hidden; /* 防止触须溢出造成滚动条 */

    .footer-inner {
      width: min(1100px, 94%);
      margin: 0 auto;
      text-align: center;
      position: relative;
      z-index: 2;
    }

    .footer-slogan {
      font-size: 1.2rem;
      background: linear-gradient(90deg, var(--jelly-blue), var(--venom-pink));
      -webkit-background-clip: text;
      background-clip: text;
      -webkit-text-fill-color: transparent;
      letter-spacing: 0.1rem;
      margin-bottom: 8px;
    }

    .footer-meta {
      color: rgba(255, 255, 255, 0.5);
      font-size: 0.9rem;
      margin-bottom: 16px;
    }

    /* 群号容器 - 秘药瓶风格 */
    .footer-group {
      position: relative;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
      background: rgba(2, 6, 20, 0.5);
      backdrop-filter: blur(8px);
      border: 1px solid var(--glass-edge);
      border-radius: 60px;
      padding: 8px 24px 8px 20px;
      margin: 0 auto;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5),
        0 0 0 1px rgba(122, 226, 255, 0.1) inset;
      transition: all 0.3s ease;

      /* 悬停时微微发光 */
      &:hover {
        border-color: var(--jelly-blue);
        box-shadow: 0 10px 30px rgba(122, 226, 255, 0.2),
          0 0 20px rgba(122, 226, 255, 0.2);
        transform: translateY(-2px);
      }

      .group-icon {
        font-size: 1.4rem;
        filter: drop-shadow(0 0 8px var(--jelly-blue));
        transition: filter 0.3s;
      }

      .group-text {
        color: rgba(255, 255, 255, 0.8);
        font-size: 1rem;
        letter-spacing: 0.5px;
      }

      .group-number {
        font-size: 1.2rem;
        font-weight: 600;
        background: linear-gradient(
          135deg,
          var(--jelly-blue),
          var(--venom-pink)
        );
        font-family: "Courier New", Courier, monospace;
        -webkit-background-clip: text;
        background-clip: text;
        -webkit-text-fill-color: transparent;
        text-shadow: 0 0 10px rgba(122, 226, 255, 0.3);
        margin-left: 4px;
      }

      /* 瓶身内部光晕 */
      .group-glow {
        position: absolute;
        inset: 0;
        border-radius: 60px;
        background: radial-gradient(
          circle at 30% 30%,
          rgba(122, 226, 255, 0.1),
          transparent 70%
        );
        pointer-events: none;
        opacity: 0;
        transition: opacity 0.3s;
      }

      &:hover .group-glow {
        opacity: 1;
      }

      /* 瓶底触须（与页脚触须呼应） */
      .group-tentacles {
        position: absolute;
        bottom: -10px;
        left: 20%;
        width: 60%;
        height: 12px;
        background: radial-gradient(
          ellipse at center,
          rgba(122, 226, 255, 0.3),
          transparent 70%
        );
        filter: blur(4px);
        border-radius: 50%;
        transition: all 0.3s;
      }

      &:hover .group-tentacles {
        height: 16px;
        background: radial-gradient(
          ellipse at center,
          rgba(122, 226, 255, 0.5),
          transparent 70%
        );
      }
    }

    /* 底部水母触须（增强） */
    .footer-tentacles {
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 40px;
      background: repeating-linear-gradient(
        90deg,
        transparent,
        transparent 30px,
        rgba(122, 226, 255, 0.15) 30px,
        rgba(255, 179, 198, 0.1) 60px
      );
      filter: blur(8px);
      pointer-events: none;
      animation: tentacle-wave 8s infinite alternate;
    }

    /* 触须浮动动画 */
    @keyframes tentacle-wave {
      0% {
        opacity: 0.3;
        transform: translateY(0);
      }
      50% {
        opacity: 0.7;
        transform: translateY(-4px);
      }
      100% {
        opacity: 0.3;
        transform: translateY(0);
      }
    }
  }

  /* 响应式：移动端调整群号容器 */
  @media (max-width: 720px) {
    .dreamshore-footer .footer-group {
      flex-wrap: wrap;
      padding: 8px 16px;
      gap: 6px;
      max-width: 90%;

      .group-text,
      .group-number {
        font-size: 0.95rem;
      }
      .group-icon {
        font-size: 1.2rem;
      }
    }
  }
}

// 响应式
@media (max-width: 768px) {
  .carousel-deep {
    display: none;
  }
  .carousel-shallows {
    display: block;
  }
  .sanctum {
    .title {
      .title-line {
        font-size: 1.5rem;
      }
      .title-glow {
        font-size: 3.2rem;
      }
    }
    .dream-words {
      font-size: 1.6rem;
    }
  }
}

// 动画
@keyframes swirlMove {
  0% {
    transform: rotate(0deg) scale(1);
    opacity: 0.3;
  }
  100% {
    transform: rotate(12deg) scale(1.3);
    opacity: 0.6;
  }
}
@keyframes floatJelly {
  0% {
    transform: translate(0, 0) rotate(0deg) scale(1);
    opacity: 0.3;
  }
  100% {
    transform: translate(40px, -30px) rotate(12deg) scale(1.1);
    opacity: 0.6;
  }
}
@keyframes bubbleUp {
  0% {
    transform: translateY(0) scale(1);
    opacity: 0.3;
  }
  100% {
    transform: translateY(-150px) scale(0.6);
    opacity: 0;
  }
}
@keyframes bubbleUpSmall {
  0% {
    transform: translateY(0) scale(1);
    opacity: 0.4;
  }
  100% {
    transform: translateY(-40px) scale(0.5);
    opacity: 0;
  }
}
@keyframes staffScroll {
  0% {
    transform: translateY(-5%);
  }
  100% {
    transform: translateY(5%);
  }
}
@keyframes umbrellaFloat {
  0% {
    transform: rotate(-8deg) scale(1);
  }
  50% {
    transform: rotate(-2deg) scale(1.05);
  }
  100% {
    transform: rotate(-8deg) scale(1);
  }
}
@keyframes tentacleWave {
  0% {
    opacity: 0.2;
    filter: blur(12px);
  }
  100% {
    opacity: 0.6;
    filter: blur(20px);
  }
}
@keyframes numberPop {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
    text-shadow: 0 0 40px white;
  }
  100% {
    transform: scale(1);
  }
}
@keyframes blink {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
</style>
