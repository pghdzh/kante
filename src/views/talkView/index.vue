<template>
  <div class="cantarella-chat">
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

    <!-- 头部：珍珠标题 + 统计信息 -->
    <header class="chat-header">
      <div class="header-pearl">
        <h1>翡萨烈·渊海问答</h1>
        <div class="header-stats">
          <div class="stat-badge">
            <span class="stat-label">今日</span>
            <span class="stat-value">{{ stats.dailyChats[today] || 0 }}</span>
          </div>
          <div class="stat-badge">
            <span class="stat-label">总对话</span>
            <span class="stat-value">{{ stats.totalChats }}</span>
          </div>
          <button class="stats-btn" @click="showModal = true" title="详细统计">
            📊
          </button>
        </div>
      </div>
    </header>

    <!-- 聊天主体 -->
    <div class="chat-container">
      <div class="messages" ref="msgList">
        <transition-group name="msg" tag="div">
          <div
            v-for="msg in chatLog"
            :key="msg.id"
            :class="[
              'message',
              msg.role,
              { error: msg.isError, egg: msg.isEgg },
            ]"
          >
            <div class="avatar" :class="msg.role"></div>
            <div class="bubble">
              <div class="content" v-html="msg.text"></div>
            </div>
          </div>
          <div v-if="loading" class="message bot" key="loading">
            <div class="avatar bot"></div>
            <div class="bubble loading">
              浮出水面
              <span class="dots">
                <span class="dot">.</span>
                <span class="dot">.</span>
                <span class="dot">.</span>
              </span>
            </div>
          </div>
        </transition-group>
      </div>

      <!-- 输入区 -->
      <div class="input-panel">
        <form class="input-area" @submit.prevent="sendMessage">
          <textarea
            v-model="input"
            placeholder="向坎特蕾拉提问…"
            :disabled="loading"
            @keydown="handleKeydown"
            rows="1"
          ></textarea>
          <div class="input-actions">
            <button
              type="button"
              class="action-btn"
              @click="clearChat"
              :disabled="loading"
              title="清空对话"
            >
              ✕
            </button>
            <button
              type="button"
              class="action-btn"
              :disabled="!chatLog.length || loading"
              @click="copyChat"
            >
              {{ copyButtonText }}
            </button>
            <button
              type="submit"
              class="send-btn"
              :disabled="!input.trim() || loading"
            >
              沉入幻梦
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- 详细统计弹窗 -->
    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-content">
        <h3>渊海秘录</h3>
        <ul class="detail-list">
          <li>总对话次数：{{ stats.totalChats }}</li>
          <li>首次使用：{{ formatDate(stats.firstTimestamp) }}</li>
          <li>活跃天数：{{ stats.activeDates.length }} 天</li>
          <li>今日对话：{{ stats.dailyChats[today] || 0 }} 次</li>
          <li>总使用时长：{{ formatDuration(stats.totalTime) }}</li>
          <li>当前连续：{{ stats.currentStreak }} 天</li>
          <li>最长连续：{{ stats.longestStreak }} 天</li>
          <li>
            最活跃日：{{ mostActiveDay }}（{{
              stats.dailyChats[mostActiveDay] || 0
            }}
            次）
          </li>
        </ul>
        <button class="close-btn" @click="showModal = false">关闭</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  reactive,
  ref,
  computed,
  onMounted,
  nextTick,
  watch,
  onBeforeUnmount,
} from "vue";
import { sendMessageToHui } from "@/api/deepseekApi";

const STORAGE_KEY = "hui_chat_log";
const STORAGE_STATS_KEY = "hui_chat_stats";
const showModal = ref(false);

interface Stats {
  firstTimestamp: number;
  totalChats: number;
  activeDates: string[];
  dailyChats: Record<string, number>;
  currentStreak: number;
  longestStreak: number;
  totalTime: number;
}

const defaultStats: Stats = {
  firstTimestamp: Date.now(),
  totalChats: 0,
  activeDates: [],
  dailyChats: {},
  currentStreak: 0,
  longestStreak: 0,
  totalTime: 0,
};

function loadStats(): Stats {
  const saved = localStorage.getItem(STORAGE_STATS_KEY);
  if (saved) {
    try {
      const parsed = JSON.parse(saved);
      return { ...defaultStats, ...parsed };
    } catch {
      console.warn("加载统计数据失败，使用默认值");
    }
  }
  return { ...defaultStats };
}

function saveStats() {
  localStorage.setItem(STORAGE_STATS_KEY, JSON.stringify(stats));
}

function updateActive(date: string) {
  if (!stats.activeDates.includes(date)) {
    stats.activeDates.push(date);
    updateStreak();
    saveStats();
  }
}

function updateStreak() {
  const dates = [...stats.activeDates].sort();
  let curr = 0,
    max = stats.longestStreak,
    prevTs = 0;
  const todayStr = new Date().toISOString().slice(0, 10);
  dates.forEach((d) => {
    const ts = new Date(d).getTime();
    if (prevTs && ts - prevTs === 86400000) curr++;
    else curr = 1;
    max = Math.max(max, curr);
    prevTs = ts;
  });
  stats.currentStreak = dates[dates.length - 1] === todayStr ? curr : 0;
  stats.longestStreak = max;
  saveStats();
}

function updateDaily(date: string) {
  stats.dailyChats[date] = (stats.dailyChats[date] || 0) + 1;
  saveStats();
}

const mostActiveDay = computed(() => {
  let day = "",
    max = 0;
  for (const [d, c] of Object.entries(stats.dailyChats)) {
    if (c > max) {
      max = c;
      day = d;
    }
  }
  return day || new Date().toISOString().slice(0, 10);
});

function formatDuration(ms: number): string {
  const totalMin = Math.floor(ms / 60000);
  const h = Math.floor(totalMin / 60);
  const m = totalMin % 60;
  return h ? `${h} 小时 ${m} 分钟` : `${m} 分钟`;
}

function formatDate(timestamp: number): string {
  return new Date(timestamp).toISOString().slice(0, 10);
}

const stats = reactive<Stats>(loadStats());
const sessionStart = Date.now();

interface ChatMsg {
  id: number;
  role: "user" | "bot";
  text: string;
  isError?: boolean;
  isEgg?: boolean;
}

const chatLog = ref<ChatMsg[]>(loadChatLog());
const input = ref("");
const loading = ref(false);
const msgList = ref<HTMLElement>();

const today = new Date().toISOString().slice(0, 10);

async function sendMessage() {
  if (!input.value.trim()) return;
  if (stats.totalChats === 0 && !localStorage.getItem(STORAGE_STATS_KEY)) {
    stats.firstTimestamp = Date.now();
    saveStats();
  }
  stats.totalChats++;
  updateActive(today);
  updateDaily(today);
  saveStats();

  const userText = input.value;
  chatLog.value.push({
    id: Date.now(),
    role: "user",
    text: userText,
  });
  input.value = "";
  loading.value = true;

  try {
    const history = chatLog.value.filter((msg) => !msg.isEgg && !msg.isError);
    const botReply = await sendMessageToHui(userText, history);
    if (botReply == "error") {
      chatLog.value.push({
        id: Date.now() + 2,
        role: "bot",
        text: "API余额耗尽了，去b站提醒我充钱吧",
        isError: true,
      });
    } else {
      chatLog.value.push({
        id: Date.now() + 1,
        role: "bot",
        text: botReply,
      });
    }
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
    await scrollToBottom();
  }
}

function handleKeydown(e: KeyboardEvent) {
  if (e.key === "Enter") sendMessage();
}

function clearChat() {
  if (confirm("确定要清空全部对话吗？")) {
    chatLog.value = [
      {
        id: Date.now(),
        role: "bot",
        text: "要留下来喝杯茶吗？普通的茶，不苦的——和我熬的秘药不一样，很简单，很温暖。你会喜欢的。",
      },
    ];
    localStorage.removeItem(STORAGE_KEY);
  }
}

function loadChatLog(): ChatMsg[] {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      return JSON.parse(saved);
    } catch (e) {
      console.error("chatLog 解析失败：", e);
    }
  }
  return [
    {
      id: Date.now(),
      role: "bot",
      text: "要留下来喝杯茶吗？普通的茶，不苦的——和我熬的秘药不一样，很简单，很温暖。你会喜欢的。",
    },
  ];
}

async function scrollToBottom() {
  await nextTick();
  if (msgList.value) {
    msgList.value.scrollTop = msgList.value.scrollHeight;
  }
}

const copyButtonText = ref("复制");
let _copyTimer: number | null = null;

function stripHtml(html = ""): string {
  if (typeof document === "undefined") {
    return html.replace(/<br\s*\/?>/gi, "\n").replace(/<[^>]+>/g, "");
  }
  const div = document.createElement("div");
  const withBreaks = String(html).replace(/<br\s*\/?>/gi, "\n");
  div.innerHTML = withBreaks;
  return div.textContent || div.innerText || "";
}

function buildChatPlainText(): string {
  return chatLog.value
    .map((msg) => {
      const time = new Date(msg.id).toLocaleString();
      const who = msg.role === "user" ? "你" : "坎特蕾拉";
      const text = stripHtml(msg.text);
      return `[${time}] ${who}: ${text}`;
    })
    .join("\n\n");
}

function fallbackCopyText(text: string) {
  const textarea = document.createElement("textarea");
  textarea.value = text;
  textarea.style.position = "fixed";
  textarea.style.left = "-9999px";
  textarea.style.top = "0";
  document.body.appendChild(textarea);
  textarea.select();
  textarea.setSelectionRange(0, textarea.value.length);
  const ok = document.execCommand("copy");
  document.body.removeChild(textarea);
  if (!ok) throw new Error("execCommand copy failed");
}

async function copyChat() {
  const text = buildChatPlainText();
  if (!text) {
    copyButtonText.value = "无内容可复制";
    clearTimeout(_copyTimer as number);
    _copyTimer = window.setTimeout(() => (copyButtonText.value = "复制"), 1600);
    return;
  }

  try {
    if (navigator.clipboard && navigator.clipboard.writeText) {
      await navigator.clipboard.writeText(text);
    } else {
      fallbackCopyText(text);
    }
    copyButtonText.value = "已复制";
  } catch (err) {
    try {
      fallbackCopyText(text);
      copyButtonText.value = "已复制";
    } catch (e) {
      console.error("复制失败：", e);
      copyButtonText.value = "复制失败";
    }
  } finally {
    clearTimeout(_copyTimer as number);
    _copyTimer = window.setTimeout(() => (copyButtonText.value = "复制"), 1600);
  }
}

watch(
  chatLog,
  async () => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(chatLog.value));
    await scrollToBottom();
  },
  { deep: true }
);

function handleBeforeUnload() {
  stats.totalTime += Date.now() - sessionStart;
  saveStats();
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

const desktopImages = ref<string[]>(shuffle(allSrcs).slice(0, 5));
const mobileImages = ref<string[]>(shuffle(allSrcs2).slice(0, 5));
const currentIndex = ref(0);
let carouselTimer: number | undefined;

onMounted(() => {
  scrollToBottom();
  window.addEventListener("beforeunload", handleBeforeUnload);
  carouselTimer = window.setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % desktopImages.value.length;
  }, 5200);
});

onBeforeUnmount(() => {
  window.removeEventListener("beforeunload", handleBeforeUnload);
  if (_copyTimer) clearTimeout(_copyTimer);
  if (carouselTimer) clearInterval(carouselTimer);
});
</script>

<style scoped lang="scss">
/* 坎特蕾拉色板 */
$deep-bg: #030614;
$mid-bg: #14213d;
$accent-lavender: #b8a9ff;
$accent-aqua: #7ae2ff;
$accent-pink: #ffb3c6;
$accent-deep-blue: #2a4a7a;
$text-light: #f0f5fe;
$card-bg: rgba(255, 255, 255, 0.02);
$glass-edge: rgba($accent-aqua, 0.2);

.cantarella-chat {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(180deg, $deep-bg 0%, $mid-bg 100%);
  color: $text-light;
  overflow-x: hidden;
  padding-top: 80px;

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

  /* 头部珍珠 */
  .chat-header {
    position: relative;
    z-index: 10;
    max-width: 960px;
    margin: 0 auto 30px;

    .header-pearl {
      background: rgba(6, 10, 20, 0.6);
      backdrop-filter: blur(12px);
      border: 1px solid $glass-edge;
      border-radius: 120px;
      padding: 16px 24px;
      box-shadow: 0 0 50px rgba($accent-aqua, 0.2),
        0 20px 40px rgba(0, 0, 0, 0.4);
      display: flex;
      align-items: center;
      justify-content: space-between;
      flex-wrap: wrap;
      gap: 16px;

      h1 {
        margin: 0;
        font-size: 1.8rem;
        font-weight: 400;
        background: linear-gradient(
          135deg,
          #fff,
          $accent-lavender,
          $accent-aqua
        );
        -webkit-background-clip: text;
        background-clip: text;
        color: transparent;
        letter-spacing: 2px;
      }

      .header-stats {
        display: flex;
        gap: 16px;
        align-items: center;

        .stat-badge {
          display: flex;
          flex-direction: column;
          align-items: center;
          background: rgba(0, 0, 0, 0.3);
          padding: 6px 12px;
          border-radius: 40px;
          border: 1px solid $glass-edge;

          .stat-label {
            font-size: 0.7rem;
            opacity: 0.7;
          }
          .stat-value {
            font-size: 1.2rem;
            font-weight: 600;
            color: $accent-aqua;
          }
        }

        .stats-btn {
          width: 40px;
          height: 40px;
          border-radius: 50%;
          background: rgba($accent-deep-blue, 0.5);
          border: 1px solid $glass-edge;
          color: $accent-aqua;
          font-size: 1.2rem;
          cursor: pointer;
          transition: all 0.2s;

          &:hover {
            transform: scale(1.1);
            background: rgba($accent-aqua, 0.2);
          }
        }
      }
    }
  }

  /* 聊天主体 */
  .chat-container {
    position: relative;
    z-index: 5;
    max-width: 960px;
    margin: 0 auto;
  }

  .messages {
    max-height: 60vh;
    overflow-y: auto;
    padding: 20px 10px 30px;
    scroll-behavior: smooth;

    &::-webkit-scrollbar {
      width: 6px;
    }
    &::-webkit-scrollbar-thumb {
      background: $accent-aqua;
      border-radius: 6px;
    }
  }

  /* 消息卡片 */
  .message {
    display: flex;
    align-items: flex-start;
    margin-bottom: 20px;

    &.user {
      flex-direction: row-reverse;
    }

    .avatar {
      width: 48px;
      height: 48px;
      border-radius: 50%;
      margin: 0 10px;
      background-size: cover;
      background-position: center;
      flex-shrink: 0;
      z-index: 10;
      box-shadow: 0 8px 28px rgba(2, 6, 12, 0.6);
      border: 1px solid rgba($text-light, 0.03);
      background-clip: padding-box;

      &.bot {
        background-image: url("@/assets/avatar/changli.png");
        box-shadow: 0 10px 34px rgba($accent-aqua, 0.06),
          inset 0 1px 0 rgba($accent-lavender, 0.03);
        border: 2px solid $accent-aqua;
        transform: scaleX(-1);
      }

      &.user {
        background: linear-gradient(
          180deg,
          rgba(6, 6, 10, 1),
          rgba(6, 6, 10, 0.96)
        );
        box-shadow: 0 8px 22px rgba(2, 6, 12, 0.6);
        border: 1px solid rgba($accent-aqua, 0.02);
      }
    }

    .bubble {
      max-width: 70%;
      background: $card-bg;
      backdrop-filter: blur(8px);
      border: 1px solid $glass-edge;
      border-radius: 40px 40px 30px 30px;
      padding: 14px 18px;
      box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
      position: relative;
      transition: all 0.2s;

      &::before {
        content: "";
        position: absolute;
        top: -8px;
        left: 10%;
        width: 80%;
        height: 20px;
        background: radial-gradient(
          ellipse at center,
          rgba($accent-aqua, 0.15),
          transparent 70%
        );
        filter: blur(8px);
        pointer-events: none;
      }

      &:hover {
        transform: translateY(-2px);
        border-color: rgba($accent-aqua, 0.4);
        box-shadow: 0 20px 30px rgba($accent-aqua, 0.2);
      }

      .content {
        word-break: break-word;
        line-height: 1.6;
      }

      &.loading {
        color: rgba($text-light, 0.8);
      }

      .dots {
        display: inline-flex;
        margin-left: 6px;
        .dot {
          animation: dotBlink 1.4s infinite;
          font-size: 1.2rem;
          &:nth-child(2) {
            animation-delay: 0.2s;
          }
          &:nth-child(3) {
            animation-delay: 0.4s;
          }
        }
      }
    }

    &.user .bubble {
      border-radius: 40px 40px 30px 30px;
      background: rgba($accent-deep-blue, 0.4);
    }

    &.error .bubble {
      border-color: rgba($accent-pink, 0.3);
    }
  }

  /* 输入区 */
  .input-panel {
    position: fixed;
    width: 100%;
    max-width: 980px;
    bottom: 20px;
    margin-top: 30px;
  }

  .input-area {
    background: rgba(6, 10, 20, 0.7);
    backdrop-filter: blur(16px);
    border: 1px solid $glass-edge;
    border-radius: 30px;
    padding: 20px;
    box-shadow: 0 0 60px rgba($accent-aqua, 0.2);
    display: flex;
    flex-direction: column;
    gap: 16px;

    textarea {
      width: 100%;
      padding: 14px 20px;
      background: rgba(0, 0, 0, 0.3);
      border: 1px solid rgba($accent-aqua, 0.2);
      border-radius: 30px;
      color: $text-light;
      font-size: 1rem;
      resize: none;
      transition: all 0.2s;

      &:focus {
        outline: none;
        border-color: $accent-aqua;
        box-shadow: 0 0 20px rgba($accent-aqua, 0.3);
      }

      &::placeholder {
        color: rgba($text-light, 0.3);
        font-style: italic;
      }
    }

    .input-actions {
      display: flex;
      gap: 12px;
      justify-content: flex-end;
      align-items: center;

      .action-btn {
        padding: 8px 16px;
        background: rgba(0, 0, 0, 0.3);
        border: 1px solid $glass-edge;
        border-radius: 30px;
        color: $text-light;
        cursor: pointer;
        transition: all 0.2s;

        &:hover:not(:disabled) {
          border-color: $accent-aqua;
          box-shadow: 0 0 15px rgba($accent-aqua, 0.3);
        }
        &:disabled {
          opacity: 0.4;
          cursor: not-allowed;
        }
      }

      .send-btn {
        padding: 8px 32px;
        background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
        border: none;
        border-radius: 30px;
        color: $deep-bg;
        font-weight: 600;
        cursor: pointer;
        box-shadow: 0 0 20px rgba($accent-aqua, 0.4);
        transition: all 0.2s;

        &:hover:not(:disabled) {
          transform: translateY(-2px);
          box-shadow: 0 0 30px rgba($accent-aqua, 0.7);
        }
        &:disabled {
          opacity: 0.4;
          cursor: not-allowed;
        }
      }
    }
  }

  /* 统计弹窗 */
  .modal-overlay {
    position: fixed;
    inset: 0;
    z-index: 2000;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(2, 6, 10, 0.8);
    backdrop-filter: blur(12px);
  }

  .modal-content {
    width: 400px;
    background: rgba(6, 10, 20, 0.9);
    backdrop-filter: blur(16px);
    border: 1px solid $glass-edge;
    border-radius: 30px;
    padding: 30px;
    box-shadow: 0 0 80px rgba($accent-aqua, 0.2);

    h3 {
      text-align: center;
      font-size: 1.8rem;
      font-weight: 400;
      background: linear-gradient(135deg, #fff, $accent-lavender, $accent-aqua);
      -webkit-background-clip: text;
      background-clip: text;
      color: transparent;
      margin: 0 0 20px;
    }

    .detail-list {
      list-style: none;
      padding: 0;
      margin: 0 0 30px;

      li {
        padding: 8px 0;
        border-bottom: 1px dashed rgba($accent-aqua, 0.2);
        display: flex;
        justify-content: space-between;
      }
    }

    .close-btn {
      display: block;
      width: 100%;
      padding: 12px;
      background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
      border: none;
      border-radius: 30px;
      color: $deep-bg;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.2s;

      &:hover {
        transform: translateY(-2px);
        box-shadow: 0 0 30px rgba($accent-aqua, 0.5);
      }
    }
  }

  /* 动画 */
  @keyframes dotBlink {
    0%,
    100% {
      opacity: 0.3;
      transform: translateY(0);
    }
    50% {
      opacity: 1;
      transform: translateY(-4px);
    }
  }

  /* 移动端适配 */
  @media (max-width: 720px) {
    padding-top: 60px;

    .carousel-desktop {
      display: none;
    }
    .carousel-mobile {
      display: block;
    }

    .chat-header .header-pearl {
      flex-direction: column;
      text-align: center;
      h1 {
        font-size: 1.4rem;
      }
    }

    .message .bubble {
      max-width: 85%;
    }

    .input-area .input-actions {
      flex-wrap: wrap;
      justify-content: center;
    }
  }
}
</style>
