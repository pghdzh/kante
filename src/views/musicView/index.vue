<template>
  <section
    class="cantarella-player"
    @keydown.space.prevent="onSpace"
    tabindex="0"
    ref="rootEl"
    aria-label="坎特蕾拉 音乐播放器"
  >
    <div class="stage">
      <!-- 左侧：封面与控制 -->
      <div class="left" role="region" aria-label="播放器控制区">
        <div class="cover" :style="coverStyle">
          <!-- video 作为纯装饰性背景，屏读器隐藏 -->
          <video
            v-if="videoSrc"
            class="video-background"
            :src="videoSrc"
            autoplay
            muted
            loop
            playsinline
            aria-hidden="true"
            tabindex="-1"
            :class="videoClass"
          ></video>

          <!-- 加载遮罩 -->
          <div v-if="loadingAudio" class="loading-overlay" aria-hidden="true">
            <div class="spinner" />
            <div class="loading-text">加载中…</div>
          </div>
        </div>

        <div class="controls">
          <div class="title" :title="current?.title || '未选择曲目'">
            {{ current?.title || "未选择曲目" }}
          </div>

          <div class="meta">
            <span class="time">{{ formatTime(currentTime) }}</span>
            <span class="divider">/</span>
            <span class="time">{{ formatTime(duration) }}</span>
          </div>

          <!-- 进度条 -->
          <div
            class="progress-wrap"
            ref="progressWrap"
            @click="seekByClick"
            @pointerdown.prevent="onPointerDownProgress"
            role="slider"
            :aria-valuemin="0"
            :aria-valuemax="duration"
            :aria-valuenow="currentTime"
            aria-label="进度条"
          >
            <div class="progress-bar">
              <div
                class="progress"
                :style="{ width: progressPercent + '%' }"
              ></div>
            </div>
            <div
              class="progress-handle"
              :style="{ left: progressPercent + '%' }"
              aria-hidden="true"
            ></div>
          </div>

          <!-- 控件行 -->
          <div class="btns">
            <button class="icon" @click="prev" aria-label="上一首">⟵</button>

            <button
              class="play"
              @click="togglePlay"
              :aria-pressed="playing"
              :aria-label="playing ? '暂停' : '播放'"
            >
              <span v-if="!playing">▶</span>
              <span v-else>▌▌</span>
            </button>

            <button class="icon" @click="next" aria-label="下一首">⟶</button>

            <div class="modes" role="group" aria-label="播放模式">
              <button
                :class="{ active: shuffle }"
                @click="toggleShuffle"
                title="随机播放"
              >
                🔀
              </button>
              <button
                :class="{ active: repeatMode !== 'off' }"
                @click="toggleRepeat"
                title="循环模式"
              >
                🔁
              </button>
            </div>

            <div class="volume" aria-label="音量控制">
              <input
                type="range"
                min="0"
                max="1"
                step="0.01"
                v-model.number="volume"
                aria-label="音量"
              />
            </div>
          </div>

          <div v-if="errorMessage" class="error-msg" role="status">
            {{ errorMessage }}
          </div>
        </div>
      </div>

      <!-- 右侧：播放列表 -->
      <div class="right" role="region" aria-label="播放列表">
        <div class="playlist-header">
          <div class="left-head">
            <h3>播放列表</h3>

            <div class="api-hint">
              {{ loading ? "加载中…" : list.length ? "" : "目录为空" }}
            </div>
          </div>

          <div class="search-wrap">
            <input
              v-model="searchTerm"
              @input="onSearchInput"
              placeholder="搜索曲名..."
              aria-label="搜索曲目"
            />
            <button
              v-if="searchTerm"
              class="clear"
              @click="clearSearch"
              aria-label="清除搜索"
            >
              ✕
            </button>
          </div>
        </div>

        <div class="list-area">
          <div v-if="loading" class="list-loading">
            <div class="small-spinner" />
            加载目录...
          </div>

          <ul class="playlist" role="list">
            <li
              v-for="(item, idx) in filteredList"
              :key="item.name || idx"
              :class="{ active: idx === index }"
              @click="selectTrack(idx)"
              tabindex="0"
              @keyup.enter="selectTrack(idx)"
              role="listitem"
              :aria-current="idx === index ? 'true' : 'false'"
            >
              <div class="left-col">
                <div class="dot" aria-hidden="true"></div>
                <div class="title" :title="item.title">{{ item.title }}</div>
              </div>
              <div class="right-col">
                <div class="len">
                  {{ item.duration ? formatTime(item.duration) : "--:--" }}
                </div>
              </div>
            </li>
          </ul>
        </div>

     
      </div>
    </div>

    <!-- audio 元素 -->
    <audio
      ref="audioRef"
      @timeupdate="onTimeUpdate"
      @ended="onEnded"
      @loadedmetadata="onLoadedMetadata"
      @error="onAudioError"
      preload="metadata"
    ></audio>
  </section>
</template>

<script setup lang="ts">
// 脚本部分完全保留原逻辑，不做任何改动
import {
  ref,
  computed,
  onMounted,
  onBeforeUnmount,
  watch,
  nextTick,
} from "vue";
import { getMusicList, getMusicUrl } from "@/api/modules/music"; // 请确认路径

type MusicItem = {
  name: string;
  title: string;
  url?: string;
  duration?: number | null;
};

const list = ref<MusicItem[]>([]);
const loading = ref(false);
const index = ref<number>(-1);
const playing = ref(false);
const audioRef = ref<HTMLAudioElement | null>(null);
const currentTime = ref<number>(0);
const duration = ref<number>(0);
const volume = ref<number>(
  Number(localStorage.getItem("cantarella_volume") ?? 0.8)
);
const shuffle = ref<boolean>(false);
const repeatMode = ref<"off" | "one" | "all">("off");

const rootEl = ref<HTMLElement | null>(null);
const progressWrap = ref<HTMLElement | null>(null);
const dragging = ref(false);

const errorMessage = ref<string | null>(null);
const loadingAudio = ref(false);

// 根据窗口宽度判断移动端
const isMobile = ref<boolean>(window.innerWidth <= 920);
window.addEventListener("resize", () => {
  isMobile.value = window.innerWidth <= 920;
});

// 随机选择视频源（mp1/mp2 里）
const videoSrc = ref("");
const videoClass = ref(""); // "landscape" or "portrait"

// 搜索相关
const searchTerm = ref("");
let searchTimer: any = null;
const searchDebounceMs = 240;

// 当前曲目与进度计算
const current = computed(() =>
  index.value >= 0 && list.value[index.value] ? list.value[index.value] : null
);
const progressPercent = computed(() =>
  duration.value
    ? Math.min(100, Math.max(0, (currentTime.value / duration.value) * 100))
    : 0
);

// 封面渐变（坎特蕾拉风格）
const coverStyle = computed(() => {
  const t = current.value?.title || "cantarella";
  let hash = 0;
  for (let i = 0; i < t.length; i++)
    hash = (hash << 5) - hash + t.charCodeAt(i);
  const r1 = (Math.abs(hash) % 120) + 40;
  const r2 = (Math.abs(hash * 3) % 120) + 40;
  return {
    background: `radial-gradient(circle at 28% 28%, rgba(95,224,255,0.06), transparent 12%), linear-gradient(135deg, rgba(${r1},${r2},255,0.08), rgba(111,92,230,0.08))`,
  };
});

// 过滤后的列表（基于搜索）
const filteredList = computed(() => {
  const term = (searchTerm.value || "").trim().toLowerCase();
  if (!term) return list.value;
  return list.value.filter((i) => (i.title || "").toLowerCase().includes(term));
});

// 获取列表
async function fetchList() {
  loading.value = true;
  try {
    const res = await getMusicList();
    const items =
      res?.ok && Array.isArray(res.list)
        ? res.list
        : Array.isArray(res)
        ? res
        : res?.list ?? [];
    list.value = items.map((it: any) => ({
      name: it.name,
      title: it.title ?? (it.name ? it.name.replace(/\.mp3$/i, "") : "未知"),
      url: getMusicUrl(it.name),
      duration: null,
    }));
  } catch (e) {
    console.error("获取音乐列表失败", e);
    list.value = [];
    errorMessage.value = "加载目录失败";
  } finally {
    loading.value = false;
  }
}

// 设置并预检音源
async function safeSetSrc(url: string) {
  const a = audioRef.value!;
  errorMessage.value = null;
  loadingAudio.value = true;
  try {
    try {
      const head = await fetch(url, { method: "HEAD" });
      if (!head.ok) throw new Error(`资源响应 ${head.status}`);
      const ct = head.headers.get("content-type") || "";
      if (!ct.includes("audio")) {
        console.warn("content-type 不是 audio:", ct);
      }
    } catch (e) {
      // 忽略 HEAD 失败
    }
    a.src = url;
    a.load();
  } catch (err) {
    console.error("设置音源失败", err);
    errorMessage.value = "无法加载音频资源";
    throw err;
  } finally {
    // loadingAudio 会在 loadedmetadata 或 error 中被关闭
  }
}

async function loadCurrent(doPlay = false) {
  const a = audioRef.value;
  const curr = current.value;
  if (!a || !curr) return;
  a.pause();
  duration.value = 0;
  currentTime.value = 0;
  try {
    await safeSetSrc(curr.url || getMusicUrl(curr.name));
    if (doPlay) await play();
  } catch {
    playing.value = false;
    loadingAudio.value = false;
  }
}

async function play() {
  const a = audioRef.value;
  if (!a) return;
  try {
    await a.play();
    playing.value = true;
    errorMessage.value = null;
  } catch (e: any) {
    console.warn("播放失败", e);
    playing.value = false;
    errorMessage.value = "播放被浏览器阻止或资源不可用";
  }
}
function pause() {
  audioRef.value?.pause();
  playing.value = false;
}
function togglePlay() {
  if (!audioRef.value) return;
  if (playing.value) pause();
  else play();
}

function selectTrack(i: number) {
  if (i < 0 || i >= list.value.length) return;
  index.value = i;
  loadCurrent(true);
}

// 音频事件
function onTimeUpdate(e: Event) {
  const t = e.target as HTMLAudioElement;
  currentTime.value = t.currentTime || 0;
}
function onLoadedMetadata(e: Event) {
  const t = e.target as HTMLAudioElement;
  duration.value = isFinite(t.duration) ? t.duration : 0;
  if (current.value && !current.value.duration)
    current.value.duration = duration.value;
  loadingAudio.value = false;
}
function onEnded() {
  loadingAudio.value = false;
  if (repeatMode.value === "one") {
    if (audioRef.value) {
      audioRef.value.currentTime = 0;
      play();
    }
    return;
  }
  if (shuffle.value) {
    playRandom();
    return;
  }
  if (index.value < list.value.length - 1) selectTrack(index.value + 1);
  else {
    if (repeatMode.value === "all") selectTrack(0);
    else playing.value = false;
  }
}
function onAudioError(e: Event) {
  const a = audioRef.value;
  console.error("audio error", a?.error);
  errorMessage.value = "音频播放出错";
  playing.value = false;
  loadingAudio.value = false;
}

// 上一/下一/随机
function next() {
  if (!list.value.length) return;
  if (shuffle.value) {
    playRandom();
    return;
  }
  if (index.value < list.value.length - 1) selectTrack(index.value + 1);
  else if (repeatMode.value === "all") selectTrack(0);
}
function prev() {
  if (!audioRef.value) return;
  if (audioRef.value.currentTime > 4) {
    audioRef.value.currentTime = 0;
    return;
  }
  if (index.value > 0) selectTrack(index.value - 1);
  else if (repeatMode.value === "all") selectTrack(list.value.length - 1);
}
function playRandom() {
  if (!list.value.length) return;
  if (list.value.length === 1) {
    selectTrack(0);
    return;
  }
  let i = index.value;
  while (i === index.value) i = Math.floor(Math.random() * list.value.length);
  selectTrack(i);
}

// 进度条：点击 & 拖拽
function seekByClick(e: MouseEvent | TouchEvent) {
  if (!progressWrap.value || !duration.value || !audioRef.value) return;
  const rect = progressWrap.value.getBoundingClientRect();
  const clientX =
    (e as MouseEvent).clientX ?? (e as TouchEvent).touches?.[0]?.clientX;
  if (clientX == null) return;
  const x = Math.min(Math.max(0, clientX - rect.left), rect.width);
  const ratio = x / rect.width;
  audioRef.value.currentTime = ratio * duration.value;
  currentTime.value = audioRef.value.currentTime;
}
function onPointerDownProgress(e: PointerEvent) {
  if (!progressWrap.value || !audioRef.value || !duration.value) return;
  dragging.value = true;
  (e.target as Element).setPointerCapture?.(e.pointerId);
  window.addEventListener("pointermove", onPointerMoveProgress);
  window.addEventListener("pointerup", onPointerUpProgress);
  handlePointer(e);
}
function onPointerMoveProgress(e: PointerEvent) {
  handlePointer(e);
}
function onPointerUpProgress(e: PointerEvent) {
  dragging.value = false;
  window.removeEventListener("pointermove", onPointerMoveProgress);
  window.removeEventListener("pointerup", onPointerUpProgress);
}
function handlePointer(e: PointerEvent) {
  if (!progressWrap.value || !audioRef.value || !duration.value) return;
  const rect = progressWrap.value.getBoundingClientRect();
  const x = Math.min(Math.max(0, e.clientX - rect.left), rect.width);
  const ratio = x / rect.width;
  audioRef.value.currentTime = ratio * duration.value;
  currentTime.value = audioRef.value.currentTime;
}

// 音量持久化
watch(volume, (v) => {
  if (audioRef.value) audioRef.value.volume = v;
  localStorage.setItem("cantarella_volume", String(v));
});

// 模式切换
function toggleShuffle() {
  shuffle.value = !shuffle.value;
}
function toggleRepeat() {
  if (repeatMode.value === "off") repeatMode.value = "all";
  else if (repeatMode.value === "all") repeatMode.value = "one";
  else repeatMode.value = "off";
}

// 键盘空格
function onSpace() {
  if (
    document.activeElement &&
    ["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)
  )
    return;
  togglePlay();
}

// 搜索防抖
function onSearchInput() {
  if (searchTimer) clearTimeout(searchTimer);
  searchTimer = setTimeout(() => {
    searchTimer = null;
  }, searchDebounceMs);
}
function clearSearch() {
  searchTerm.value = "";
}

// 格式化时间
function formatTime(sec?: number) {
  if (!sec || !isFinite(sec)) return "--:--";
  const s = Math.floor(sec % 60);
  const m = Math.floor(sec / 60);
  return `${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`;
}

// 生命周期：绑定 audio ref、初始化列表和 videoSrc
onMounted(async () => {
  audioRef.value =
    (document.querySelector(".cantarella-player audio") as HTMLAudioElement) ??
    null;
  if (audioRef.value) audioRef.value.volume = volume.value;

  // 设置 videoSrc 与 class：桌面优先横屏(mp1), 移动优先竖屏(mp2)
  const isM = isMobile.value;
  const folder = isM ? "/mp1" : "/mp2";
  const idx = Math.floor(Math.random() * 4) + 1;
  videoSrc.value = `${folder}/1 (${idx}).mp4`;
  videoClass.value = isM ? "landscape" : "portrait";

  await fetchList();

  window.addEventListener("keydown", globalKeydown);
});

onBeforeUnmount(() => {
  window.removeEventListener("keydown", globalKeydown);
});

// 全局键盘
function globalKeydown(e: KeyboardEvent) {
  if (e.code === "Space") {
    if (
      document.activeElement &&
      ["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)
    )
      return;
    e.preventDefault();
    togglePlay();
  } else if (e.code === "Escape") {
    pause();
  }
}
</script>

<style scoped lang="scss">
/* 坎特蕾拉色板 - 深海幻梦 */
$deep-bg: #030614; // 极深海
$mid-bg: #14213d; // 深海中层
$accent-lavender: #b8a9ff; // 薰衣草紫
$accent-aqua: #7ae2ff; // 水母荧光
$accent-pink: #ffb3c6; // 毒药粉
$accent-deep-blue: #2a4a7a; // 深海蓝
$text-light: #f0f5fe;
$card-bg: rgba(255, 255, 255, 0.02);
$glass-edge: rgba($accent-aqua, 0.2);
$glass-bg: rgba(6, 10, 20, 0.5);

.cantarella-player {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(180deg, $deep-bg 0%, $mid-bg 100%);
  color: $text-light;
  padding: 80px 20px 60px;
  font-family: "Noto Sans SC", "Inter", system-ui, sans-serif;
  outline: none;
  -webkit-font-smoothing: antialiased;

  /* 五线谱微纹理 */
  &::before {
    content: "";
    position: fixed;
    left: 0;
    top: 100px;
    width: 100%;
    height: 200px;
    background-image: repeating-linear-gradient(
      to bottom,
      rgba(255, 255, 255, 0.01) 0px,
      rgba(255, 255, 255, 0.01) 1px,
      transparent 1px,
      transparent 18px
    );
    opacity: 0.04;
    pointer-events: none;
    mix-blend-mode: overlay;
  }

  /* 主区域布局 */
  .stage {
    display: flex;
    gap: 24px;
    max-width: 1200px;
    margin: 0 auto;
    align-items: flex-start;
  }

  /* 左侧卡片 */
  .left {
    width: 420px;
    background: $glass-bg;
    backdrop-filter: blur(12px);
    border: 1px solid $glass-edge;
    border-radius: 40px 40px 30px 30px;
    padding: 20px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
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
        rgba($accent-aqua, 0.15),
        transparent 70%
      );
      filter: blur(10px);
      pointer-events: none;
    }
  }

  /* cover 区 */
  .cover {
    width: 100%;
    height: 640px;
    border-radius: 30px;
    overflow: hidden;
    position: relative;
    box-shadow: inset 0 -6px 28px rgba(0, 0, 0, 0.45);
    border: 1px solid $glass-edge;

    .video-background {
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0.7;
      transition: opacity 0.3s;

      &.landscape {
        aspect-ratio: 16/9;
      }
      &.portrait {
        aspect-ratio: 9/16;
        width: auto;
        height: 110%;
      }
    }
  }

  /* 加载遮罩 */
  .loading-overlay {
    position: absolute;
    inset: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: linear-gradient(180deg, rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.3));
    backdrop-filter: blur(4px);
    z-index: 8;

    .spinner {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      border: 3px solid rgba($accent-aqua, 0.2);
      border-top-color: $accent-aqua;
      animation: spin 1s infinite linear;
    }
    .loading-text {
      margin-top: 12px;
      color: $accent-aqua;
      font-weight: 300;
      letter-spacing: 1px;
    }
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  /* 控件区 */
  .controls {
    margin-top: 20px;

    .title {
      font-size: 1.2rem;
      font-weight: 400;
      color: $accent-aqua;
      letter-spacing: 0.5px;
      margin-bottom: 8px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .meta {
      display: flex;
      gap: 8px;
      color: rgba($text-light, 0.7);
      font-size: 0.9rem;
      margin-bottom: 12px;

      .time {
        font-family: monospace;
      }
    }
  }

  /* 进度条 */
  .progress-wrap {
    cursor: pointer;
    touch-action: none;
    margin-bottom: 16px;

    .progress-bar {
      height: 8px;
      background: rgba(255, 255, 255, 0.05);
      border-radius: 4px;
      overflow: hidden;
      position: relative;

      .progress {
        height: 100%;
        background: linear-gradient(90deg, $accent-lavender, $accent-aqua);
        border-radius: 4px;
        transition: width 0.1s linear;
      }
    }

    .progress-handle {
      width: 16px;
      height: 16px;
      border-radius: 50%;
      background: $accent-aqua;
      transform: translateX(-50%);
      position: relative;
      top: -12px;
      box-shadow: 0 0 15px $accent-aqua;
      transition: transform 0.1s;
      pointer-events: none;
    }
  }

  /* 按钮行 */
  .btns {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;

    .icon,
    .play {
      background: transparent;
      border: none;
      color: $text-light;
      font-size: 1.2rem;
      cursor: pointer;
      padding: 8px;
      border-radius: 40% 60% 40% 60%;
      transition: all 0.2s;

      &:hover {
        transform: scale(1.1);
        background: rgba($accent-aqua, 0.1);
        box-shadow: 0 0 15px $accent-aqua;
      }
    }

    .play {
      background: linear-gradient(
        135deg,
        rgba($accent-lavender, 0.2),
        rgba($accent-aqua, 0.2)
      );
      padding: 8px 16px;
      font-size: 1.4rem;
      border-radius: 40px;
    }

    .modes {
      display: flex;
      gap: 8px;
      margin-left: auto;

      button {
        background: transparent;
        border: 1px solid $glass-edge;
        border-radius: 30px;
        padding: 6px 12px;
        color: $text-light;
        cursor: pointer;
        transition: all 0.2s;

        &.active {
          background: rgba($accent-aqua, 0.15);
          border-color: $accent-aqua;
          box-shadow: 0 0 10px $accent-aqua;
        }

        &:hover {
          border-color: $accent-aqua;
        }
      }
    }

    .volume input[type="range"] {
      width: 100px;
      height: 4px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 2px;
      outline: none;
      -webkit-appearance: none;

      &::-webkit-slider-thumb {
        -webkit-appearance: none;
        width: 12px;
        height: 12px;
        border-radius: 50%;
        background: $accent-aqua;
        cursor: pointer;
        box-shadow: 0 0 10px $accent-aqua;
      }
    }
  }

  .error-msg {
    margin-top: 12px;
    color: $accent-pink;
    font-size: 0.9rem;
    text-align: center;
  }

  /* 右侧播放列表 */
  .right {
    flex: 1;
    background: $glass-bg;
    backdrop-filter: blur(12px);
    border: 1px solid $glass-edge;
    border-radius: 40px 40px 30px 30px;
    padding: 20px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
    max-height: 70vh;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    transition: max-height 0.3s;

    &.collapsed {
      max-height: 64px;
    }

    .playlist-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 16px;

      .left-head {
        display: flex;
        gap: 12px;
        align-items: center;

        h3 {
          margin: 0;
          font-size: 1.3rem;
          font-weight: 400;
          color: $accent-aqua;
          letter-spacing: 1px;
        }

        .api-hint {
          color: rgba($text-light, 0.5);
          font-size: 0.9rem;
        }
      }

      .search-wrap {
        display: flex;
        gap: 8px;
        align-items: center;

        input {
          padding: 8px 16px;
          background: rgba(0, 0, 0, 0.3);
          border: 1px solid $glass-edge;
          border-radius: 30px;
          color: $text-light;
          font-size: 0.9rem;
          width: 200px;
          transition: all 0.2s;

          &:focus {
            outline: none;
            border-color: $accent-aqua;
            box-shadow: 0 0 15px rgba($accent-aqua, 0.3);
          }
        }

        .clear {
          background: transparent;
          border: none;
          color: $accent-pink;
          cursor: pointer;
          font-size: 1rem;
          padding: 4px;
        }
      }
    }

    .list-area {
      flex: 1;
      overflow-y: auto;

      &::-webkit-scrollbar {
        width: 6px;
      }
      &::-webkit-scrollbar-thumb {
        background: $accent-aqua;
        border-radius: 6px;
      }
    }

    .list-loading {
      display: flex;
      align-items: center;
      gap: 8px;
      color: $accent-aqua;
      padding: 20px;

      .small-spinner {
        width: 20px;
        height: 20px;
        border: 2px solid rgba($accent-aqua, 0.2);
        border-top-color: $accent-aqua;
        border-radius: 50%;
        animation: spin 1s infinite linear;
      }
    }

    .playlist {
      list-style: none;
      padding: 0;
      margin: 0;

      li {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 12px 16px;
        margin-bottom: 8px;
        border-radius: 40px 40px 30px 30px;
        background: rgba(255, 255, 255, 0.02);
        border: 1px solid transparent;
        transition: all 0.2s;
        cursor: pointer;

        &:hover {
          transform: translateX(-4px);
          border-color: $glass-edge;
          background: rgba($accent-aqua, 0.05);
        }

        &.active {
          border-color: $accent-aqua;
          background: rgba($accent-aqua, 0.1);
          box-shadow: 0 0 20px rgba($accent-aqua, 0.2);
        }

        .left-col {
          display: flex;
          gap: 12px;
          align-items: center;
          min-width: 0;

          .dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: $accent-aqua;
            box-shadow: 0 0 10px $accent-aqua;
            flex-shrink: 0;
          }

          .title {
            font-size: 1rem;
            color: $text-light;
            overflow: hidden;
            text-overflow: ellipsis;
            display: -webkit-box;
            -webkit-line-clamp: 2;
            -webkit-box-orient: vertical;
            word-break: break-word;
          }
        }

        .right-col .len {
          color: rgba($text-light, 0.6);
          font-size: 0.9rem;
          font-family: monospace;
        }
      }
    }
  }



  /* 响应式 */
  @media (max-width: 920px) {
    .stage {
      flex-direction: column;
    }

    .left {
      width: 100%;
    }

    .right {
      width: 100%;
      max-height: 50vh;
    }

 
  }

  @media (max-width: 520px) {
    padding: 60px 12px 40px;

    .cover {
      height: 240px;
    }

    .btns {
      justify-content: center;
    }

    .search-wrap input {
      width: 140px;
    }
  }
}
</style>
