<template>
  <div class="cantarella-gallery">
    <!-- 深海粒子背景（动态浮游光点） -->
    <div class="deep-sea-particles">
      <div class="particle-layer layer1"></div>
      <div class="particle-layer layer2"></div>
      <div class="particle-layer layer3"></div>
      <div class="glow-overlay"></div>
    </div>

    <!-- 头部：深海明珠 + 触须 -->
    <header class="gallery-header">
      <div class="pearl">
        <h1>翡萨烈·幻梦画廊</h1>
        <div class="pearl-count">
          <span>{{ imgTotal }}</span> 幅回响
        </div>
      </div>
    </header>

    <!-- 主内容区：画廊 + 排行榜 -->
    <div class="gallery-layout">
      <!-- 左侧画廊主体 -->
      <section class="gallery-main">
        <!-- 排序与上传控制区 -->
        <div class="gallery-toolbar">
          <button class="sort-btn" @click="toggleSort">
            <span class="btn-icon"></span>
            {{ sortBy === "like_count" ? "点赞排序" : "最新上传" }}
          </button>
          <button class="upload-btn" @click="openUploadModal">
            <span class="btn-icon"></span>
            沉入幻梦·上传
          </button>
        </div>

        <!-- 图片网格 -->
        <div class="gallery-grid">
          <div
            v-for="(img, index) in images"
            :key="img.id"
            class="card"
            @click="openLightbox(index)"
            ref="cards"
          >
            <div class="card-inner">
              <img :src="img.src" :alt="img.alt" loading="lazy" />
              <div class="card-overlay">
                <span class="view-label">窥视幻梦</span>
              </div>
              <button class="like-btn" @click.stop="handleLike(img)">
                <span class="heart" :class="{ liked: img.liked }"></span>
                <span class="like-count">{{ img.likeCount }}</span>
              </button>
              <div class="card-depth"></div>
            </div>
          </div>
        </div>

        <!-- 无限滚动哨兵 -->
        <div ref="sentinel" class="sentinel"></div>

        <!-- 加载状态 -->
        <div class="loading-indicator" v-if="loading">
          <div class="jelly-loader"></div>
          <span>浮出水面……</span>
        </div>
        <div class="finished-indicator" v-if="finished">—— 已至深海之底 ——</div>
      </section>

      <!-- 右侧排行榜面板 -->
      <aside class="ranking-panel" :class="{ collapsed: !expanded }">
        <div class="panel-header" @click="expanded = !expanded">
          <h3 class="ranking-title">贡献榜·幻梦回响</h3>
          <span class="panel-count">{{ rankingList.length }}</span>
          <span class="toggle-icon">{{ expanded ? "▾" : "▸" }}</span>
        </div>
        <transition name="fade">
          <ul v-if="expanded" class="ranking-list">
            <li
              v-for="(item, idx) in rankingList"
              :key="idx"
              class="ranking-item"
              :class="`rank-${idx + 1}`"
            >
              <span class="rank">{{ idx + 1 }}</span>
              <span class="name">{{ item.nickname }}</span>
              <span class="count">{{ item.count }} 幅</span>
            </li>
          </ul>
        </transition>
      </aside>
    </div>

    <!-- Lightbox 幻梦灯箱 -->
    <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
      <span class="close" @click="closeLightbox">✕</span>
      <span class="prev" @click.stop="prevImage">‹</span>
      <img :src="images[currentIndex].src" :alt="images[currentIndex].alt" />
      <span class="next" @click.stop="nextImage">›</span>
      <div class="lightbox-depth"></div>
    </div>

    <!-- 上传弹窗 -->
    <div
      v-if="uploadModalOpen"
      class="upload-modal-overlay"
      @click.self="closeUploadModal"
    >
      <div class="upload-modal">
        <div class="modal-pearl">
          <h3>沉入幻梦·上传</h3>
          <span class="pearl-glow"></span>
        </div>

        <div class="stats">
          今日已上传 <strong>{{ uploadedToday }}</strong> 张， 剩余
          <strong>{{ remaining }}</strong> 张
        </div>

        <div class="tip-container">
          <ul class="tips-list">
            <li>
              审核规则： 1.不要色情倾向（不要露三点，我怕被封）
              2.要我能认出是坎特蕾拉。
            </li>
            <li>
              由于没有用户系统，我这边不好做审核反馈，但只要显示上传成功，我这边肯定能收到。
            </li>
            <li>
              如果图片数量较多请在b站私信联系我给我网盘链接，因为我云服务器比较小一次性上传太多图片可能会导致上传不上，感谢理解。
            </li>
            <li>
              因为审核上传一次比较麻烦，所以审核时间不定，最晚一周，感谢谅解。
            </li>
          </ul>
        </div>

        <label>
          昵称（翡萨烈之名）
          <input v-model="nickname" type="text" placeholder="请输入昵称" />
        </label>

        <label class="file-label">
          选择图片（最多 {{ remaining }} 张）
          <input
            ref="fileInput"
            type="file"
            multiple
            accept="image/*"
            @change="handleFileSelect"
          />
          <span class="file-hint" v-if="selectedFiles.length">
            已选 {{ selectedFiles.length }} 张
          </span>
        </label>

        <div class="modal-actions">
          <button :disabled="!canSubmit || isUploading" @click="submitUpload">
            {{ isUploading ? "熬制秘药中…" : "沉入幻梦" }}
          </button>
          <button class="cancel" @click="closeUploadModal">取消</button>
        </div>

        <div class="modal-tentacles">
          <span></span><span></span><span></span><span></span>
        </div>
      </div>
    </div>

    <!-- 浮动小人 -->
    <div class="floating-chibis">
      <img
        v-for="(pet, i) in chibiList"
        :key="i"
        :src="pet.src"
        :style="{ top: pet.top + 'px', left: pet.left + 'px' }"
        class="chibi-img"
      />
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, computed, nextTick, onBeforeUnmount } from "vue";
import { uploadImages } from "@/api/modules/images"; // 前面封装的上传接口
import { getRankingList } from "@/api/modules/ranking"; // 根据你的实际路径调整
import { gsap } from "gsap"; // ← 本地引入
import { getImagesLikesList, likeImage } from "@/api/modules/imagesLikes";
import { debounce } from "lodash";

const sortBy = ref<"uploaded_at" | "like_count">("like_count");
const order = ref<"asc" | "desc">("desc");
function toggleSort() {
  if (sortBy.value === "uploaded_at") {
    sortBy.value = "like_count";
    order.value = "desc";
  } else {
    sortBy.value = "uploaded_at";
    order.value = "desc";
  }
  pageImage.value = 1;
  images.value = [];
  finished.value = false;
  window.scrollTo(0, 0);
  loadNextPage();
}
// 获取已点赞 ID 数组
function getLikedIds(): number[] {
  const data = localStorage.getItem("likedImageIds");
  return data ? JSON.parse(data) : [];
}

// 保存已点赞 ID 数组
function setLikedIds(ids: number[]) {
  localStorage.setItem("likedImageIds", JSON.stringify(ids));
}

async function handleLike(img: ImageItem) {
  if (img.liked) return; // 已点过就不重复调用

  try {
    await likeImage(img.id); // 调用后端接口
    img.likeCount += 1; // 本地更新点赞数
    img.liked = true; // 标记已点赞

    // 更新 localStorage
    const likedIds = getLikedIds();
    likedIds.push(img.id);
    setLikedIds(likedIds);
  } catch (error) {
    console.error("点赞失败", error);
    alert("点赞失败，请稍后重试");
  }
}

interface ImageItem {
  src: string;
  alt: string;
  likeCount: number;
  id: number;
  liked: Boolean;
}

interface RankingItem {
  id?: number; // 如果接口返回有 id，可加上
  nickname: string;
  count: number;
}
const rankingList = ref<RankingItem[]>([]);
const expanded = ref(true);

// 默认分页参数（如不分页可省略）
const page = 1;
const pageSize = 99;

const fetchRanking = async () => {
  const res = await getRankingList({ page, pageSize, character_key: "kante" });
  if (res.success) {
    rankingList.value = res.data;
  } else {
    console.error("获取排行榜失败", res.message);
  }
};

// 响应式存放最终图片列表
const images = ref<ImageItem[]>([]);

const pageImage = ref(1);
const limit = ref(10);
const loading = ref(false);
const finished = ref(false);

const sentinel = ref<HTMLElement | null>(null);

// 1. 在外层创建一个单例 observerCard
const observerCard = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add("visible");
        observerCard.unobserve(entry.target);
      }
    });
  },
  { threshold: 0.1 }
);
// 2. 每次有新卡片时，都调用这个方法去挂载观察
async function observeNewCards(startIndex = 0) {
  await nextTick();
  const cards = document.querySelectorAll<HTMLElement>(".card");
  for (let i = startIndex; i < cards.length; i++) {
    observerCard.observe(cards[i]);
  }
}
const imgTotal = ref(0);
async function loadNextPage() {
  if (loading.value || finished.value) return;
  loading.value = true;
  try {
    const res = await getImagesLikesList({
      page: pageImage.value,
      limit: limit.value,
      sortBy: sortBy.value,
      character_key: "kante",
      order: order.value,
    });
    imgTotal.value = res.total;
    const likedIds = getLikedIds();
    const list = (
      res.images as Array<{ url: string; like_count: number; id: number }>
    ).map((item) => ({
      src: item.url,
      alt: "",
      likeCount: item.like_count,
      id: item.id, // 如果需要的话，方便点赞用
      liked: likedIds.includes(item.id),
    }));
    if (list.length === 0) {
      finished.value = true;
      return;
    }
    // 记录加载前的长度，方便后面找出“新增”节点
    const oldLength = images.value.length;
    const existingIds = new Set(images.value.map((i) => i.id));
    const filtered = list.filter((item) => !existingIds.has(item.id));
    images.value.push(...filtered);
    pageImage.value++;

    observeNewCards(oldLength);
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}

// 3. 给 loadNextPage 包装一个防抖版
const debouncedLoad = debounce(
  () => {
    loadNextPage();
  },
  200,
  { leading: true, trailing: false }
);

const lightboxOpen = ref(false);
const currentIndex = ref(0);

function openLightbox(index: number) {
  currentIndex.value = index;
  lightboxOpen.value = true;
}
function closeLightbox() {
  lightboxOpen.value = false;
}
function prevImage() {
  currentIndex.value =
    (currentIndex.value + images.value.length - 1) % images.value.length;
}
function nextImage() {
  currentIndex.value = (currentIndex.value + 1) % images.value.length;
}

// 上传弹窗逻辑

const uploadModalOpen = ref(false);
const nickname = ref("");
const fileInput = ref<HTMLInputElement>();
const selectedFiles = ref<File[]>([]);

// 从 localStorage 读取“今天”已上传数量
function getTodayKey() {
  return `uploaded_${new Date().toISOString().slice(0, 10)}`;
}
const uploadedToday = ref<number>(
  Number(localStorage.getItem(getTodayKey()) || 0)
);
const remaining = computed(() => Math.max(27 - uploadedToday.value, 0));

// 控制提交按钮
const canSubmit = computed(() => {
  return (
    nickname.value.trim().length > 0 &&
    selectedFiles.value.length > 0 &&
    selectedFiles.value.length <= remaining.value
  );
});

// 放在 script 顶部，或者 utils 里
function clearOldUploadRecords() {
  const today = new Date();
  const storage = window.localStorage;
  for (const key of Object.keys(storage)) {
    if (!key.startsWith("uploaded_")) continue;

    // key 格式 uploaded_YYYY-MM-DD
    const dateStr = key.slice("uploaded_".length);
    const recordDate = new Date(dateStr);
    if (isNaN(recordDate.getTime())) continue;

    // 计算相差天数
    const diffMs = today.getTime() - recordDate.getTime();
    const diffDays = diffMs / (1000 * 60 * 60 * 24);

    // 如果超过 2 天，就删掉
    if (diffDays > 2) {
      storage.removeItem(key);
    }
  }
}

function openUploadModal() {
  clearOldUploadRecords();
  nickname.value = "";
  selectedFiles.value = [];
  if (fileInput.value) fileInput.value.value = "";
  // 每次打开重新刷新已上传数
  uploadedToday.value = Number(localStorage.getItem(getTodayKey()) || 0);
  uploadModalOpen.value = true;
}
function closeUploadModal() {
  uploadModalOpen.value = false;
}

// 本地截断到剩余数量
function handleFileSelect(e: Event) {
  const files = Array.from((e.target as HTMLInputElement).files || []);

  if (!files) return;

  const validFiles: File[] = [];
  for (const file of files) {
    if (file.size > 20 * 1024 * 1024) {
      alert(`文件太大：${file.name}，请控制在 20MB 内`);
      continue;
    }
    validFiles.push(file);
  }

  if (validFiles.length === 0) return;

  if (validFiles.length > remaining.value) {
    alert(
      `今天最多还能上传 ${remaining.value} 张，已为你截取前 ${remaining.value} 张`
    );
    selectedFiles.value = files.slice(0, remaining.value);
  } else {
    selectedFiles.value = files;
  }
}
const isUploading = ref(false);
async function submitUpload() {
  if (!canSubmit.value) return;
  isUploading.value = true;
  try {
    const res = await uploadImages(
      selectedFiles.value,
      nickname.value.trim(),
      "kante"
    );
    const uploadedCount = res.data.length;
    // 更新 localStorage
    uploadedToday.value += uploadedCount;
    localStorage.setItem(getTodayKey(), String(uploadedToday.value));

    alert(`成功上传 ${uploadedCount} 张图片`);
    closeUploadModal();
    // …可选：刷新画廊列表或把新图片追加到 images …
  } catch (err: any) {
    console.error(err);
    alert(err.message || "上传失败");
  } finally {
    isUploading.value = false;
  }
}

interface Chibi {
  src: string;
  top: number;
  left: number;
}

const chibiList = ref<Chibi[]>([]);
let sentinelObserver: IntersectionObserver;
// Scroll-triggered lazy animation
onMounted(async () => {
  // 1. 拉排行榜
  await fetchRanking();

  // 2. 拉第一页图片并挂载动画 observer
  await loadNextPage(); // 内部会调用 observeNewCards(oldLen)
  // 对首次卡片做一次完整 observe
  observeNewCards(0);

  // 3. 初始化 sentinelObserver，再 observe
  sentinelObserver = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) debouncedLoad();
    },
    { rootMargin: "0px", threshold: 0.1 }
  );
  if (sentinel.value) {
    sentinelObserver.observe(sentinel.value);
  }
  // 1. 基础配置信息
  const total = 9;
  let pickCount = 3; // 每次抽取 3 张
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const isMobile = window.innerWidth <= 768;
  // 如果已知单张小人图片的宽高，可避免超出边界；
  // 假设小人图片宽 100px、高 100px，按需替换：
  const imgWidth = 100;
  const imgHeight = 100;

  // 2. Fisher–Yates 洗牌函数
  function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  // 3. 随机选出 3 个编号
  if (isMobile) {
    pickCount = 1;
  }
  const nums = shuffle(Array.from({ length: total }, (_, k) => k + 1));
  const picks = nums.slice(0, pickCount);

  // 4. 生成随机位置并填充 chibiList
  chibiList.value = []; // 先清空
  picks.forEach((i) => {
    chibiList.value.push({
      src: `/QImages/1 (${i}).png`,
      left: Math.random() * (vw - imgWidth), // 保证不超出左右边界
      top: Math.random() * (vh - imgHeight), // 保证不超出上下边界
    });
  });

  // 2. 等 img 渲染到 DOM
  await nextTick();

  // 3. 给每个小人绑定 GSAP 动画
  const imgs = document.querySelectorAll<HTMLImageElement>(".chibi-img");
  imgs.forEach((img, index) => {
    const padding = 200; // 边缘预留空间
    // ✅ 初始出场动画（闪现）
    gsap.fromTo(
      img,
      { opacity: 0, scale: 0.5 },
      {
        opacity: 1,
        scale: 1,
        duration: 0.8,
        ease: "back.out(2)",
        delay: 0.2 * index,
      }
    );

    // ✅ 鼠标靠近闪避
    img.addEventListener("mouseenter", () => {
      gsap.killTweensOf(img);

      gsap.to(img, {
        x: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        y: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        duration: 1.2,
        ease: "back.out(2)",
        onComplete: () => {
          // 闪避完成后，再重新启用动画
          animate(img);
        },
      });
    });

    const animate = (img: HTMLImageElement) => {
      let { x, y } = img.getBoundingClientRect();
      let deltaX = (Math.random() - 0.5) * 200;
      let deltaY = (Math.random() - 0.5) * 200;

      // 预测一下偏移后的位置
      let nextX = x + deltaX;
      let nextY = y + deltaY;

      // 校正：防漂出左、右、上、下边界
      if (nextX < padding) deltaX = padding - x;
      if (nextX + img.width > window.innerWidth - padding)
        deltaX = window.innerWidth - padding - (x + img.width);
      if (nextY < padding) deltaY = padding - y;
      if (nextY + img.height > window.innerHeight - padding)
        deltaY = window.innerHeight - padding - (y + img.height);

      gsap.to(img, {
        x: `+=${deltaX.toFixed(0)}`,
        y: `+=${deltaY.toFixed(0)}`,
        rotation: `+=${((Math.random() - 0.5) * 60).toFixed(0)}`,
        duration: 2 + Math.random() * 2,
        ease: "power1.inOut",
        onComplete: () => animate(img),
      });
    };
    animate(img);
  });
});

onBeforeUnmount(() => {
  observerCard.disconnect();
  sentinelObserver.disconnect();
  // 以及你在 onMounted 里新建的其它 Observer
});
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

.cantarella-gallery {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(180deg, $deep-bg 0%, $mid-bg 100%);
  color: $text-light;
  overflow-x: hidden;
  padding-top: 60px;
  /* 深海粒子背景（取代轮播） */
  .deep-sea-particles {
    position: fixed;
    inset: 0;
    z-index: 0;
    pointer-events: none;
    overflow: hidden;

    .particle-layer {
      position: absolute;
      inset: 0;
      background-repeat: repeat;
      background-size: 600px 600px;
      mix-blend-mode: screen;
      opacity: 0.3;
      animation: particleFloat 60s infinite linear;
    }

    .layer1 {
      background-image: radial-gradient(
        circle at 30% 40%,
        rgba($accent-aqua, 0.15) 2px,
        transparent 3px
      );
      animation-duration: 80s;
      opacity: 0.25;
    }
    .layer2 {
      background-image: radial-gradient(
        circle at 70% 60%,
        rgba($accent-lavender, 0.12) 3px,
        transparent 4px
      );
      background-size: 800px 800px;
      animation-duration: 120s;
      animation-direction: reverse;
      opacity: 0.2;
    }
    .layer3 {
      background-image: radial-gradient(
        circle at 20% 80%,
        rgba($accent-pink, 0.08) 4px,
        transparent 5px
      );
      background-size: 1000px 1000px;
      animation-duration: 100s;
      opacity: 0.15;
    }

    .glow-overlay {
      position: absolute;
      inset: 0;
      background: radial-gradient(
          circle at 30% 30%,
          rgba($accent-aqua, 0.05),
          transparent 60%
        ),
        radial-gradient(
          circle at 80% 70%,
          rgba($accent-lavender, 0.05),
          transparent 70%
        );
      animation: glowPulse 10s ease-in-out infinite alternate;
    }
  }

  /* 头部深海明珠 */
  .gallery-header {
    position: relative;
    z-index: 10;
    max-width: 1200px;
    margin: 0 auto 20px;
    padding: 30px 20px 0;
    text-align: center;

    .pearl {
      display: inline-block;
      background: rgba(255, 255, 255, 0.02);
      backdrop-filter: blur(8px);
      border: 1px solid $glass-edge;
      border-radius: 120px;
      padding: 16px 48px;
      box-shadow: 0 0 50px rgba($accent-aqua, 0.3),
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
  }

  /* 双栏布局 */
  .gallery-layout {
    position: relative;
    z-index: 5;
    display: flex;
    gap: 24px;
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
  }

  /* 左侧画廊主体 */
  .gallery-main {
    flex: 1;
    min-width: 0;
  }

  /* 工具栏 */
  .gallery-toolbar {
    display: flex;
    gap: 16px;
    margin-bottom: 24px;
    justify-content: flex-start;
    flex-wrap: wrap;

    button {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      padding: 10px 24px 10px 20px;
      background: rgba(6, 10, 20, 0.6);
      backdrop-filter: blur(8px);
      border: 1px solid $glass-edge;
      border-radius: 40px;
      color: $text-light;
      font-size: 0.95rem;
      cursor: pointer;
      transition: all 0.3s;
      box-shadow: 0 0 20px rgba($accent-aqua, 0.1);

      .btn-icon {
        width: 18px;
        height: 18px;
        background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
        border-radius: 50%;
        box-shadow: 0 0 10px $accent-aqua;
      }

      &:hover {
        transform: translateY(-2px);
        border-color: rgba($accent-aqua, 0.5);
        box-shadow: 0 0 30px rgba($accent-aqua, 0.3);
      }
    }

    .upload-btn {
      background: linear-gradient(
        135deg,
        rgba($accent-lavender, 0.2),
        rgba($accent-aqua, 0.2)
      );
      border-color: rgba($accent-pink, 0.3);
    }
  }

  /* 图片网格 */
  .gallery-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
    gap: 20px;
  }

  /* 卡片 - 修复图片模糊问题 */
  .card {
    opacity: 0;
    transform: translateY(20px);
    transition: opacity 0.6s ease, transform 0.6s ease;

    &.visible {
      opacity: 1;
      transform: translateY(0);
    }

    .card-inner {
      position: relative;
      border-radius: 40px 40px 30px 30px;
      overflow: hidden;
      background: $card-bg;
      backdrop-filter: blur(8px);
      border: 1px solid $glass-edge;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
      transition: all 0.4s ease;

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
        z-index: 1;
      }

      &:hover {
        transform: translateY(-6px) scale(1.02);
        border-color: rgba($accent-aqua, 0.4);
        box-shadow: 0 30px 60px rgba($accent-aqua, 0.2);
      }

      img {
        width: 100%;
        height: 240px;
        object-fit: cover;
        display: block;
        opacity: 1;
        transition: opacity 0.6s ease;
      }

      .card-overlay {
        position: absolute;
        inset: 0;
        background: linear-gradient(to top, rgba($deep-bg, 0.8), transparent);
        display: flex;
        align-items: flex-end;
        justify-content: center;
        padding-bottom: 20px;
        opacity: 0;
        transition: opacity 0.4s;
        z-index: 2;

        .view-label {
          background: rgba(0, 0, 0, 0.5);
          backdrop-filter: blur(4px);
          padding: 6px 16px;
          border-radius: 40px;
          border: 1px solid rgba($accent-aqua, 0.3);
          color: $accent-aqua;
          font-size: 0.9rem;
        }
      }

      &:hover .card-overlay {
        opacity: 1;
      }

      .like-btn {
        position: absolute;
        bottom: 12px;
        right: 12px;
        display: flex;
        align-items: center;
        gap: 6px;
        background: rgba(0, 0, 0, 0.5);
        backdrop-filter: blur(4px);
        border: 1px solid rgba($accent-aqua, 0.3);
        border-radius: 30px;
        padding: 6px 12px;
        cursor: pointer;
        z-index: 3;
        transition: all 0.2s;

        &:hover {
          transform: scale(1.1);
          border-color: $accent-aqua;
        }

        .heart {
          width: 20px;
          height: 20px;
          background: url("/icons/heart-red-outline.svg") no-repeat center;
          background-size: contain;
          filter: drop-shadow(0 0 6px $accent-aqua);
          transition: all 0.3s;

          &.liked {
            background: url("/icons/heart-red-filled.svg") no-repeat center;
            background-size: contain;
            filter: drop-shadow(0 0 10px $accent-pink);
          }
        }

        .like-count {
          color: $text-light;
          font-size: 0.9rem;
          font-weight: 600;
        }
      }

      .card-depth {
        height: 20px;
        background: linear-gradient(
          to bottom,
          transparent,
          rgba($accent-deep-blue, 0.2)
        );
        margin: 0;
        position: absolute;
        bottom: 0;
        left: 0;
        right: 0;
        pointer-events: none;
        z-index: 1;
      }
    }
  }

  /* 无限滚动相关 */
  .sentinel {
    height: 1px;
    opacity: 0;
  }

  .loading-indicator {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 12px;
    padding: 30px;
    color: rgba($accent-aqua, 0.7);

    .jelly-loader {
      width: 24px;
      height: 24px;
      border: 2px solid rgba($accent-aqua, 0.2);
      border-top-color: $accent-aqua;
      border-radius: 50%;
      animation: spin 1s infinite linear;
    }
  }

  .finished-indicator {
    text-align: center;
    padding: 30px;
    color: rgba($text-light, 0.2);
    font-style: italic;
  }

  /* 右侧排行榜面板 */
  .ranking-panel {
    width: 260px;
    background: rgba(6, 10, 20, 0.7);
    backdrop-filter: blur(12px);
    border: 1px solid $glass-edge;
    border-radius: 40px 40px 30px 30px;
    padding: 20px 16px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
    align-self: start;
    position: sticky;
    top: 100px;
    transition: all 0.3s;

    &.collapsed {
      padding-bottom: 12px;
    }

    .panel-header {
      display: flex;
      align-items: center;
      gap: 8px;
      cursor: pointer;
      padding-bottom: 12px;
      border-bottom: 1px solid rgba($accent-aqua, 0.2);

      .ranking-title {
        margin: 0;
        font-size: 1.1rem;
        font-weight: 400;
        color: $accent-aqua;
        letter-spacing: 1px;
        flex: 1;
      }

      .panel-count {
        background: rgba($accent-deep-blue, 0.5);
        padding: 2px 8px;
        border-radius: 30px;
        font-size: 0.8rem;
      }

      .toggle-icon {
        font-size: 1.2rem;
        color: $accent-aqua;
      }
    }

    .ranking-list {
      list-style: none;
      padding: 12px 0 0;
      margin: 0;
      max-height: 400px;
      overflow-y: auto;

      &::-webkit-scrollbar {
        width: 4px;
      }
      &::-webkit-scrollbar-thumb {
        background: $accent-aqua;
        border-radius: 4px;
      }
    }

    .ranking-item {
      display: flex;
      align-items: center;
      gap: 8px;
      padding: 10px 12px;
      margin-bottom: 6px;
      border-radius: 30px;
      transition: all 0.2s;

      &:hover {
        background: rgba($accent-aqua, 0.1);
        transform: translateX(-4px);
      }

      .rank {
        width: 28px;
        height: 28px;
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 50%;
        background: rgba($accent-deep-blue, 0.5);
        font-weight: 600;
        font-size: 0.9rem;
      }

      .name {
        flex: 1;
        font-size: 0.95rem;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
      }

      .count {
        color: $accent-aqua;
        font-weight: 600;
        font-size: 0.9rem;
      }

      &.rank-1 .rank {
        background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
        color: $deep-bg;
      }
      &.rank-2 .rank {
        background: $accent-lavender;
      }
      &.rank-3 .rank {
        background: $accent-deep-blue;
      }
    }
  }

  /* Lightbox 幻梦灯箱 */
  .lightbox {
    position: fixed;
    inset: 0;
    background: rgba(2, 6, 10, 0.95);
    backdrop-filter: blur(16px);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;

    img {
      max-width: 85%;
      max-height: 85%;
      border-radius: 24px;
      border: 2px solid rgba($accent-aqua, 0.3);
      box-shadow: 0 0 60px rgba($accent-aqua, 0.3);
    }

    .close,
    .prev,
    .next {
      position: absolute;
      color: $text-light;
      font-size: 3rem;
      cursor: pointer;
      padding: 16px;
      text-shadow: 0 0 20px $accent-aqua;
      transition: all 0.2s;
      &:hover {
        color: $accent-aqua;
        transform: scale(1.2);
      }
    }
    .close {
      top: 20px;
      right: 20px;
    }
    .prev {
      left: 20px;
      top: 50%;
      transform: translateY(-50%);
    }
    .next {
      right: 20px;
      top: 50%;
      transform: translateY(-50%);
    }

    .lightbox-depth {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      height: 100px;
      background: linear-gradient(
        to top,
        rgba($accent-deep-blue, 0.3),
        transparent
      );
      pointer-events: none;
    }
  }

  /* 上传弹窗 */
  .upload-modal-overlay {
    position: fixed;
    inset: 0;
    z-index: 2000;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(2, 6, 10, 0.8);
    backdrop-filter: blur(12px);
  }

  .upload-modal {
    width: 500px;
    max-width: calc(100% - 40px);
    background: rgba(6, 10, 20, 0.9);
    backdrop-filter: blur(16px);
    border: 1px solid $glass-edge;
    border-radius: 60px 60px 40px 40px;
    padding: 30px;
    position: relative;
    box-shadow: 0 0 80px rgba($accent-aqua, 0.2);

    .modal-pearl {
      text-align: center;
      margin-bottom: 20px;
      h3 {
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
        font-family: "Times New Roman", serif;
      }
      .pearl-glow {
        display: block;
        width: 100px;
        height: 4px;
        background: linear-gradient(
          90deg,
          transparent,
          $accent-aqua,
          transparent
        );
        margin: 8px auto 0;
        border-radius: 2px;
      }
    }

    .stats {
      text-align: center;
      margin-bottom: 20px;
      font-size: 1rem;
      strong {
        color: $accent-aqua;
      }
    }

    .tip-container {
      background: rgba(0, 0, 0, 0.3);
      border-left: 4px solid $accent-aqua;
      border-radius: 16px;
      padding: 16px;
      margin-bottom: 20px;

      .tips-list {
        list-style: none;
        padding: 0;
        margin: 0;
        li {
          position: relative;
          padding-left: 24px;
          margin-bottom: 8px;
          font-size: 0.9rem;
          color: rgba($text-light, 0.8);
          &::before {
            content: "";
            position: absolute;
            left: 4px;
            top: 8px;
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: $accent-aqua;
            box-shadow: 0 0 10px $accent-aqua;
          }
        }
      }
    }

    label {
      display: block;
      margin-bottom: 20px;
      color: rgba($text-light, 0.9);
      font-size: 0.95rem;

      input,
      .file-hint {
        width: 100%;
        margin-top: 8px;
        padding: 12px 16px;
        background: rgba(0, 0, 0, 0.3);
        border: 1px solid rgba($accent-aqua, 0.2);
        border-radius: 40px;
        color: $text-light;
        font-size: 1rem;
        transition: all 0.2s;

        &:focus {
          outline: none;
          border-color: $accent-aqua;
          box-shadow: 0 0 20px rgba($accent-aqua, 0.3);
        }
      }
      .file-hint {
        display: inline-block;
        width: auto;
        margin-top: 8px;
        background: rgba($accent-deep-blue, 0.3);
      }
    }

    .file-label {
      input[type="file"] {
        padding: 8px;
      }
    }

    .modal-actions {
      display: flex;
      gap: 16px;
      justify-content: flex-end;
      margin-top: 30px;

      button {
        padding: 12px 32px;
        border: none;
        border-radius: 40px;
        font-weight: 600;
        cursor: pointer;
        transition: all 0.2s;
        font-size: 1rem;

        &:not(.cancel) {
          background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
          color: $deep-bg;
          box-shadow: 0 0 20px rgba($accent-aqua, 0.4);

          &:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 0 30px rgba($accent-aqua, 0.7);
          }
          &:disabled {
            opacity: 0.4;
            cursor: not-allowed;
          }
        }

        &.cancel {
          background: transparent;
          border: 1px solid rgba($accent-aqua, 0.3);
          color: $text-light;
          &:hover {
            background: rgba($accent-aqua, 0.1);
          }
        }
      }
    }

    .modal-tentacles {
      display: flex;
      justify-content: center;
      gap: 20px;
      margin-top: 20px;
      span {
        width: 4px;
        height: 30px;
        background: linear-gradient(to top, $accent-aqua, transparent);
        border-radius: 2px;
        animation: tentacleWave 3s infinite alternate;
        &:nth-child(1) {
          height: 40px;
          animation-delay: 0s;
        }
        &:nth-child(2) {
          height: 25px;
          animation-delay: 0.7s;
          background: $accent-pink;
        }
        &:nth-child(3) {
          height: 50px;
          animation-delay: 0.3s;
        }
        &:nth-child(4) {
          height: 35px;
          animation-delay: 1.1s;
        }
      }
    }
  }

  /* 浮动小人 */
  .floating-chibis {
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 99;
    .chibi-img {
      position: absolute;
      width: 80px;
      pointer-events: auto;
      filter: drop-shadow(0 0 15px rgba($accent-aqua, 0.5));
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
  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  @keyframes particleFloat {
    0% {
      background-position: 0 0;
    }
    100% {
      background-position: 600px 600px;
    }
  }
  @keyframes glowPulse {
    0% {
      opacity: 0.3;
    }
    100% {
      opacity: 0.7;
    }
  }

  /* 移动端适配 */
  @media (max-width: 720px) {
    .gallery-header .pearl {
      padding: 12px 24px;
      h1 {
        font-size: 1.5rem;
      }
    }

    .gallery-layout {
      flex-direction: column;
      padding: 0 12px;
    }

    .ranking-panel {
      width: 100%;
      position: static;
      margin-top: 20px;
    }

    .gallery-toolbar {
      justify-content: center;
    }

    .gallery-grid {
      grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
      gap: 12px;
    }

    .upload-modal {
      padding: 20px;
      .modal-actions {
        flex-direction: column;
        button {
          width: 100%;
        }
      }
    }
  }
}
</style>
