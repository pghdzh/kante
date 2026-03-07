<template>
  <div class="cantarella-message-board" aria-live="polite">
    <!-- 背景轮播（两组用于桌面/移动不同裁切） -->
    <div class="carousel carousel-desktop" aria-hidden="true">
      <img
        v-for="(src, idx) in desktopImages"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
        alt=""
      />
    </div>
    <div class="carousel carousel-mobile" aria-hidden="true">
      <img
        v-for="(src, idx) in mobileImages"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
        alt=""
      />
    </div>

    <!-- 头部：深海明珠 -->
    <header class="board-header" role="banner">
      <div class="pearl">
        <h1>翡萨烈·幻梦回响</h1>
        <div class="pearl-count">
          <span>{{ totalCount }}</span> 条回响
        </div>
      </div>
      <div class="tendrils">
        <span></span><span></span><span></span><span></span><span></span>
      </div>
      <p class="subtitle">微光穿透深海，映照出隐现于汹涌浪潮之间的温柔呢喃……</p>
    </header>

    <!-- 留言列表（无限滚动） -->
    <section class="message-list">
      <div class="message-list-inner">
        <!-- 首次加载骨架屏 -->
        <div v-if="loading && messages.length === 0" class="skeleton-wrap">
          <div class="skeleton" v-for="i in 3" :key="i">
            <div class="sk-glow"></div>
          </div>
        </div>

        <!-- 留言卡片 -->
        <transition-group name="msg" tag="div" class="messages-container">
          <div
            v-for="(msg, idx) in messages"
            :key="msg.id"
            class="message-card"
            :data-index="idx"
            tabindex="0"
            role="article"
            :aria-label="`留言来自 ${msg.name || '匿名'}，内容：${msg.content}`"
          >
            <div class="card-inner">
              <div class="message-header">
                <div class="avatar" :title="msg.name || '匿名'">
                  {{ getInitial(msg.name) }}
                  <span class="avatar-glow"></span>
                </div>
                <div class="header-text">
                  <div class="message-name">{{ msg.name || "匿名" }}</div>
                  <div class="message-time">
                    {{ formatTime(msg.created_at) }}
                  </div>
                </div>
              </div>
              <p class="message-content">{{ msg.content }}</p>
              <div class="card-footer">
                <span class="ripple"></span>
              </div>
            </div>
          </div>
        </transition-group>

        <!-- 无限滚动哨兵 & 加载更多指示器 -->
        <div ref="sentinel" class="sentinel"></div>
        <div v-if="loadingMore" class="loading-more">
          <div class="jelly-loader"></div>
          <span>浮出水面……</span>
        </div>
        <div v-if="!hasMore && messages.length > 0" class="no-more">
          —— 已至深海之底 ——
        </div>
      </div>
    </section>

    <!-- 底部留言区：珊瑚台 -->
    <section class="message-form" aria-label="留下你的幻梦回响">
      <div class="coral-base">
        <div class="form-container">
          <label class="sr-only" for="mb-name">昵称</label>
          <input
            id="mb-name"
            v-model="name"
            type="text"
            placeholder="你的名字（翡萨烈之名）"
            @keydown.enter.prevent
          />

          <label class="sr-only" for="mb-content">留言内容</label>
          <textarea
            id="mb-content"
            v-model="content"
            placeholder="写下你的幻梦回响..."
            @keydown.ctrl.enter.prevent="submitMessage"
            @input="autoGrow"
            ref="textareaRef"
          />

          <div class="form-actions">
            <div class="hint"><kbd>ctrl</kbd> + <kbd>enter</kbd>快速发送</div>
            <button
              @click="submitMessage"
              :disabled="isSending || !content.trim()"
            >
              {{ isSending ? "熬制秘药中…" : "沉入幻梦" }}
            </button>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, onBeforeUnmount } from "vue";
import { getMessageList, createMessage } from "@/api/modules/message";

// 留言数据
const messages = ref<any[]>([]);
const totalCount = ref(0);
const name = ref(localStorage.getItem("message_name") || "");
const content = ref("");
const loading = ref(true);
const isSending = ref(false);
const textareaRef = ref<HTMLTextAreaElement | null>(null);

// 分页相关
const currentPage = ref(1);
const pageSize = 20;
const hasMore = ref(true);
const loadingMore = ref(false);

// 背景图片轮播
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

const desktopImages = ref<string[]>(shuffle(allSrcs).slice(0, 5));
const mobileImages = ref<string[]>(shuffle(allSrcs2).slice(0, 5));
const currentIndex = ref(0);
let carouselTimer: number | undefined;

// 获取留言（支持分页）
const fetchMessages = async (page: number, append = false) => {
  if (append) {
    loadingMore.value = true;
  } else {
    loading.value = true;
  }

  try {
    const res = await getMessageList({ page, pageSize });
    const newMessages = res.data || [];
    const pagination = res.pagination;

    if (append) {
      messages.value = [...messages.value, ...newMessages];
    } else {
      messages.value = newMessages;
    }

    totalCount.value = pagination.total;
    hasMore.value = page < pagination.totalPages;
    currentPage.value = page;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
    loadingMore.value = false;
  }
};

// 提交留言
const submitMessage = async () => {
  if (!content.value.trim() || isSending.value) return;
  isSending.value = true;
  const payload = { name: name.value || "匿名", content: content.value };
  try {
    localStorage.setItem("message_name", name.value);
    content.value = "";
    await nextTick();
    await createMessage(payload);
    // 重置列表，从第一页重新加载
    messages.value = [];
    await fetchMessages(1, false);
  } catch (err) {
    console.error(err);
  } finally {
    isSending.value = false;
  }
};

// 时间格式化
const formatTime = (time: string) => {
  if (!time) return "";
  const d = new Date(time);
  const y = d.getFullYear();
  const m = String(d.getMonth() + 1).padStart(2, "0");
  const day = String(d.getDate()).padStart(2, "0");
  const hh = String(d.getHours()).padStart(2, "0");
  const mm = String(d.getMinutes()).padStart(2, "0");
  return `${y}-${m}-${day} ${hh}:${mm}`;
};

// 获取首字母
const getInitial = (n?: string) => {
  if (!n) return "匿";
  return n.trim().slice(0, 1).toUpperCase();
};

// 自动增高文本框
const autoGrow = () => {
  const ta = textareaRef.value;
  if (!ta) return;
  ta.style.height = "auto";
  ta.style.height = Math.min(ta.scrollHeight, 220) + "px";
};

// 无限滚动哨兵
const sentinel = ref<HTMLElement | null>(null);
let observer: IntersectionObserver | null = null;

const setupObserver = () => {
  observer = new IntersectionObserver(
    (entries) => {
      if (
        entries[0].isIntersecting &&
        hasMore.value &&
        !loadingMore.value &&
        !loading.value
      ) {
        fetchMessages(currentPage.value + 1, true);
      }
    },
    { threshold: 0.1 }
  );

  if (sentinel.value) {
    observer.observe(sentinel.value);
  }
};

onMounted(() => {
  fetchMessages(1, false).then(() => {
    nextTick(() => {
      setupObserver();
    });
  });

  carouselTimer = window.setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % desktopImages.value.length;
  }, 5200);

  nextTick(() => autoGrow());
});

onBeforeUnmount(() => {
  if (carouselTimer) clearInterval(carouselTimer);
  if (observer) observer.disconnect();
});
</script>

<style scoped lang="scss">
/* 坎特蕾拉色板 - 全新轻透版本 */
$deep-bg: #030614; // 极深海
$mid-bg: #14213d; // 深海中层
$accent-lavender: #b8a9ff; // 薰衣草紫
$accent-aqua: #7ae2ff; // 水母荧光
$accent-pink: #ffb3c6; // 毒药粉
$accent-deep-blue: #2a4a7a; // 深海蓝
$text-light: #f0f5fe;
$card-bg: rgba(255, 255, 255, 0.03);
$glass-edge: rgba($accent-aqua, 0.15);

.cantarella-message-board {
  position: relative;
  min-height: 100vh;
  padding: 100px 20px 200px;
  display: flex;
  flex-direction: column;
  background: linear-gradient(180deg, $deep-bg 0%, $mid-bg 100%);
  color: $text-light;
  overflow-y: auto;

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

  /* ===== 头部：深海明珠 ===== */
  .board-header {
    position: relative;
    z-index: 10;
    max-width: 960px;
    margin: 0 auto 40px;
    width: 100%;
    text-align: center;

    .pearl {
      display: inline-block;
      background: rgba(255, 255, 255, 0.02);
      backdrop-filter: blur(2px);
      -webkit-backdrop-filter: blur(2px);
      border: 1px solid $glass-edge;
      border-radius: 120px;
      padding: 16px 48px;
      box-shadow: 0 0 40px rgba($accent-aqua, 0.3),
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

      .pearl-count {
        margin-top: 8px;
        font-size: 0.95rem;
        color: rgba($text-light, 0.7);
        span {
          color: $accent-aqua;
          font-weight: 600;
          font-size: 1.2rem;
          margin-right: 4px;
        }
      }
    }

    .tendrils {
      display: flex;
      justify-content: center;
      gap: 24px;
      margin: 8px 0 12px;

      span {
        width: 2px;
        height: 30px;
        background: linear-gradient(to bottom, $accent-aqua, transparent);
        border-radius: 1px;
        animation: tendrilWave 4s infinite alternate;
        opacity: 0.7;

        &:nth-child(1) {
          height: 40px;
          animation-delay: 0s;
        }
        &:nth-child(2) {
          height: 25px;
          animation-delay: 0.8s;
        }
        &:nth-child(3) {
          height: 50px;
          background: $accent-lavender;
          animation-delay: 0.3s;
        }
        &:nth-child(4) {
          height: 35px;
          animation-delay: 1.2s;
        }
        &:nth-child(5) {
          height: 45px;
          animation-delay: 0.5s;
        }
      }
    }

    .subtitle {
      font-size: 1rem;
      color: rgba($text-light, 0.8);
      font-style: italic;
      font-weight: 300;
      text-shadow: 0 2px 10px rgba($accent-aqua, 0.3);
      max-width: 600px;
      margin: 0 auto;
    }
  }

  /* ===== 留言列表 ===== */
  .message-list {
    position: relative;
    z-index: 5;
    flex: 1;
    max-width: 960px;
    width: 100%;
    margin: 0 auto;

    .message-list-inner {
      display: flex;
      flex-direction: column;
      gap: 24px;
    }

    /* 骨架屏 */
    .skeleton-wrap {
      display: flex;
      flex-direction: column;
      gap: 20px;

      .skeleton {
        height: 120px;
        background: rgba(255, 255, 255, 0.02);
        border-radius: 60px 60px 30px 30px;
        border: 1px solid $glass-edge;
        position: relative;
        overflow: hidden;

        .sk-glow {
          position: absolute;
          inset: 0;
          background: linear-gradient(
            90deg,
            transparent,
            rgba($accent-aqua, 0.1),
            transparent
          );
          animation: shimmer 2s infinite;
        }
      }
    }

    .messages-container {
      display: flex;
      flex-direction: column;
      gap: 20px;
    }

    /* 留言卡片 - 浮游生物造型 */
    .message-card {
      background: transparent;
      perspective: 1000px;

      .card-inner {
        background: $card-bg;
        backdrop-filter: blur(2px);
        -webkit-backdrop-filter: blur(2px);
        border: 1px solid $glass-edge;
        border-radius: 40px 40px 30px 30px;
        padding: 24px 28px;
        box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3),
          0 0 0 1px rgba($accent-lavender, 0.1) inset;
        transition: all 0.3s ease;
        position: relative;
        overflow: hidden;

        &::before {
          content: "";
          position: absolute;
          top: -10px;
          left: 10%;
          width: 80%;
          height: 30px;
          background: radial-gradient(
            ellipse at center,
            rgba($accent-aqua, 0.2),
            transparent 70%
          );
          filter: blur(10px);
          pointer-events: none;
        }

        &:hover {
          transform: translateY(-4px) scale(1.01);
          border-color: rgba($accent-aqua, 0.4);
          box-shadow: 0 30px 60px rgba($accent-aqua, 0.2);
        }
      }

      .message-header {
        display: flex;
        gap: 16px;
        align-items: center;
        margin-bottom: 16px;
        position: relative;
        z-index: 2;

        .avatar {
          position: relative;
          width: 52px;
          height: 52px;
          border-radius: 40% 60% 40% 60%; /* 不规则水母形状 */
          background: linear-gradient(
            135deg,
            $accent-lavender,
            $accent-deep-blue
          );
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 1.5rem;
          font-weight: 300;
          color: white;
          box-shadow: 0 0 20px rgba($accent-aqua, 0.5);
          border: 1px solid rgba(white, 0.2);

          .avatar-glow {
            position: absolute;
            inset: -5px;
            border-radius: inherit;
            background: radial-gradient(
              circle at 30% 30%,
              rgba($accent-aqua, 0.4),
              transparent 70%
            );
            filter: blur(8px);
            z-index: -1;
          }
        }

        .header-text {
          .message-name {
            font-size: 1.2rem;
            font-weight: 400;
            color: $accent-aqua;
            margin-bottom: 4px;
            letter-spacing: 0.5px;
          }
          .message-time {
            font-size: 0.8rem;
            color: rgba($text-light, 0.5);
            font-weight: 300;
          }
        }
      }

      .message-content {
        font-size: 1.05rem;
        line-height: 1.6;
        margin: 0 0 12px 0;
        color: rgba($text-light, 0.95);
        white-space: pre-wrap;
        word-break: break-word;
        font-weight: 300;
        letter-spacing: 0.3px;
        position: relative;
        z-index: 2;
        padding-left: 68px; // 与头像对齐
      }

      .card-footer {
        height: 16px;
        position: relative;
        .ripple {
          display: block;
          width: 100%;
          height: 2px;
          background: linear-gradient(
            90deg,
            transparent,
            $accent-pink,
            $accent-aqua,
            transparent
          );
          opacity: 0.3;
          border-radius: 2px;
        }
      }
    }

    /* 无限滚动哨兵 */
    .sentinel {
      height: 1px;
      opacity: 0;
    }

    /* 加载更多指示器 */
    .loading-more {
      margin-bottom: 100px;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 12px;
      padding: 20px;
      color: rgba($accent-aqua, 0.7);
      font-size: 0.95rem;
      letter-spacing: 1px;

      .jelly-loader {
        width: 24px;
        height: 24px;
        border: 2px solid rgba($accent-aqua, 0.2);
        border-top-color: $accent-aqua;
        border-radius: 50%;
        animation: spin 1s infinite linear;
      }
    }

    .no-more {
      margin-bottom: 100px;
      text-align: center;
      padding: 20px;
      color: rgba($text-light, 0.2);
      font-style: italic;
      font-size: 0.9rem;
    }
  }

  /* ===== 底部表单：珊瑚台 ===== */
  .message-form {
    position: fixed;
    left: 50%;
    transform: translateX(-50%);
    bottom: 20px;
    width: calc(100% - 40px);
    max-width: 960px;
    z-index: 20;

    .coral-base {
      background: rgba($deep-bg, 0.7);
      backdrop-filter: blur(20px);
      -webkit-backdrop-filter: blur(20px);
      border-radius: 60px 60px 30px 30px;
      border: 1px solid rgba($accent-aqua, 0.2);
      border-bottom-width: 3px;
      border-bottom-color: rgba($accent-aqua, 0.4);
      box-shadow: 0 0 50px rgba($accent-aqua, 0.2);
      padding: 24px 28px;
      position: relative;
    }

    .form-container {
      display: flex;
      flex-direction: column;
      gap: 16px;
    }

    input,
    textarea {
      width: 100%;
      padding: 14px 20px;
      background: rgba(0, 0, 0, 0.3);
      border: 1px solid rgba($accent-aqua, 0.2);
      border-radius: 40px;
      color: $text-light;
      font-size: 1rem;
      transition: all 0.3s;
      font-weight: 300;

      &:focus {
        outline: none;
        border-color: $accent-aqua;
        box-shadow: 0 0 25px rgba($accent-aqua, 0.3);
        background: rgba(0, 0, 0, 0.5);
      }

      &::placeholder {
        color: rgba($text-light, 0.3);
        font-style: italic;
      }
    }

    textarea {
      min-height: 80px;
      resize: none;
      border-radius: 30px;
    }

    .form-actions {
      display: flex;
      justify-content: space-between;
      align-items: center;

      .hint {
        display: flex;
        align-items: center;
        gap: 8px;
        color: rgba($text-light, 0.5);
        font-size: 0.9rem;

        kbd {
          background: rgba($accent-deep-blue, 0.6);
          padding: 4px 10px;
          border-radius: 30px;
          border: 1px solid rgba($accent-aqua, 0.3);
          color: $accent-aqua;
          font-family: monospace;
        }
      }

      button {
        padding: 12px 36px;
        background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
        border: none;
        border-radius: 40px;
        color: $deep-bg;
        font-weight: 500;
        font-size: 1rem;
        cursor: pointer;
        transition: all 0.3s;
        box-shadow: 0 0 20px rgba($accent-aqua, 0.4);
        letter-spacing: 2px;

        &:hover:not(:disabled) {
          transform: translateY(-3px);
          box-shadow: 0 0 35px rgba($accent-aqua, 0.7);
        }

        &:disabled {
          opacity: 0.4;
          cursor: not-allowed;
        }
      }
    }
  }

  /* 动画 */
  @keyframes tendrilWave {
    0% {
      transform: translateY(0) scaleY(1);
      opacity: 0.5;
    }
    100% {
      transform: translateY(-8px) scaleY(1.2);
      opacity: 0.9;
    }
  }

  @keyframes tentacleWave {
    0% {
      transform: translateY(0) scaleY(1);
      opacity: 0.4;
    }
    100% {
      transform: translateY(-10px) scaleY(1.3);
      opacity: 0.8;
    }
  }

  @keyframes shimmer {
    0% {
      transform: translateX(-100%);
    }
    100% {
      transform: translateX(100%);
    }
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  /* 移动端适配 */
  @media (max-width: 720px) {
    padding: 80px 16px 200px;

    .carousel-desktop {
      display: none;
    }
    .carousel-mobile {
      display: block;
    }

    .board-header {
      .pearl {
        padding: 12px 24px;
        h1 {
          font-size: 1.6rem;
        }
      }
      .subtitle {
        font-size: 0.9rem;
      }
    }

    .message-card {
      .card-inner {
        padding: 18px 20px;
      }
      .message-content {
        padding-left: 0;
      }
    }

    .message-form {
      .coral-base {
        padding: 18px 20px;
      }
      .form-actions {
        flex-direction: column;
        gap: 12px;
        button {
          width: 100%;
        }
      }
    }
  }

  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
  }
}
</style>
