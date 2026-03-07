<template>
  <div class="cantarella-wiki">
    <!-- 背景轮播（两组用于桌面/移动） -->
    <div class="carousel carousel-desktop" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
        alt=""
      />
    </div>
    <div class="carousel carousel-mobile" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
        alt=""
      />
    </div>

    <!-- 头部：深海明珠 -->
    <header class="wiki-header">
      <div class="pearl">
        <h1>翡萨烈·渊海手札</h1>
        <p class="subtitle">于幽海深处，打捞那些未曾沉没的文字</p>
      </div>
      <div class="header-actions">
        <input
          v-model="search"
          class="search-input"
          placeholder="搜索标题或标签..."
        />
        <button class="btn-create" @click="openCreate">沉入幻梦·新建</button>
      </div>
      <div class="header-decoration">
        <span></span><span></span><span></span>
      </div>
    </header>

    <main class="wiki-body">
      <div v-if="filteredEntries.length === 0" class="empty">
        深海静谧，尚无回响……
      </div>

      <ul class="entry-list">
        <li v-for="entry in filteredEntries" :key="entry.id" class="entry-card">
          <div class="card-inner">
            <div class="card-head" @click="openDetail(entry)">
              <div class="title-badge">
                <h2 class="entry-title">{{ entry.title }}</h2>
                <span class="entry-badge">#{{ entry.slug || "未标记" }}</span>
              </div>
              <div class="entry-meta">
                <span class="meta-item">作者：{{ entry.author }}</span>
                <span class="meta-item"
                  >时间：{{ formatTime(entry.updatedAt) }}</span
                >
              </div>
            </div>

            <div class="card-actions">
              <button
                class="like-btn"
                :class="{ active: isLiked(entry.id) }"
                :aria-pressed="isLiked(entry.id)"
                @click.stop="toggleLike(entry.id)"
              >
                <span class="heart"></span>
                <span class="like-count">{{ entry.likes }}</span>
              </button>
              <div class="action-group" v-if="canEdit(entry.id)">
                <button class="action-btn" @click="openEdit(entry)">
                  编辑
                </button>
                <button class="action-btn danger" @click="remove(entry.id)">
                  删除
                </button>
              </div>
            </div>
            <div class="card-depth"></div>
          </div>
        </li>
      </ul>
    </main>

    <!-- Edit/Create Modal -->
    <transition name="fade-zoom">
      <div class="modal-overlay" v-if="showModal">
        <div class="modal">
          <div class="modal-pearl">
            <h3>{{ editing ? "编辑词条" : "新建词条" }}</h3>
            <span class="pearl-glow"></span>
          </div>
          <button class="close-btn" @click="closeModal">✕</button>
          <div class="modal-body">
            <label>
              标题
              <input v-model="form.title" placeholder="输入标题" />
            </label>

            <label>
              词条（短标签）
              <input
                v-model="form.slug"
                placeholder="比如：彩蛋、考据、点位等等"
              />
            </label>

            <label>
              作者
              <input v-model="form.author" placeholder="作者昵称" />
            </label>

            <label>
              内容
              <textarea
                v-model="form.content"
                rows="8"
                placeholder="在这里输入词条内容"
              ></textarea>
            </label>
          </div>
          <div class="modal-actions">
            <button class="btn ghost" @click="closeModal">取消</button>
            <button class="btn primary" @click="submit">
              {{ editing ? "保存" : "创建" }}
            </button>
          </div>
          <div class="modal-decoration">
            <span></span><span></span><span></span>
          </div>
        </div>
      </div>
    </transition>

    <!-- Detail Modal -->
    <transition name="fade-zoom">
      <div class="modal-overlay" v-if="detailEntry">
        <div class="modal detail-modal">
          <div class="modal-pearl">
            <h3>{{ detailEntry.title }}</h3>
            <span class="pearl-glow"></span>
          </div>
          <button class="close-btn" @click="detailEntry = null">✕</button>
          <div class="modal-body">
            <div class="detail-content">{{ detailEntry.content }}</div>
          </div>
          <div class="modal-decoration">
            <span></span><span></span><span></span>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
// 脚本部分完全保留原逻辑，不做任何改动
import { ref, reactive, computed, onMounted, onUnmounted } from "vue";
import { ElMessage } from "element-plus";
import {
  getWikiList,
  createWiki,
  updateWiki,
  deleteWiki,
  likeWiki,
} from "@/api/modules/wiki";

// 本地存储自己创建的词条 ID
const LS_MY_WIKI_IDS = "yuzuriha:wiki:my_ids";
const myWikiIds: string[] = JSON.parse(
  localStorage.getItem(LS_MY_WIKI_IDS) || "[]"
);
const markAsMine = (id: string | number) => {
  if (!myWikiIds.includes(String(id))) {
    myWikiIds.push(String(id));
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
  }
};
const canEdit = (id: string | number) => myWikiIds.includes(String(id));

// 数据状态
const entries = ref<any[]>([]);

// 本地存储键
const LS_LIKED_IDS = "yuzuriha:wiki:liked_ids";
// 从 localStorage 读取已点赞 id 列表（字符串形式）
const likedIds = ref<string[]>(
  JSON.parse(localStorage.getItem(LS_LIKED_IDS) || "[]")
);

const showModal = ref(false);
const editing = ref(false);
const editingId = ref<string | number | null>(null);
const detailEntry = ref<any>(null);
const form = reactive({ title: "", slug: "", author: "", content: "" });
const search = ref("");

// 时间格式化
function formatTime(ts: string | number | null | undefined) {
  if (!ts) return "未知时间";
  const date = new Date(ts);
  if (isNaN(date.getTime())) return "未知时间";
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(
    2,
    "0"
  )}-${String(date.getDate()).padStart(2, "0")}`;
}

// 加载词条列表
async function loadEntries() {
  try {
    const res: any = await getWikiList();
    entries.value = res.data.map((e: any) => ({
      ...e,
      createdAt: e.createdAt || e.created_at,
      updatedAt: e.updatedAt || e.updated_at,
    }));
  } catch (err) {
    console.error(err);
    ElMessage.error("加载词条失败");
  }
}

// 打开/关闭弹窗
function openCreate() {
  editing.value = false;
  editingId.value = null;
  form.title = "";
  form.slug = "";

  form.content = "";
  showModal.value = true;
}
function openEdit(entry: any) {
  if (!canEdit(entry.id)) {
    ElMessage.warning("只有创建者可以编辑");
    return;
  }
  editing.value = true;
  editingId.value = entry.id;
  form.title = entry.title;
  form.slug = entry.slug;
  form.author = entry.author;
  form.content = entry.content;
  showModal.value = true;
}
function closeModal() {
  showModal.value = false;
}

const canSubmit = computed(() => form.title.trim() && form.content.trim());

// 提交
async function submit() {
  if (!canSubmit.value) {
    ElMessage.warning("请填写标题和内容");
    return;
  }
  const payload = {
    title: form.title.trim(),
    author: form.author.trim() || "匿名",
    content: form.content.trim(),
    slug: null,
  };
  if (form.slug.trim()) payload.slug = form.slug.trim();
  try {
    if (editing.value && editingId.value) {
      await updateWiki(editingId.value, payload);
      ElMessage.success("编辑成功");
    } else {
      const res: any = await createWiki(payload);
      markAsMine(res.data.id);
      editingId.value = res.data.id;
      ElMessage.success("创建成功");
    }
    showModal.value = false;
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("提交失败");
  }
}

// 删除
async function remove(id: string | number) {
  if (!canEdit(id)) {
    ElMessage.warning("只有创建者可以删除");
    return;
  }
  if (!confirm("确认删除该词条？此操作不可撤销")) return;
  try {
    await deleteWiki(id);
    const index = myWikiIds.indexOf(String(id));
    if (index !== -1) myWikiIds.splice(index, 1);
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
    ElMessage.success("删除成功");
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("删除失败");
  }
}

// 点赞
function persistLikedIds() {
  try {
    localStorage.setItem(LS_LIKED_IDS, JSON.stringify(likedIds.value));
  } catch (e) {
    console.warn("保存 likedIds 失败", e);
  }
}

// 判断是否已点赞（供模板绑定 class/aria-pressed）
function isLiked(id: string | number) {
  return likedIds.value.includes(String(id));
}

// 点赞 / 取消点赞（乐观更新，本地仅存 id，点赞数使用 entry.likes）
async function toggleLike(id: string | number) {
  const entry = entries.value.find((e) => e.id === id);
  if (!entry) return;

  const idStr = String(id);
  const wasLiked = likedIds.value.includes(idStr);

  // 乐观更新 UI（立即反映）
  if (wasLiked) {
    // 取消点赞：保证不低于 0
    entry.likes = Math.max(0, (entry.likes || 0) - 1);
    likedIds.value = likedIds.value.filter((x) => x !== idStr);
  } else {
    // 点赞
    entry.likes = (entry.likes || 0) + 1;
    likedIds.value.push(idStr);
  }
  persistLikedIds();

  try {
    // 调用后端（action: 'like' | 'unlike' | 'toggle'）
    // 我们明确传 'like' 或 'unlike'
    const action = wasLiked ? "unlike" : "like";
    await likeWiki(id, action);

    // 可选：如果后端在响应中返回了最新的 likes 数（res.data.likes），
    // 你可以在这里用后端值覆盖本地（示例注释）
    // const res = await likeWiki(id, action)
    // if (res?.data?.likes !== undefined) entry.likes = res.data.likes
  } catch (err) {
    // 出错则回滚乐观更新
    console.error("toggleLike error", err);
    if (wasLiked) {
      // 取消点赞失败 -> 重新标记为已点赞
      entry.likes = (entry.likes || 0) + 1;
      if (!likedIds.value.includes(idStr)) likedIds.value.push(idStr);
    } else {
      // 点赞失败 -> 取消之前增加的 count
      entry.likes = Math.max(0, (entry.likes || 0) - 1);
      likedIds.value = likedIds.value.filter((x) => x !== idStr);
    }
    persistLikedIds();
    ElMessage.error("点赞失败，请稍后重试");
  }
}

// 详情弹窗
async function openDetail(entry: any) {
  detailEntry.value = entry;
}

// 搜索过滤
const filteredEntries = computed(() => {
  const q = String(search.value || "")
    .trim()
    .toLowerCase();
  let list = entries.value;

  // 搜索过滤
  if (q) {
    list = list.filter(
      (e) =>
        (e.title || "").toLowerCase().includes(q) ||
        (e.slug || "").toLowerCase().includes(q)
    );
  }

  // 按点赞数排序（默认降序：点赞多的在前）
  return [...list].sort((a, b) => (b.likes || 0) - (a.likes || 0));
});

// 1. 分别导入两套图
const pcModules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const mobileModules = import.meta.glob(
  "@/assets/images2/*.{jpg,png,jpeg,webp}",
  { eager: true }
);

const pcSrcs: string[] = Object.values(pcModules).map((m: any) => m.default);
const mobileSrcs: string[] = Object.values(mobileModules).map(
  (m: any) => m.default
);

// 洗牌函数
function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

const randomFive = ref<string[]>([]);
const currentIndex = ref(0);
let timer: number;

function pickImages() {
  const isMobile = window.innerWidth < 768;
  const all = isMobile ? mobileSrcs : pcSrcs;
  randomFive.value = shuffle(all).slice(0, 5);
}

onMounted(() => {
  loadEntries();
  pickImages(); // 首次判断
  // 监听窗口大小变化
  window.addEventListener("resize", pickImages);

  // 轮播
  timer = window.setInterval(() => {
    if (randomFive.value.length > 0) {
      currentIndex.value = (currentIndex.value + 1) % randomFive.value.length;
    }
  }, 5000);
});

onUnmounted(() => {
  clearInterval(timer);
  window.removeEventListener("resize", pickImages);
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
$glass-bg: rgba(6, 10, 20, 0.5);

.cantarella-wiki {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(180deg, $deep-bg 0%, $mid-bg 100%);
  color: $text-light;
  overflow-x: hidden;
  padding: 80px 20px 60px;
  font-family: "Noto Sans SC", "Inter", system-ui, sans-serif;

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

  /* 头部 */
  .wiki-header {
    position: relative;
    z-index: 10;
    max-width: 960px;
    margin: 0 auto 30px;
    text-align: center;

    .pearl {
      display: inline-block;
      background: $glass-bg;
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

    .header-actions {
      display: flex;
      gap: 16px;
      justify-content: center;
      margin-top: 20px;
      flex-wrap: wrap;

      .search-input {
        padding: 12px 24px;
        background: rgba(0, 0, 0, 0.3);
        border: 1px solid $glass-edge;
        border-radius: 60px;
        color: $text-light;
        font-size: 1rem;
        min-width: 280px;
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

      .btn-create {
        padding: 12px 32px;
        background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
        border: none;
        border-radius: 60px;
        color: $deep-bg;
        font-weight: 600;
        cursor: pointer;
        box-shadow: 0 0 20px rgba($accent-aqua, 0.4);
        transition: all 0.2s;
        letter-spacing: 1px;

        &:hover {
          transform: translateY(-2px);
          box-shadow: 0 0 30px rgba($accent-aqua, 0.7);
        }
      }
    }

    .header-decoration {
      display: flex;
      justify-content: center;
      gap: 16px;
      margin-top: 16px;
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

  /* 主体 */
  .wiki-body {
    position: relative;
    z-index: 5;
    max-width: 960px;
    margin: 0 auto;

    .empty {
      text-align: center;
      padding: 60px 20px;
      color: rgba($text-light, 0.3);
      font-style: italic;
      font-size: 1.2rem;
    }

    .entry-list {
      list-style: none;
      padding: 0;
      margin: 0;
      display: grid;
      gap: 20px;
    }
  }

  /* 卡片 */
  .entry-card {
    .card-inner {
      background: $glass-bg;
      backdrop-filter: blur(2px);
      border: 1px solid $glass-edge;
      border-radius: 40px 40px 30px 30px;
      padding: 20px 24px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
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
        filter: blur(2px);
        pointer-events: none;
      }

      &:hover {
        transform: translateY(-4px);
        border-color: rgba($accent-aqua, 0.4);
        box-shadow: 0 30px 60px rgba($accent-aqua, 0.2);
      }
    }

    .card-head {
      cursor: pointer;
      margin-bottom: 16px;

      .title-badge {
        display: flex;
        align-items: center;
        gap: 12px;
        margin-bottom: 8px;

        .entry-title {
          margin: 0;
          font-size: 1.3rem;
          font-weight: 500;
          color: $accent-lavender;
          letter-spacing: 1px;
        }

        .entry-badge {
          padding: 4px 12px;
          background: rgba($accent-deep-blue, 0.5);
          border: 1px solid $glass-edge;
          border-radius: 60px;
          color: $accent-aqua;
          font-size: 0.8rem;
          font-weight: 400;
        }
      }

      .entry-meta {
        display: flex;
        gap: 16px;
        color: rgba($text-light, 0.7);
        font-size: 0.9rem;
        flex-wrap: wrap;

        .meta-item {
          display: flex;
          align-items: center;
          gap: 4px;
        }
      }
    }

    .card-actions {
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 16px;

      .like-btn {
        background: transparent;
        border: 1px solid $glass-edge;
        border-radius: 40px;
        padding: 6px 16px;
        display: flex;
        align-items: center;
        gap: 8px;
        cursor: pointer;
        transition: all 0.2s;

        .heart {
          width: 20px;
          height: 20px;
          background: url("/icons/heart-red-outline.svg") no-repeat center;
          background-size: contain;
          filter: drop-shadow(0 0 6px $accent-aqua);
        }

        &.active .heart {
          background: url("/icons/heart-red-filled.svg") no-repeat center;
          background-size: contain;
          filter: drop-shadow(0 0 10px $accent-pink);
        }

        .like-count {
          color: $text-light;
          font-size: 0.9rem;
          min-width: 20px;
          text-align: center;
        }

        &:hover {
          border-color: $accent-aqua;
          background: rgba($accent-aqua, 0.1);
        }
      }

      .action-group {
        display: flex;
        gap: 8px;

        .action-btn {
          padding: 6px 16px;
          background: transparent;
          border: 1px solid $glass-edge;
          border-radius: 40px;
          color: $text-light;
          font-size: 0.9rem;
          cursor: pointer;
          transition: all 0.2s;

          &:hover {
            border-color: $accent-aqua;
            background: rgba($accent-aqua, 0.1);
          }

          &.danger {
            border-color: rgba($accent-pink, 0.3);
            color: $accent-pink;
            &:hover {
              border-color: $accent-pink;
              background: rgba($accent-pink, 0.1);
            }
          }
        }
      }
    }

    .card-depth {
      height: 16px;
      background: linear-gradient(
        to bottom,
        transparent,
        rgba($accent-deep-blue, 0.2)
      );
      margin: 16px -24px -20px -24px;
      border-radius: 0 0 30px 30px;
    }
  }

  /* 模态框 */
  .modal-overlay {
    position: fixed;
    inset: 0;
    z-index: 2000;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(2, 6, 10, 0.8);
   
  }

  .modal {
    width: 600px;
    max-width: calc(100% - 40px);
    max-height: 90vh;
    overflow-y: auto;
    background: rgba(6, 10, 20, 0.9);
    backdrop-filter: blur(16px);
    border: 1px solid $glass-edge;
    border-radius: 60px 60px 40px 40px;
    padding: 30px;
    position: relative;
    box-shadow: 0 0 80px rgba($accent-aqua, 0.2);

    &::-webkit-scrollbar {
      width: 6px;
    }
    &::-webkit-scrollbar-thumb {
      background: $accent-aqua;
      border-radius: 6px;
    }

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

    .close-btn {
      position: absolute;
      top: 20px;
      right: 20px;
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: transparent;
      border: 1px solid $glass-edge;
      color: $text-light;
      font-size: 1.2rem;
      cursor: pointer;
      transition: all 0.2s;

      &:hover {
        border-color: $accent-aqua;
        background: rgba($accent-aqua, 0.1);
      }
    }

    .modal-body {
      display: flex;
      flex-direction: column;
      gap: 20px;
      margin: 20px 0;

      label {
        display: flex;
        flex-direction: column;
        gap: 6px;
        color: rgba($text-light, 0.9);
        font-size: 0.95rem;

        input,
        textarea {
          padding: 12px 18px;
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

          &::placeholder {
            color: rgba($text-light, 0.3);
            font-style: italic;
          }
        }

        textarea {
          border-radius: 30px;
          resize: vertical;
        }
      }

      .detail-content {
        white-space: pre-wrap;
        line-height: 1.8;
        color: $text-light;
        font-size: 1rem;
        padding: 0 10px;
      }
    }

    .modal-actions {
      display: flex;
      justify-content: flex-end;
      gap: 16px;
      margin-top: 30px;

      .btn {
        padding: 12px 32px;
        border: none;
        border-radius: 40px;
        font-weight: 600;
        cursor: pointer;
        transition: all 0.2s;
        font-size: 1rem;

        &.primary {
          background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
          color: $deep-bg;
          box-shadow: 0 0 20px rgba($accent-aqua, 0.4);

          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 0 30px rgba($accent-aqua, 0.7);
          }
        }

        &.ghost {
          background: transparent;
          border: 1px solid $glass-edge;
          color: $text-light;

          &:hover {
            border-color: $accent-aqua;
            background: rgba($accent-aqua, 0.1);
          }
        }
      }
    }

    .modal-decoration {
      display: flex;
      justify-content: center;
      gap: 16px;
      margin-top: 20px;

      span {
        width: 2px;
        height: 20px;
        background: linear-gradient(to top, $accent-aqua, transparent);
        border-radius: 1px;
        animation: glowPulse 3s infinite alternate;

        &:nth-child(1) {
          height: 30px;
          animation-delay: 0s;
        }
        &:nth-child(2) {
          height: 20px;
          animation-delay: 0.5s;
          background: $accent-pink;
        }
        &:nth-child(3) {
          height: 25px;
          animation-delay: 0.2s;
        }
      }
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

  .fade-zoom-enter-active,
  .fade-zoom-leave-active {
    transition: all 0.3s ease;
  }
  .fade-zoom-enter-from,
  .fade-zoom-leave-to {
    opacity: 0;
    transform: scale(0.96);
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

    .wiki-header .pearl {
      padding: 16px 24px;
      h1 {
        font-size: 1.5rem;
      }
    }

    .header-actions {
      flex-direction: column;
      align-items: stretch;

      .search-input {
        width: 100%;
        min-width: 0;
      }
    }

    .entry-card .card-actions {
      flex-direction: column;
      align-items: flex-start;

      .action-group {
        width: 100%;
        justify-content: flex-end;
      }
    }

    .modal {
      padding: 20px;

      .modal-actions {
        flex-direction: column;
        .btn {
          width: 100%;
        }
      }
    }
  }
}
</style>
