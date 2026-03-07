<template>
  <div class="cantarella-resources">
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
        v-for="(src, idx) in randomFive2"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
        alt=""
      />
    </div>

    <!-- 头部：深海明珠 -->
    <header class="resources-header">
      <div class="pearl">
        <h1>翡萨烈·渊海秘藏</h1>
        <p class="subtitle">可自由上传关于坎特蕾拉的相关链接</p>
      </div>
     
    </header>

    <main class="container">
      <!-- 上传区（可折叠） -->
      <section class="uploader" :class="{ collapsed: uploaderCollapsed }">
        <div class="uploader-head">
          <button class="toggle" @click="toggleUploader" :aria-expanded="!uploaderCollapsed">
            <span v-if="uploaderCollapsed">展开上传区</span>
            <span v-else>收起上传区</span>
          </button>
        </div>

        <form @submit.prevent="addResource" class="upload-form" :aria-hidden="uploaderCollapsed">
          <div class="row">
            <input
              v-model="form.title"
              type="text"
              placeholder="标题（必填，如有解压码请写这里）"
              aria-label="标题"
            />
            <input
              v-model="form.type"
              type="text"
              placeholder="链接类型（网页、B站、网盘等）"
              aria-label="来源"
            />
          </div>

          <div class="row">
            <input
              v-model="form.uploader"
              type="text"
              placeholder="上传人（可选）"
              aria-label="上传人"
            />
            <input
              v-model="form.link"
              type="url"
              placeholder="链接（只输入网址，不能有中文）"
              aria-label="链接"
            />
          </div>

          <div class="actions">
            <button type="submit" class="btn primary">沉入幻梦 · 上传</button>
          </div>
        </form>
      </section>

      <!-- 资源列表 -->
      <section class="list">
        <div class="list-header">
          <h2>幻梦回响 · {{ resources.length }}</h2>
          <div class="sort">
            <label>
              排序：
              <select v-model="sortBy">
                <option value="time">按时间（新→旧）</option>
                <option value="likes">按点赞（高→低）</option>
              </select>
            </label>
          </div>
        </div>

        <ul class="items">
          <li v-for="item in sortedResources" :key="item.id" class="item">
            <div class="item-inner">
              <a
                :href="item.link"
                target="_blank"
                rel="noopener noreferrer"
                class="title"
              >{{ item.title }}</a>

              <div class="meta">
                <div class="left">
                  <span class="uploader">{{ item.uploader || "匿名" }}</span>
                  <span class="dot">⌇</span>
                  <time :datetime="item.time">{{ formatTime(item.time) }}</time>
                </div>

                <div class="right">
                  <button
                    @click.prevent="handleLike(item)"
                    :aria-pressed="likedIds.has(String(item.id))"
                    class="like-btn"
                    :class="{ active: likedIds.has(String(item.id)) }"
                  >
                    <span class="heart"></span>
                    <span class="count">{{ item.likes }}</span>
                  </button>

                  <span class="badge">{{ item.type }}</span>
                </div>
              </div>
              <div class="item-depth"></div>
            </div>
          </li>
        </ul>

        <p v-if="resources.length === 0" class="empty">
          深海静谧，尚无回响……
        </p>
      </section>
    </main>

    <footer class="foot">提示：点击标题将直接跳转至外部链接</footer>
  </div>
</template>

<script setup lang="ts">
// 脚本部分完全保留原逻辑，不做任何改动
import { ref, computed, onMounted, onBeforeUnmount } from "vue";
import {
  getResourceList,
  createResource,
  likeResource,
} from "@/api/modules/resource";
import { ElMessage } from "element-plus";

interface Resource {
  id: number | string;
  title: string;
  uploader?: string;
  time: string;
  likes: number;
  link: string;
  type: string;
  role_key?: string;
}

const STORAGE_KEY = "fll_resources_v1";
const DEFAULT_ROLE = "kante";

const form = ref<{
  title: string;
  uploader: string;
  link: string;
  type: string;
}>({
  title: "",
  uploader: "",
  link: "",
  type: "",
});

const resources = ref<Resource[]>([]);
const likedIds = ref(new Set<string>());
const sortBy = ref<"time" | "likes">("time");
const uploaderCollapsed = ref(false);

function mapServerToLocal(row: any): Resource {
  return {
    id: row.id,
    title: row.title,
    uploader: row.uploader || "匿名",
    time: row.created_at || row.time || new Date().toISOString(),
    likes: row.likes ?? 0,
    link: row.link,
    type: row.storage_type || row.type || "other",
    role_key: row.role_key,
  };
}

async function loadResources() {
  try {
    const res: any = await getResourceList({
      role_key: DEFAULT_ROLE,
      page: 1,
      pageSize: 100,
    });
    if (res && res.success && Array.isArray(res.data)) {
      resources.value = res.data.map(mapServerToLocal);
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        try {
          const parsed = JSON.parse(raw);
          if (parsed.liked && Array.isArray(parsed.liked)) {
            parsed.liked.forEach((id: string) => likedIds.value.add(id));
          }
        } catch (e) {}
      }
      return;
    }
  } catch (err) {
    console.warn("拉取资源失败，使用本地缓存", err);
  }
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) {
      const parsed = JSON.parse(raw);
      if (parsed.liked && Array.isArray(parsed.liked)) {
        parsed.liked.forEach((id: string) => likedIds.value.add(id));
      }
    }
  } catch (e) {}
}

function saveLocalCache() {
  try {
    const liked = Array.from(likedIds.value);
    localStorage.setItem(STORAGE_KEY, JSON.stringify({ liked }));
  } catch (e) {}
}

// 背景图片导入与轮播
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
let Imgtimer: number | undefined;

onMounted(() => {
  loadResources();
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  uploaderCollapsed.value = window.innerWidth <= 640;
});
function toggleUploader() {
  uploaderCollapsed.value = !uploaderCollapsed.value;
}
onBeforeUnmount(() => {
  if (Imgtimer) clearInterval(Imgtimer);
});

async function addResource() {
  const t = form.value.title.trim();
  const l = form.value.link.trim();
  if (!form.value.title.trim() || !form.value.link.trim()) {
    return ElMessage.warning("请填写完整信息");
  }
  if (!/^https?:\/\//i.test(l)) {
    return ElMessage.error("请输入正确的链接(https开头)");
  }
  try {
    const payload = {
      title: t,
      uploader: form.value.uploader.trim() || "匿名",
      link: l,
      storage_type: form.value.type,
      role_key: DEFAULT_ROLE,
    };
    const res: any = await createResource(payload);
    if (res && res.success && res.data) {
      const added = mapServerToLocal(res.data);
      resources.value.unshift(added);
      saveLocalCache();
      resetForm();
      ElMessage.success("上传成功");
      return;
    }
    ElMessage.error("上传失败");
  } catch (err) {
    console.warn("创建资源失败", err);
  }
}

function resetForm() {
  form.value.title = "";
  form.value.uploader = "";
  form.value.link = "";
  form.value.type = "";
}

async function handleLike(item: Resource) {
  const id = item.id;
  const wasLiked = likedIds.value.has(String(id));
  if (wasLiked) {
    likedIds.value.delete(String(id));
    item.likes = Math.max(0, item.likes - 1);
  } else {
    likedIds.value.add(String(id));
    item.likes++;
  }
  saveLocalCache();

  try {
    const action = wasLiked ? "unlike" : "like";
    const res: any = await likeResource(id, action);
    if (res && res.success && res.data && typeof res.data.likes !== "undefined") {
      item.likes = res.data.likes;
    }
  } catch (err) {
    console.warn("点赞接口调用失败，回滚本地状态", err);
    if (wasLiked) {
      likedIds.value.add(String(id));
      item.likes++;
    } else {
      likedIds.value.delete(String(id));
      item.likes = Math.max(0, item.likes - 1);
    }
    saveLocalCache();
  }
}

const sortedResources = computed(() => {
  const arr = [...resources.value];
  if (sortBy.value === "time") {
    arr.sort((a, b) => +new Date(b.time) - +new Date(a.time));
  } else {
    arr.sort((a, b) => b.likes - a.likes);
  }
  return arr;
});

function formatTime(iso: string) {
  try {
    const d = new Date(iso);
    return new Intl.DateTimeFormat("zh-CN", {
      month: "2-digit",
      day: "2-digit",
      hour: "2-digit",
      minute: "2-digit",
    }).format(d);
  } catch (e) {
    return iso;
  }
}
</script>

<style scoped lang="scss">
/* 坎特蕾拉色板 */
$deep-bg: #030614;        // 极深海
$mid-bg: #14213d;         // 深海中层
$accent-lavender: #b8a9ff; // 薰衣草紫
$accent-aqua: #7ae2ff;     // 水母荧光
$accent-pink: #ffb3c6;     // 毒药粉
$accent-deep-blue: #2a4a7a; // 深海蓝
$text-light: #f0f5fe;
$card-bg: rgba(255, 255, 255, 0.02);
$glass-edge: rgba($accent-aqua, 0.2);

.cantarella-resources {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(180deg, $deep-bg 0%, $mid-bg 100%);
  color: $text-light;
  overflow-x: hidden;
  padding: 80px 20px 60px;

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
      background: linear-gradient(180deg, rgba($deep-bg, 0.6) 0%, rgba($mid-bg, 0.8) 100%);
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
  .carousel-mobile { display: none; }

  /* 头部深海明珠 */
  .resources-header {
    position: relative;
    z-index: 10;
    max-width: 960px;
    margin: 0 auto 30px;
    text-align: center;

    .pearl {
      display: inline-block;
      background: rgba(6, 10, 20, 0.6);
      backdrop-filter: blur(2px);
      border: 1px solid $glass-edge;
      border-radius: 120px;
      padding: 20px 48px;
      box-shadow: 0 0 50px rgba($accent-aqua, 0.2), 0 20px 40px rgba(0,0,0,0.4);
      position: relative;
      overflow: hidden;

      &::before {
        content: "";
        position: absolute;
        inset: 0;
        background: radial-gradient(circle at 30% 30%, rgba($accent-lavender, 0.2), transparent 70%);
        pointer-events: none;
      }

      h1 {
        margin: 0;
        font-size: 2rem;
        font-weight: 400;
        background: linear-gradient(135deg, #ffffff, $accent-lavender, $accent-aqua);
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

   
  }

  /* 主容器 */
  .container {
    position: relative;
    z-index: 5;
    max-width: 960px;
    margin: 0 auto;
    padding: 0 20px;
  }

  /* 上传区 */
  .uploader {
    background: rgba(6, 10, 20, 0.5);
    backdrop-filter: blur(2px);
    border: 1px solid $glass-edge;
    border-radius: 40px 40px 30px 30px;
    margin-bottom: 30px;
    box-shadow: 0 20px 40px rgba(0,0,0,0.4);
    overflow: hidden;
    transition: all 0.3s;

    .uploader-head {
      display: flex;
      justify-content: flex-end;
      padding: 12px 20px;

      .toggle {
        background: transparent;
        border: 1px solid $glass-edge;
        border-radius: 30px;
        padding: 6px 18px;
        color: $accent-aqua;
        cursor: pointer;
        font-size: 0.9rem;
        transition: all 0.2s;

        &:hover {
          background: rgba($accent-aqua, 0.1);
          border-color: $accent-aqua;
        }
      }
    }

    .upload-form {
      padding: 0 20px 20px;
      transition: all 0.3s;

      .row {
        display: flex;
        gap: 12px;
        margin-bottom: 16px;

        input {
          flex: 1;
          padding: 12px 18px;
          background: rgba(0,0,0,0.3);
          border: 1px solid rgba($accent-aqua, 0.2);
          border-radius: 40px;
          color: $text-light;
          font-size: 0.95rem;
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
      }

      .actions {
        display: flex;
        justify-content: flex-end;

        .btn.primary {
          padding: 12px 32px;
          background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
          border: none;
          border-radius: 40px;
          color: $deep-bg;
          font-weight: 600;
          cursor: pointer;
          box-shadow: 0 0 20px rgba($accent-aqua, 0.4);
          transition: all 0.2s;

          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 0 30px rgba($accent-aqua, 0.7);
          }
        }
      }
    }

    &.collapsed {
      .upload-form {
        padding-top: 0;
        padding-bottom: 0;
        height: 0;
        overflow: hidden;
      }
    }
  }

  /* 列表区 */
  .list {
    background: rgba(6, 10, 20, 0.5);
    backdrop-filter: blur(2px);
    border: 1px solid $glass-edge;
    border-radius: 40px 40px 30px 30px;
    padding: 20px;

    .list-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      padding: 0 10px;

      h2 {
        margin: 0;
        font-size: 1.4rem;
        font-weight: 400;
        color: $accent-aqua;
        letter-spacing: 1px;
      }

      .sort {
        select {
          padding: 8px 16px;
          background: rgba(0,0,0,0.3);
          border: 1px solid $glass-edge;
          border-radius: 40px;
          color: $text-light;
          font-size: 0.9rem;
          cursor: pointer;
          outline: none;

          option {
            background: $deep-bg;
          }
        }
      }
    }

    .items {
      list-style: none;
      padding: 0;
      margin: 0;
      max-height: 60vh;
      overflow-y: auto;

      &::-webkit-scrollbar {
        width: 6px;
      }
      &::-webkit-scrollbar-thumb {
        background: $accent-aqua;
        border-radius: 6px;
      }
    }

    .item {
      margin-bottom: 16px;
      padding: 0 10px;

      .item-inner {
        background: $card-bg;
        backdrop-filter: blur(2px);
        border: 1px solid $glass-edge;
        border-radius: 40px 40px 30px 30px;
        padding: 18px 20px;
        position: relative;
        overflow: hidden;
        transition: all 0.3s;

        &::before {
          content: '';
          position: absolute;
          top: -10px;
          left: 10%;
          width: 80%;
          height: 30px;
          background: radial-gradient(ellipse at center, rgba($accent-aqua, 0.15), transparent 70%);
          filter: blur(2px);
          pointer-events: none;
        }

        &:hover {
          transform: translateY(-4px);
          border-color: rgba($accent-aqua, 0.4);
          box-shadow: 0 20px 30px rgba($accent-aqua, 0.2);
        }

        .title {
          display: block;
          font-size: 1.2rem;
          font-weight: 500;
          color: $accent-lavender;
          text-decoration: none;
          margin-bottom: 12px;
          word-break: break-word;
          padding-left: 8px;

          &:hover {
            text-decoration: underline;
            color: $accent-aqua;
          }
        }

        .meta {
          display: flex;
          justify-content: space-between;
          align-items: center;
          font-size: 0.9rem;

          .left {
            display: flex;
            align-items: center;
            gap: 8px;

            .uploader {
              color: $accent-aqua;
              font-weight: 600;
              margin: auto 0;
              padding: 12px;
            }

            .dot {
              color: $accent-pink;
              font-size: 1.2rem;
              opacity: 0.6;
            }

            time {
              color: rgba($text-light, 0.7);
            }
          }

          .right {
            display: flex;
            align-items: center;
            gap: 12px;

            .like-btn {
              background: transparent;
              border: none;
              display: flex;
              align-items: center;
              gap: 6px;
              cursor: pointer;
              padding: 4px 12px;
              border-radius: 30px;
              transition: all 0.2s;
              border: 1px solid transparent;

              .heart {
                width: 20px;
                height: 20px;
                background: url("/icons/heart-red-outline.svg") no-repeat center;
                background-size: contain;
                filter: drop-shadow(0 0 6px $accent-aqua);
                transition: all 0.2s;
              }

              &.active .heart {
                background: url("/icons/heart-red-filled.svg") no-repeat center;
                background-size: contain;
                filter: drop-shadow(0 0 10px $accent-pink);
              }

              .count {
                color: $text-light;
                font-size: 0.9rem;
                min-width: 20px;
                text-align: center;
              }

              &:hover {
                border-color: $glass-edge;
                background: rgba($accent-aqua, 0.05);
              }
            }

            .badge {
              padding: 4px 12px;
              border-radius: 40px;
              background: rgba($accent-deep-blue, 0.5);
              border: 1px solid $glass-edge;
              color: $accent-aqua;
              font-size: 0.8rem;
              font-weight: 400;
            }
          }
        }

        .item-depth {
          height: 10px;
          background: linear-gradient(to bottom, transparent, rgba($accent-deep-blue, 0.2));
          margin: 12px -20px -18px -20px;
          border-radius: 0 0 30px 30px;
        }
      }
    }

    .empty {
      text-align: center;
      padding: 40px 20px;
      color: rgba($text-light, 0.3);
      font-style: italic;
      font-size: 1rem;
    }
  }

  /* 页脚 */
  .foot {
    position: relative;
    z-index: 5;
    text-align: center;
    margin-top: 30px;
    color: rgba($text-light, 0.3);
    font-size: 0.85rem;
  }



  /* 移动端适配 */
  @media (max-width: 720px) {
    padding: 60px 12px 40px;

    .carousel-desktop { display: none; }
    .carousel-mobile { display: block; }

    .resources-header .pearl {
      padding: 16px 24px;
      h1 { font-size: 1.5rem; }
    }

    .uploader .upload-form .row {
      flex-direction: column;
    }

    .list .list-header {
      flex-direction: column;
      gap: 12px;
    }

    .item .meta {
      flex-direction: column;
      align-items: flex-start !important;
      gap: 8px;
    }
  }
}
</style>