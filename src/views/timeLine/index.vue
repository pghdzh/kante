<template>
  <div class="cantarella-wiki">
    <!-- 浮动水母装饰 -->
    <div class="floating-jellyfish" aria-hidden="true">
      <span v-for="i in 6" :key="i" class="jelly"></span>
    </div>

    <!-- 珊瑚触须装饰 -->
    <div class="coral-tentacles left" aria-hidden="true">
      <span></span><span></span><span></span><span></span>
    </div>
    <div class="coral-tentacles right" aria-hidden="true">
      <span></span><span></span><span></span><span></span>
    </div>

    <main class="container" ref="mainContainer">
      <!-- HERO 区域 -->
      <header class="hero" role="banner">
        <div class="left-col">
          <div class="avatar-card" @click="triggerAvatarFloat">
            <picture class="avatar-wrap">
              <img
                src="@/assets/images1/slty (2).webp"
                class="avatar"
                loading="lazy"
                decoding="async"
              />
            </picture>
            <div class="avatar-meta">
              <span class="avatar-badge">共鸣能力：海中梦</span>
              <span class="avatar-badge">翡萨烈·第36任家主</span>
            </div>
            <div class="avatar-glow"></div>
          </div>
        </div>
        <div class="right-col" ref="metaCard">
          <div class="hero-panel">
            <h1 class="name">坎特蕾拉 <span class="jp">CANTARELLA</span></h1>
            <p class="subtitle">翡萨烈现任家主 · 幻海织梦者</p>
            <p class="lead">
              神秘、高贵、美艳、剧毒……她居于山巅的冠冕，而山崖间流淌着的，
              都是由她编织的幻梦。当水母群从伞下涌出，翩飞成一片幻影之海。
            </p>
          </div>
        </div>
      </header>

      <!-- 新增：时间线板块（整合官方剧情） -->
      <section id="timeline" class="card timeline-section">
        <div class="card-head">
          <h2>时间线 · 往事如梦</h2>
          <span class="head-decoration"></span>
        </div>
        <div class="timeline-wrap">
          <div class="timeline-line"></div>
          <div class="timeline-items">
            <div
              v-for="(item, idx) in timelineData"
              :key="idx"
              class="timeline-item"
              :class="`phase-${idx}`"
            >
              <div class="timeline-dot"></div>
              <div class="timeline-card">
                <h3 class="timeline-title">{{ item.title }}</h3>
                <p class="timeline-desc">{{ item.desc }}</p>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- 核心机制卡片（保留原有并优化） -->
      <section id="personality" class="card personality">
        <div class="card-head">
          <h2>核心机制 · 迷离幻梦</h2>
          <span class="head-decoration"></span>
        </div>
        <ul class="traits">
          <li v-for="(trait, idx) in combatTraits" :key="idx">
            <span class="trait-icon"></span>
            {{ trait }}
          </li>
        </ul>
      </section>

      <!-- 角色背景与形象 -->
      <section id="overview" class="card overview">
        <div class="card-head">
          <h2>角色背景 · 幽海秘闻</h2>
          <span class="head-decoration"></span>
        </div>
        <div class="two-col">
          <div class="text-block">
            <h3>背景溯源</h3>
            <p>
              翡萨烈家族是黎那汐塔最为古老的家族之一，与隐海修会来往密切。作为第36任家主，坎特蕾拉长居于悬崖之上的波蒂维诺堡——飞瀑穿流，毒花环绕，淡紫色的雾气氤氲在朦胧的夜色中。
            </p>
            <p>
              幼年时，她被困于空旷宅邸，只能透过窗棂仰望飞鸟，幻想林间的公主冒险。那些天真的幻想，最终化作梦境中的水母，畅游于无边幽海。
            </p>
          </div>
          <div class="text-block">
            <h3>外貌着装</h3>
            <p>
              紫丝绸般的长发如瀑布散落，蓝紫色眼眸氤氲着朦胧雾气，深不见底，似有幽海翻涌。胸口佩戴深蓝色蓝宝石吊坠——这是翡萨烈家主的证明。身着的衣裙随光线变幻色彩，如蛰伏深海的剧毒之物，美丽、危险、迷幻。
            </p>
          </div>
        </div>
      </section>

      <!-- 共鸣状况 -->
      <section id="resonance-status" class="card status-card">
  <div class="card-head">
    <h2>共鸣状况 · 幻海织梦</h2>
    <span class="head-decoration"></span>
  </div>

  <div class="status-container">
    <!-- 公开报告：药剂师罗斯玛丽的检验记录 -->
    <div class="report-card public">
      <div class="report-header">
        <div class="report-badge">📋 公开 · 检验记录</div>
        <div class="report-seal">翡萨烈·特许</div>
      </div>
      <h3 class="report-title">频谱检验报告</h3>
      <div class="report-meta">「级别：公开」 「翡萨烈药剂师罗斯玛丽的检验记录-家主特许版」</div>
      <div class="report-content">
        <p>显而易见，现任翡萨烈家主坎特蕾拉代表着家族的最高毒药异能。而本人受邀来宅邸小坐，能为她做一次检查，并获得了可自由出入波蒂维诺堡花园进行研究的许可，已经倍感光荣。</p>
        <p>家主的毒药异能是家族中极为少见的<strong class="highlight">幻觉毒</strong>，区别于一般翡萨烈成员常见的致死毒，幻觉毒并不会立刻致死。当水母群从她的伞中游出，她的身边会形成一片美轮美奂的幻影之海。中毒之人先是会看到幻象——可能是内心最深处的渴望，抑或是家主编织的谎言。若是沉沦幻海太久，先是麻痹，进而昏厥，最严重时，才会致命。</p>
        <p>家主的毒药异能炉火纯青，最精妙时，可使人看到幻觉，而不造成任何实质损伤。</p>
        <p>虽说翡萨烈家的毒药异能是一把双刃剑，发动异能时往往自己也会受到一定损伤……但从家主一贯优雅得体的行为表现来看，并没有受到「幻梦」太大的影响吧……<span class="doubt">应该？</span></p>
      </div>
      <div class="report-footer">
        <div class="signature">—— 药剂师 罗斯玛丽</div>
        <div class="annotation">备注：家主对自身异能状况把握得非常清晰，破例让我为她进行检查，不免令我有些紧张……我应该没有弄错什么吧？</div>
      </div>
    </div>

    <!-- 中央引语 -->
    <div class="status-quote">
      <div class="quote-mark">❝</div>
      <p>「用自己的灵魂熬制一锅独一无二的秘药，让那些深埋海底的幻想从过去的时光里飞出来。」</p>
      <div class="quote-mark close">❞</div>
    </div>

    <!-- 机密报告：管家赛巴斯的记录 -->
    <div class="report-card confidential">
      <div class="report-header">
        <div class="report-badge">⚠️ 最高机密 · 精神健康记录</div>
        <div class="report-seal red">绝密</div>
      </div>
      <h3 class="report-title">超频诊断报告</h3>
      <div class="report-meta">「级别：最高机密」 「翡萨烈管家赛巴斯的记录-家主精神健康管理记录」</div>
      <div class="report-content">
        <p>幼年的经历、试炼的磋磨、以及长期使用毒药异能「幻梦」都对家主的身心造成<strong class="highlight warning">不可逆的严重影响</strong>。</p>
        <p>目前，家主的频段存在部分缺失，从波形增量来看，缺失部分的波形应该极为陡峭，濒临超频边界……</p>
        <p>留存部分波形呈现少见的<strong class="highlight warning">双相背离状</strong>，严重时会分离出数条不同波形：</p>
        <ul class="wave-list">
          <li><span class="wave-label">α波形</span>区间波动剧烈。</li>
          <li><span class="wave-label">β波形</span>非常稳定，没有任何异常波动倾向。</li>
          <li><span class="wave-label">γ波形</span>相幅较短，似乎并未发育成熟。</li>
          <li><span class="wave-label glitch">δ波形</span><span class="glitch-text">▇#%/@▇▇</span></li>
        </ul>
        <div class="system-corrupt">
          <span class="corrupt-line">（*……（@！#￥#*（……*￥▇▇▇▇▇▇%&#%￥#%&￥%▇▇#……%</span>
          <span class="corrupt-line">系统紊乱、警告、系统紊乱、乱▇▇▇%&#</span>
        </div>
      </div>
      <div class="report-footer">
        <div class="signature warning">—— 管家 赛巴斯</div>
        <div class="annotation glitch-note">
          <span class="glitch-hint">这里几个波形应该是象征着家主的其他人格具体可以参见下面的角色故事</span>
        </div>
      </div>
    </div>
  </div>
</section>

      <!-- 三大信物卡片（优化原有内容） -->
      <section id="items" class="card items-grid">
  <div class="card-head">
    <h2>珍贵信物 · 幽海见证</h2>
    <span class="head-decoration"></span>
  </div>
  <div class="items-container">
    <!-- 珊瑚骨洋伞 -->
    <div class="item-card">
      <div class="item-img">
        <img src="./img/item1.png" alt="珊瑚骨洋伞" />
        <span class="img-tentacles"></span>
      </div>
      <div class="item-info">
        <h4>「珊瑚骨」洋伞</h4>
        <div class="item-desc">
          <p>坎特蕾拉随身携带的洋伞。伞盖圆润如水母，伞骨由深海珊瑚打磨，握柄镶嵌蓝宝石。</p>
          <p class="hover-desc">转动时，水母群从伞下涌出，翩飞成幻影之海。看似温柔，实则坚不可摧。</p>
        </div>
      </div>
      <div class="item-depth"></div>
    </div>

    <!-- 美梦坩埚 -->
    <div class="item-card">
      <div class="item-img">
        <img src="./img/item2.png" alt="美梦坩埚" />
        <span class="img-tentacles"></span>
      </div>
      <div class="item-info">
        <h4>「美梦」坩埚</h4>
        <div class="item-desc">
          <p>「海中女巫」熬制秘药的坩埚，镶嵌蓝宝石。随性丢入水母触须、鱼鳞、泪水……</p>
          <p class="hover-desc">最后都能熬出一锅美梦，用甘甜化开现实的咸苦。</p>
        </div>
      </div>
      <div class="item-depth"></div>
    </div>

    <!-- 秘药瓶 -->
    <div class="item-card">
      <div class="item-img">
        <img src="./img/item3.png" alt="秘药瓶" />
        <span class="img-tentacles"></span>
      </div>
      <div class="item-info">
        <h4>秘药瓶</h4>
        <div class="item-desc">
          <p>从不离身的秘药瓶，存放精心调配的秘药。如今瓶中盛放难得的真心与柔情。</p>
          <p class="hover-desc">微光穿透深海，映照隐现于浪潮间的温柔呢喃。</p>
        </div>
      </div>
      <div class="item-depth"></div>
    </div>

    <!-- 秘制帕尼尼 -->
    <div class="item-card">
      <div class="item-img">
        <img src="./img/food.png" alt="秘制帕尼尼" />
        <span class="img-tentacles"></span>
      </div>
      <div class="item-info">
        <h4>秘制帕尼尼</h4>
        <div class="item-desc">
          <p>坎特蕾拉亲手制作的帕尼尼，酥脆面包塞满海鲜内馅。</p>
          <p class="hover-desc">一口下去，满足感填满腹腔，是简单温暖的味道。</p>
        </div>
      </div>
      <div class="item-depth"></div>
    </div>
  </div>
</section>
      <!-- 新增：彩蛋板块 -->
      <section id="easter-eggs" class="card easter-eggs">
        <div class="card-head">
          <h2>幻梦寻踪 · 隐藏彩蛋</h2>
          <span class="head-decoration"></span>
        </div>
        <div class="eggs-container">
          <div class="egg-item">
            <div class="egg-time">⏰ 11:00</div>
            <h4>隐海修会 · 午间邀约</h4>
            <p>
              隐海修会上方，遮阳伞旁。调整游戏时间至上午11:00，可触发坎特蕾拉邀约，前往她的府邸。
            </p>
          </div>
          <div class="egg-item">
            <div class="egg-time">⏰ 15:00</div>
            <h4>府邸 · 下午茶时光</h4>
            <p>
              完成首次邀约后，在坎特蕾拉府邸内将时间调至下午15:00，享受限定下午茶互动。
            </p>
          </div>
          <div class="egg-note">
            ※ 需先完成2.2版本主线及坎特蕾拉伴星任务「幽夜幻梦」
          </div>
        </div>
      </section>

      <!-- 共鸣链详解（优化原有） -->
      <section id="resonance-chain" class="card timeline">
        <div class="card-head">
          <h2>共鸣链 · 幻梦之证</h2>
          <span class="head-decoration"></span>
        </div>
        <div class="tl-wrap">
          <div class="vl" aria-hidden="true"></div>
          <ol class="tl-list">
            <li v-for="(ev, i) in timeline" :key="i" class="tl-item">
              <div class="tl-card">
                <h3 class="tl-title">第 {{ i + 1 }} 链 · {{ ev.title }}</h3>
                <p class="tl-desc">{{ ev.desc }}</p>
              </div>
            </li>
          </ol>
        </div>
      </section>

      <!-- 坎特蕾拉纪事（保留原有故事） -->
      <section class="ll-stories" ref="storyRef">
        <div class="card">
          <header class="card-head">
            <h2 class="card-title">坎特蕾拉纪事</h2>
            <span class="head-decoration"></span>
          </header>
          <main class="card-body">
            <transition name="fade-slide" mode="out-in">
              <article :key="items[current].id" class="story">
                <h3 class="story-head">{{ items[current].title }}</h3>
                <div
                  class="story-content"
                  v-html="items[current].content"
                ></div>
              </article>
            </transition>
          </main>
          <footer class="card-foot">
            <button class="nav-btn" @click="prev" aria-label="上一则故事">
              ‹
            </button>
            <div class="pager">{{ current + 1 }} / {{ items.length }}</div>
            <button class="nav-btn" @click="next" aria-label="下一则故事">
              ›
            </button>
          </footer>
        </div>
      </section>

      <!-- 页脚装饰 -->
      <div class="page-footer">
        <div class="footer-tentacles">
          <span></span><span></span><span></span><span></span><span></span>
        </div>
        <p class="footer-quote">
          「我喜欢你凝视我的样子——和你在一起的时候，我不再是『毒药』，而是我自己。」
        </p>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

// ========== 新增整合的时间线数据 ==========
const timelineData = ref([
  {
    title: "童年 · 笼中之鸟",
    desc: "幼年困于空旷宅邸，只能透过窗棂仰望飞鸟。幻想林间的公主冒险，却只能研读《毒药配方大全》。父母日益消瘦，被低语侵蚀。",
  },
  {
    title: "试炼 · 生死一赌",
    desc: "成为圣女候选人，进入波蒂维诺堡。研制出「紫绒梦」分发给同伴。在深渊边缘将瑰红水母扣上头顶，赌出生机，最终成为新任家主。",
  },
  {
    title: "家主 · 荆棘之冠",
    desc: "戴上淬毒的荆棘冠，额头沁出的鲜血将它滋润得鲜亮。幻梦在血管里流淌，无法从身体中剥离。召开圆桌会议，决定追寻梦与现实的清明。",
  },
  {
    title: "归途 · 尘埃落定",
    desc: "阿维纽林学院的真相揭晓，翡萨烈家族获得自由。午后三点的阳光里，普通的茶散发着淡淡的甜味——一切尘埃落定后的幸福味道。",
  },
]);

// ========== 原有数据 ==========
const timeline = ref([
  {
    title: "在无尽展开的波涛里",
    desc: `施放共鸣技能时，回复1点【迷离】。共鸣技能翩跹、共鸣技能斑驳幻梦、共鸣回路感知汲取的伤害倍率提升50%。共鸣回路感知汲取免疫打断。`,
  },
  {
    title: "坠入迷离幻梦",
    desc: `共鸣解放陷溺可以使目标进入迷梦，坎特蕾拉触发惊醒的伤害倍率提升245%。`,
  },
  {
    title: "凝视着深渊",
    desc: `共鸣解放陷溺的伤害倍率提升370%。施放共鸣解放陷溺后，直接进入蜃镜状态。若当前已处于蜃镜状态，施放共鸣解放陷溺不会重复进入。`,
  },
  {
    title: "就像凝视自己的灵魂",
    desc: `蜃镜状态期间，治疗效果加成提升25%。`,
  },
  {
    title: "投到倒影的怀里",
    desc: `共鸣解放弥漫的织梦水母的最大召唤数增加5。`,
  },
  {
    title: "下坠、下坠……坠入更深的幻梦",
    desc: `普攻蛰幻的伤害倍率提升80%。施放共鸣解放陷溺时，坎特蕾拉的伤害无视目标30%防御，持续10秒。迷梦的前1.2秒，目标受到伤害时，若此次伤害没有附加迷梦，则目标不会触发惊醒。`,
  },
]);

const combatTraits = ref([
  "【迷离】：血量上方上限5点的特殊能量条。拥有【迷离】时，Z进入蜃境状态，A消耗【迷离】获取【颤栗】。",
  "【迷离】获取方式：变奏技能，普通状态下E，R，第三段普攻命中，水中。",
  "【颤栗】：血量上方上限3点的特殊能量点。拥有3点【颤栗】时，E施放【感知汲取】，让敌人进入迷梦。",
  "【颤栗】获取方式：蜃境状态下普攻命中。",
  "【迷梦】：降低目标的行动速度，持续6.5秒。目标受到伤害时，将触发一次惊醒，并清除迷梦。",
  "【迷梦】触发方式：蜃境状态下的普通E和强化E。",
  "基础输出流程：AAA-EQ-R-ZE-AAA-E【感知汲取】-AAA-E",
]);

const items = [
  {
    id: "1",
    title: "圆桌会议",
    content: `光线穿过彩窗，在猩红的沙发上投下一片星星点点的斑驳。

雾紫的房间内点了几盏昏黄的小灯，圆桌边坐着几个模糊的人影，有的施施然坐在光明处，有的悠悠然浸没在黑暗里。

领头的那位发话了——至少从座次看是这样没错。这位女士有着一头紫丝绸般的光泽长发，蓝紫色的眼睛里氤氲着朦胧的雾气，使她看起来有一种似梦非梦的神采。她慵懒地靠着沙发，漫不经心地拨弄着胸口的蓝宝石吊坠——每当她陷入纠结的境地，总是会不自觉这么做。

“今日召集大家前来，是想一起探讨一下，究竟应该停留在何处？「幻梦」，抑或现实？”

“我们沿着梦的边界遨游，是留在虚幻的此岸，还是向前一步，游向现实的彼岸？幻梦中有我们熟悉的一切，虚像绚烂而轻盈，如同泡沫一般。而去往现实的路上，或许会遭遇意想不到的危险……但也可能看到从未见过的风景。”

“要我说，你还是太软弱了，坎特蕾拉。哦不对，现在应该叫你家主大人了，虽然在我看来，你还远远算不上称职。”黑暗里传来慵懒而笃定的声音，同样瀑布般的长发，同样深蓝的吊坠，同样曼妙的身段——只是这位女士似乎更加年长，更加强势，她深紫色的眼眸闪烁着不容质疑的光芒，“有什么好选的，直接把你的「紫绒梦」倒进宅子后面的瀑布，让整个拉古那都陷入由你掌控的幻梦——这样，我们就可以去任何地方，任何地方都属于我们。”

“无效回答。哦不对，硬要说的话，你是投了「幻梦」一票。好的，虽然听起来有些癫狂且不切实际，但这毕竟是讨论，为了公平公正起见，我还是记上了。”一旁端坐着的少女拿起笔记本——上面布满了奇奇怪怪的涂画，她好不容易找了个被秘药配方挤得几乎快看不见的空隙，煞有介事地划上一横。“我弃权，因为这样的讨论不见得有什么意义。我想，你大概已经有了答案……而对我来说，哪里都一样。”少女合上笔记本，抖了抖洁白的披肩，光线照到她碧蓝的眼眸中，若暗若明。

“为什么要这么复杂呢？遵从本心不好吗？坎特蕾拉姐姐，你很喜欢看窗前的飞鸟，也很喜欢外面自由的生活，那就，遵从你的本心就可以了呀。”稚声稚气的小女孩一整个趴在沙发上，用手去抓光线里飞舞的灰尘。

“这是一个慎重的决定，请大家严肃讨论。”坎特蕾拉似乎有些无奈，“就从你开始，重新好好说一下自己的选择，以及相应的理由。”`,
  },
  {
    id: "2",
    title: "荆棘冠",
    content: `深紫色的、淬毒的冠冕，刚戴上的时候会有些刺痛，但额头沁出的鲜血会将它滋润得更加鲜亮、更有光泽。

这顶冠冕本就由无数人的血泪浇灌而成。深海中孕育的毒荆棘，要多少人迎着混沌托举，才能载起天马洁白的尾羽。

这是翡萨烈的荣耀，也是翡萨烈的宿命。

那么翡萨烈自己呢？在昏暗无光的海底承受着低语的啃噬，直到心中那一缕真实信仰的微光熄灭，彻底迷失在深渊中——下一个翡萨烈又会接过荆棘冠，前赴后继，无始无终。

香气扑鼻的剧毒花朵
无色无味的剧毒眼泪
脏腑异化，呼出毒雾
神经毒
生物毒
金属毒
……

翡萨烈们引以为傲的毒药毒剂，因毒而生的异能力，最终都会被低语消解……到头来最有用的时候，竟是为同伴们和自己寻求解脱。

直到「幻梦」的出现。当毒荆棘扎进我的肌肤，流淌的鲜血泛着奇异的色泽。下了水银的红茶是香醇的蜜酿，蘸了水母花汁的司康是甜美的佳肴，咽喉处传来的灼烧感就像囫囵吞下一整个樱桃，溺水窒息的感觉好似在漫无边际的海底遨游……疼痛渐渐褪去，身体和头脑都变得很轻。大家手中的毒药落到地上破碎，神情慢慢放松。

低语扰动不了幻梦编织的世界，幻海会轻柔地化解所有喧嚣。在五彩斑斓的波涛中，那顶坚韧的荆棘冠也被冲刷得很柔和……柔和到长出了软软的褶皱，静静地趴在我头顶。它变得轻盈而洁白，就好像天马洁白的尾羽。

所以，我投「幻梦」一票。虚幻的此岸有什么不好？大家不都在这里畅游了很久吗？直接把「紫绒梦」倒进宅子后面的瀑布，让整个拉古那都陷入由我掌控的幻梦——这当然是个玩笑。我无意掌控整个拉古那，只是想让大家都过得舒服点。

反正，「幻梦」就在血管里流淌，无论愿不愿意，都无法从身体中剥离。`,
  },
  {
    id: "3",
    title: "水母灯",
    content: `昏暗、气味奇特的房间里，一盏微弱、昏黄的小灯，照着一本陈旧的笔记本，还有一口古怪的坩埚。

身着白色披肩的少女一边在笔记本上写写画画，一边往坩埚里丢材料，冒出的烟雾把她整个人熏得黑一块紫一块，原本洁白的围裙也变得斑斑点点。她随意地抹了一把脸，毫不在意。

蒸馏水、椒薄荷、蓖麻豆、藤壶鱼叶、迷迭香……气味清新，毒性微弱，有舒缓镇痛之效，可以治疗最近常见的婴幼儿灰质症……记下这个方子，等会儿写在小纸条上，留给切里。
双耳草、海秋葵、藤壶鱼叶、海苹果汁……气味难闻，毒性强烈，可以中和部分神经麻痹反应。嗯……但是有点副作用……算了，此方留用。
海秋葵粘液、藤壶鱼叶晒干捣碎、一点点海苹果汁、少许瑰红水母花汁……瑰红水母毒性太强，虽然有奇效，但太过冒险。
藤壶鱼叶捣碎，淋上紫绒丝花汁，配合海秋葵粘液，再挤入一点点海苹果汁，放在琉璃瓶里发酵……成了。喝下后精神舒缓，可以中和大部分中毒反应。赶紧给大家都派发一点，好撑过试炼的这些天。嗯……名字的话，就叫「紫绒梦」吧。

用材用料、处理方式、熬制顺序、熬制手法、环境温度……在恰当的时机操作恰当的步骤，就能熬出理想中的秘药。

同样，我坚信，在恰当的时机操作恰当的步骤，就能解决眼下遇到的大多数问题。再不符合常理的规则、再惨烈的状况，都可以找到解决的步骤和方法。

于是我拎着提灯走入黑暗，直面恐惧——深渊嘲笑我孤僻、弱小、不自量力、不知变通，回廊里传来嘲弄般的呼啸、历代家主被低语扭曲的频率喷涌向前，将我淹没。认真、死板、苍白、阴郁、缄默……祂们太了解我了，一下子就能猜出我的应对步骤，毫不费力地抓住了我。

不过，我开了此生唯一一个，也是最后一个玩笑——我提前喝下大量「紫绒梦」，并在触手上的毒荆棘扎进皮肤的瞬间，将瑰红水母直接扣上自己的脑袋——几股剧烈的毒素在血管中冲撞，我几乎被撕扯成两半，灵魂在生死之间翻滚，甚至快要顺着烧灼的咽喉、从口中呕出来——分不清是血还是泪，又或者是汗滴和药渣……向下滴落在了快要熄灭的小灯上。

一只小水母从微弱的烛光中晃晃悠悠地飞了出来。再抬起眼，黑暗已经褪去，模糊的视线里，是五彩斑斓的幻梦。

成了。

所以我说，任何情况，都能通过在恰当的时机操作恰当的步骤来解决，即便自己是步骤的一环，也不要紧。

从那之后，我就一直徘徊在生与死，梦与醒之间，我经常能看到一些遗留的幻影，其中包括那个戴红帽子的女孩。她到底是谁，是我吗？还是另有其人？我想不明白。

算了，我想来想去，投票还是弃权。对于目前的我来说，「幻梦」还是现实都一样，习惯了，在哪都可以生存。`,
  },
  {
    id: "4",
    title: "窗前鸟",
    content: `从有记忆起，我就一直被养在又大、又空旷的宅邸里。我从屋子的一头走到另一头，地板上的每一块瓷砖、架子上的每一个药瓶，我都很熟悉。整个屋子都灰扑扑的，没什么劲，只有小鸟飞过窗前的时候，我顺着洁白的、上下扑棱的翅膀看去——小小的圆点在天空中越飞越远，远处的天空蓝澄澄的，林子绿油油的，很好看。

那时候我就会想，茂密的林子里会发生什么故事呢？会不会有公主冒险的故事？

有一名聪明的公主，在丛林里辨别了有毒的苹果，为了躲避毒蛇的追赶，乘着赶来救她的小鸟远行——她在天空中飞着，在云朵上救了穿着红舞鞋不停跳舞的女孩，又落到海里去，救了快化为泡沫的塞壬……小鸟载着她们飞呀飞呀，飞不动了，落到了一座城堡里……等等，城堡是什么样的？如果城堡和我现在住的地方一样，又大又无聊，那可不太好。

对不起，我讲的故事是不是有点乱？唔……爸爸妈妈说了，翡萨烈家的女孩不需要看什么乱七八糟的童话故事，也不需要有这些奇奇怪怪的想法……只要看书就可以了。看那些《毒药配方大全》《翡萨烈家传秘药谱集》《你不知道的108种毒物》……嗯……那些书也很有意思啦，就是看多了，还是会有些无聊。

但爸爸妈妈说了，要全部看完，背得滚瓜烂熟——学会那些知识，以后或许可以救我的命，还可以当上什么圣女？什么家主？救所有人的命？嗯……听起来责任很大呢，我还不太懂，不过我会乖乖听爸爸妈妈的话。

爸爸妈妈总是很忙，神色匆匆地走来走去，每次出去都会认认真真反锁家门，还要再反复确认检查几遍。这是为什么呢？为什么他们越来越忙，身形却越来越瘦削？为什么他们要喝越来越多的“水”，总是嘟囔着奇怪的话？

那些话我也会背啦，“翡萨烈的荣耀，翡萨烈的宿命”。

后来我长大了一些，要到别的宅子里去住了。送行的时候，爸爸妈妈脸上的表情好像很骄傲，眼里却含着泪水。

“是的，家主大人，这孩子资质出众，尤其擅长配制毒药。”
“是的，成为试炼的候选人，是翡萨烈的荣耀，也是翡萨烈的宿命。”

那是我第一次踏出家门，脚步声惊动了门口枯树上的乌鸦，它扑棱着黑色的翅膀，向着远处陌生的城堡飞去。

啊，对不起，是不是扯远了？我是说，嗯……就是，在天上，在梦中，飞了太久的话，也会怀念脚踏在土地上的感觉。可能会踩到泥巴、也可能会被小石子什么的扎痛，不过，一步一步踩在地上的感觉，很安心。

那……投票的话，我投现实一票，也叫做「梦的彼岸」，对吧？大人总是喜欢这种玄乎的说法。但，你知道内心最渴望的是什么吧？不仅在梦中，也可以在现实里推开门扉，和大家一起跟着洁白的飞鸟四处转转，去寻找让所有人都幸福的结局。`,
  },
  {
    id: "5",
    title: "清明梦",
    content: `庄严魅惑的声音，清冷坚定的声音，纯真童稚的声音，在雾紫的房间内穿梭交叠，形成梦幻般的声浪。

蓝宝石闪烁着的威严光芒，翻动书页的沙沙声响，刚刚烘烤好的糕点香气，在室内氤氲发酵，酿成一锅甜美的幻梦。

坎特蕾拉端起精美的茶壶，倒出刚刚煮好的新茶——今天这茶和往日有所不同，没有了迷幻的紫色，取而代之的是清冽的澄澈。

“感谢大家的宝贵意见，我已经拿定了主意。”坎特蕾拉放下茶壶，清新的香气在房间内扩散开。
“我将继续戴着荆棘冠，因为毒荆棘已经扎入骨血，而幻梦确实无法剥离。”
“我将继续熬制秘药，因为有人需要它们，不仅仅是翡萨烈。”
“我将跨出门扉追寻飞鸟，去山野间，去市井里。困住我们的低语囚笼已经消散，我们可以拥有这种自由。”
“我仍在幻梦里，但我会努力试着让脚落到地上……在那个半梦半醒的间隙，抓住一点点确定的实感。”

坎特蕾拉深深吸了一口气，缓缓睁开半昧着的双眼——她刚刚结束一场午后小睡，走出一段迷离又清醒的梦。

下午三点的阳光透过玻璃窗，静静照着她一人的身影。会客厅内只剩茶水的咕嘟声，还有她均匀的呼吸。

坎特蕾拉端起茶杯，抿了一口。清新的香气安抚了口舌间躁动的声痕，它不再从虚幻和疼痛中汲取力量。

这是她上午刚买的茶，在拉古那城里买的，普通的茶。

淡淡的甜味，若有似无的清香，让人联想到乡野间拂过的微风，还有青叶上滴下的露珠……

原来，这就是一切尘埃落定之后的，幸福的味道。

门吱呀一声，她又倒了一杯，给刚刚到访的客人。
这位客人有着漆黑的秀发，金色的眼瞳，浑身上下散发着定海石般坚定而朴素的气息。
而这样惬意缱绻的午后时光，如此宁静祥和的日常，便是【漂泊者】赐予的奇迹。

温柔的阳光拂过房间，清新的香气在两人之间扩散，萦绕在两个茶杯之间。

她抬起蓝紫色的眼睛，朦胧的雾气已经散去，平静海面般的眼眸泛起涟漪。`,
  },
];

const current = ref(0);
const storyRef = ref<HTMLElement | null>(null);
const reducedMotion =
  window.matchMedia &&
  window.matchMedia("(prefers-reduced-motion: reduce)").matches;

function select(index: number) {
  current.value = index;
}
function prev() {
  current.value = (current.value - 1 + items.length) % items.length;
  select(current.value);
  storyRef.value?.scrollIntoView({ behavior: "smooth", block: "start" });
}
function next() {
  current.value = (current.value + 1) % items.length;
  select(current.value);
  storyRef.value?.scrollIntoView({ behavior: "smooth", block: "start" });
}
function triggerAvatarFloat() {
  gsap.to(".avatar-card", {
    scale: 1.05,
    duration: 0.3,
    yoyo: true,
    repeat: 1,
  });
}

onMounted(() => {
  if (!reducedMotion) {
    // 入场动画

    gsap.from(".hero-panel", {
      duration: 1,
      x: 30,
      opacity: 0,
      delay: 0.3,
      ease: "power3.out",
    });

    // 浮动动画
    gsap.to(".avatar-card", {
      y: -6,
      duration: 3,
      repeat: -1,
      yoyo: true,
      ease: "sine.inOut",
    });
    gsap.to(".jelly", {
      y: -15,
      duration: () => 2 + Math.random() * 3,
      repeat: -1,
      yoyo: true,
      ease: "sine.inOut",
      stagger: 0.3,
    });

    // 滚动动画
    gsap.utils.toArray(".timeline-item").forEach((item: any, i) => {
      gsap.from(item.querySelector(".timeline-card"), {
        scrollTrigger: { trigger: item, start: "top 85%" },
        opacity: 0,
        x: -20,
        duration: 0.6,
        delay: i * 0.1,
      });
    });
  }
});
</script>

<style scoped lang="scss">
/* 坎特蕾拉色板 - 深海幻梦 */
$deep-bg: #030614;
$mid-bg: #14213d;
$accent-lavender: #b8a9ff;
$accent-aqua: #7ae2ff;
$accent-pink: #ffb3c6;
$accent-deep-blue: #2a4a7a;
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

  &.reduced-motion * {
    animation: none !important;
    transition: none !important;
  }

  /* 浮动水母 */
  .floating-jellyfish {
    position: fixed;
    inset: 0;
    z-index: 1;
    pointer-events: none;
    .jelly {
      position: absolute;
      width: 60px;
      height: 80px;
      background: radial-gradient(
        circle at 50% 30%,
        rgba($accent-aqua, 0.15),
        transparent 70%
      );
      border-radius: 50% 50% 40% 40%;
      filter: blur(8px);
      animation: float 8s infinite alternate;
      &:nth-child(1) {
        top: 15%;
        left: 5%;
        width: 80px;
        height: 100px;
        animation-duration: 12s;
      }
      &:nth-child(2) {
        top: 70%;
        left: 10%;
        width: 50px;
        height: 70px;
        animation-duration: 14s;
      }
      &:nth-child(3) {
        top: 30%;
        right: 8%;
        width: 70px;
        height: 90px;
        animation-duration: 10s;
      }
      &:nth-child(4) {
        bottom: 20%;
        right: 15%;
        width: 60px;
        height: 80px;
        animation-duration: 16s;
      }
      &:nth-child(5) {
        top: 50%;
        left: 20%;
        width: 40px;
        height: 60px;
        animation-duration: 18s;
      }
      &:nth-child(6) {
        bottom: 40%;
        right: 25%;
        width: 55px;
        height: 75px;
        animation-duration: 13s;
      }
    }
  }

  /* 珊瑚触须装饰 */
  .coral-tentacles {
    position: fixed;
    bottom: 0;
    z-index: 2;
    pointer-events: none;
    display: flex;
    gap: 10px;
    &.left {
      left: 20px;
      flex-direction: column;
    }
    &.right {
      right: 20px;
      flex-direction: column;
    }
    span {
      width: 4px;
      background: linear-gradient(to top, $accent-aqua, transparent);
      border-radius: 2px;
      animation: tentacleSway 4s infinite alternate;
      &:nth-child(1) {
        height: 80px;
        animation-delay: 0s;
      }
      &:nth-child(2) {
        height: 60px;
        animation-delay: 0.8s;
        background: $accent-lavender;
      }
      &:nth-child(3) {
        height: 100px;
        animation-delay: 0.3s;
      }
      &:nth-child(4) {
        height: 70px;
        animation-delay: 1.2s;
        background: $accent-pink;
      }
    }
  }

  .container {
    position: relative;
    z-index: 10;
    max-width: 1100px;
    margin: 0 auto;
    padding: clamp(12px, 2vw, 40px);
  }

  /* ===== HERO ===== */
  .hero {
    display: grid;
    grid-template-columns: 280px 1fr;
    gap: 28px;
    align-items: start;
    margin-bottom: 30px;
    .left-col {
      display: flex;
      justify-content: center;
    }
    .avatar-card {
      position: relative;
      background: $glass-bg;
      backdrop-filter: blur(12px);
      border: 1px solid $glass-edge;
      border-radius: 40px 40px 30px 30px;
      padding: 20px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
      cursor: pointer;
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
          rgba($accent-aqua, 0.2),
          transparent 70%
        );
        filter: blur(10px);
      }
      .avatar-glow {
        position: absolute;
        inset: 0;
        border-radius: inherit;
        background: radial-gradient(
          circle at 30% 30%,
          rgba($accent-aqua, 0.2),
          transparent 70%
        );
        opacity: 0;
        transition: opacity 0.3s;
      }
      &:hover .avatar-glow {
        opacity: 1;
      }
      .avatar-wrap img {
        width: 100%;
        border-radius: 30px;
        border: 1px solid $glass-edge;
      }
      .avatar-meta {
        margin-top: 12px;
        display: flex;
        flex-direction: column;
        gap: 6px;
        .avatar-badge {
          background: rgba($accent-deep-blue, 0.6);
          border: 1px solid $glass-edge;
          border-radius: 60px;
          padding: 4px 10px;
          font-size: 0.8rem;
          text-align: center;
          color: $accent-aqua;
        }
      }
    }
    .hero-panel {
      background: $glass-bg;
      backdrop-filter: blur(12px);
      border: 1px solid $glass-edge;
      border-radius: 60px 60px 40px 40px;
      padding: 28px 32px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
      .name {
        margin: 0;
        font-size: 2.5rem;
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
        .jp {
          font-size: 1rem;
          color: $accent-aqua;
          margin-left: 8px;
        }
      }
      .subtitle {
        font-size: 1.1rem;
        color: $accent-lavender;
        margin: 8px 0 12px;
        font-style: italic;
      }
      .lead {
        line-height: 1.8;
        opacity: 0.9;
      }
    }
  }

  /* 通用卡片 */
  .card {
    background: $glass-bg;
    backdrop-filter: blur(12px);
    border: 1px solid $glass-edge;
    border-radius: 60px 60px 40px 40px;
    padding: 28px 32px;
    margin-bottom: 30px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
    transition: all 0.3s;
    &:hover {
      transform: translateY(-4px);
      box-shadow: 0 30px 60px rgba($accent-aqua, 0.2);
    }
    .card-head {
      display: flex;
      align-items: center;
      gap: 16px;
      margin-bottom: 24px;
      h2 {
        margin: 0;
        font-size: 1.6rem;
        font-weight: 400;
        color: $accent-aqua;
        letter-spacing: 1px;
      }
      .head-decoration {
        flex: 1;
        height: 2px;
        background: linear-gradient(90deg, $accent-aqua, transparent);
      }
    }
  }

  /* 时间线板块（新增） */
  .timeline-section {
    .timeline-wrap {
      position: relative;
      padding: 20px 0;
    }
    .timeline-line {
      position: absolute;
      left: 30px;
      top: 0;
      bottom: 0;
      width: 2px;
      background: linear-gradient(
        180deg,
        $accent-aqua,
        $accent-lavender,
        $accent-pink
      );
      border-radius: 2px;
      box-shadow: 0 0 15px $accent-aqua;
    }
    .timeline-items {
      display: flex;
      flex-direction: column;
      gap: 30px;
    }
    .timeline-item {
      position: relative;
      padding-left: 70px;
      .timeline-dot {
        position: absolute;
        left: 22px;
        top: 15px;
        width: 16px;
        height: 16px;
        border-radius: 50%;
        background: $accent-aqua;
        box-shadow: 0 0 20px $accent-aqua;
        animation: pulse 2s infinite;
      }
      .timeline-card {
        background: rgba($deep-bg, 0.5);
        border: 1px solid $glass-edge;
        border-radius: 40px 40px 30px 30px;
        padding: 20px;
        h3 {
          margin: 0 0 8px;
          color: $accent-lavender;
          font-size: 1.2rem;
        }
        p {
          margin: 0;
          line-height: 1.6;
          opacity: 0.9;
        }
      }
    }
  }
/* 共鸣状况模块专属样式 - 添加到原有style中 */
.status-card {
  .status-container {
    display: flex;
    flex-direction: column;
    gap: 24px;
    position: relative;
  }

  /* 报告卡片通用样式 */
  .report-card {
    background: rgba($deep-bg, 0.6);
    backdrop-filter: blur(8px);
    border: 1px solid $glass-edge;
    border-radius: 40px 40px 30px 30px;
    padding: 24px 28px;
    position: relative;
    overflow: hidden;
    transition: all 0.4s cubic-bezier(0.2, 0.9, 0.3, 1);

    &::before {
      content: '';
      position: absolute;
      top: -10px;
      left: 10%;
      width: 80%;
      height: 30px;
      background: radial-gradient(ellipse at center, rgba($accent-aqua, 0.15), transparent 70%);
      filter: blur(10px);
      pointer-events: none;
    }

    &:hover {
      transform: translateY(-6px);
      box-shadow: 0 30px 60px rgba($accent-aqua, 0.15);
    }

    .report-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 16px;

      .report-badge {
        font-size: 0.85rem;
        padding: 4px 12px;
        border-radius: 40px;
        background: rgba($accent-deep-blue, 0.4);
        border: 1px solid rgba($accent-aqua, 0.2);
        color: $accent-aqua;
        letter-spacing: 0.5px;
      }

      .report-seal {
        font-size: 0.8rem;
        padding: 2px 10px;
        border-radius: 4px;
        background: linear-gradient(135deg, $accent-lavender, $accent-aqua);
        color: $deep-bg;
        font-weight: 600;
        transform: rotate(2deg);
        box-shadow: 0 2px 8px rgba($accent-aqua, 0.3);

        &.red {
          background: linear-gradient(135deg, $accent-pink, #d14b6e);
          color: #1a0a0a;
        }
      }
    }

    .report-title {
      font-size: 1.4rem;
      font-weight: 400;
      color: $accent-lavender;
      margin: 0 0 8px;
      letter-spacing: 1px;
    }

    .report-meta {
      font-size: 0.85rem;
      color: rgba($text-light, 0.5);
      border-bottom: 1px dashed rgba($accent-aqua, 0.2);
      padding-bottom: 12px;
      margin-bottom: 16px;
      font-style: italic;
    }

    .report-content {
      line-height: 1.8;
      color: rgba($text-light, 0.92);

      p {
        margin: 12px 0;
      }

      .highlight {
        color: $accent-aqua;
        font-weight: 500;
        text-shadow: 0 0 8px rgba($accent-aqua, 0.3);

        &.warning {
          color: $accent-pink;
          text-shadow: 0 0 8px rgba($accent-pink, 0.3);
        }
      }

      .doubt {
        font-style: italic;
        color: rgba($accent-lavender, 0.8);
        animation: flicker 3s infinite;
      }

      .wave-list {
        list-style: none;
        padding: 0;
        margin: 16px 0;

        li {
          padding: 6px 0 6px 24px;
          position: relative;
          border-bottom: 1px dashed rgba($accent-aqua, 0.1);

          &::before {
            content: '⌇';
            position: absolute;
            left: 4px;
            color: $accent-aqua;
            font-size: 1.2rem;
          }

          .wave-label {
            display: inline-block;
            width: 60px;
            color: $accent-lavender;
            font-weight: 600;

            &.glitch {
              color: $accent-pink;
            }
          }

          .glitch-text {
            color: rgba($text-light, 0.6);
            font-family: monospace;
            letter-spacing: 2px;
            animation: glitch 2s infinite;
          }
        }
      }

      .system-corrupt {
        margin-top: 20px;
        padding: 12px;
        background: rgba(0, 0, 0, 0.3);
        border-radius: 20px;
        font-family: monospace;
        color: rgba($accent-pink, 0.7);
        display: flex;
        flex-direction: column;
        gap: 6px;
        border: 1px dashed rgba($accent-pink, 0.3);
        animation: glitch-bg 4s infinite;

        .corrupt-line {
          display: block;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
          animation: text-flicker 3s infinite;

          &:nth-child(2) {
            animation-delay: 0.5s;
          }
        }
      }
    }

    .report-footer {
      margin-top: 20px;
      padding-top: 12px;
      border-top: 1px dashed rgba($accent-aqua, 0.15);

      .signature {
        text-align: right;
        font-size: 0.9rem;
        color: $accent-aqua;
        font-style: italic;

        &.warning {
          color: $accent-pink;
        }
      }

      .annotation {
        margin-top: 8px;
        font-size: 0.85rem;
        color: rgba($text-light, 0.6);
        background: rgba(0, 0, 0, 0.2);
        padding: 8px 12px;
        border-radius: 16px;
        border-left: 3px solid $accent-aqua;

        &.glitch-note {
          border-left-color: $accent-pink;
          .glitch-hint {
            animation: flicker 3s infinite;
          }
        }
      }
    }

    /* 机密报告特殊样式 */
    &.confidential {
      border-color: rgba($accent-pink, 0.3);
      box-shadow: 0 10px 30px rgba($accent-pink, 0.1);

      &::after {
        content: '';
        position: absolute;
        top: 10px;
        right: 10px;
        width: 60px;
        height: 60px;
        background: repeating-linear-gradient(
          45deg,
          rgba($accent-pink, 0.1) 0px,
          rgba($accent-pink, 0.1) 10px,
          transparent 10px,
          transparent 20px
        );
        pointer-events: none;
        opacity: 0.3;
      }
    }
  }

  /* 中央引语 */
  .status-quote {
    position: relative;
    margin: 10px 0;
    padding: 20px 40px;
    text-align: center;

    .quote-mark {
      position: absolute;
      font-size: 4rem;
      color: $accent-aqua;
      opacity: 0.2;
      font-family: serif;

      &.close {
        bottom: -10px;
        right: 0;
        transform: rotate(180deg);
      }
    }

    p {
      font-size: 1.2rem;
      font-style: italic;
      color: $accent-lavender;
      line-height: 1.8;
      max-width: 80%;
      margin: 0 auto;
      text-shadow: 0 0 20px rgba($accent-aqua, 0.2);
    }
  }

  /* 动画效果 */
  @keyframes flicker {
    0%, 100% { opacity: 1; }
    25% { opacity: 0.6; }
    30% { opacity: 0.8; }
    80% { opacity: 0.9; }
    85% { opacity: 0.4; }
  }

  @keyframes glitch {
    0%, 100% { transform: translate(0); }
    10% { transform: translate(-2px, 1px); }
    20% { transform: translate(2px, -1px); }
    30% { transform: translate(-1px, 2px); }
    40% { transform: translate(1px, -2px); }
    50% { transform: translate(-2px, -1px); }
    60% { transform: translate(2px, 1px); }
    70% { transform: translate(0); }
  }

  @keyframes glitch-bg {
    0%, 100% { background: rgba(0, 0, 0, 0.3); }
    10% { background: rgba($accent-pink, 0.1); }
    20% { background: rgba(0, 0, 0, 0.3); }
    30% { background: rgba($accent-pink, 0.1); }
    40% { background: rgba(0, 0, 0, 0.3); }
  }

  @keyframes text-flicker {
    0%, 100% { opacity: 0.7; }
    15% { opacity: 0.3; }
    30% { opacity: 0.9; }
    45% { opacity: 0.2; }
    60% { opacity: 0.8; }
    75% { opacity: 0.1; }
    90% { opacity: 0.7; }
  }
}

/* 移动端适配 */
@media (max-width: 720px) {
  .status-card {
    .report-card {
      padding: 18px 16px;
      
      .report-title {
        font-size: 1.2rem;
      }
      
      .system-corrupt .corrupt-line {
        white-space: normal;
        word-break: break-all;
      }
    }
    
    .status-quote p {
      font-size: 1rem;
      max-width: 100%;
    }
  }
}
  /* 核心机制 */
  .personality .traits {
    list-style: none;
    padding: 0;
    li {
      padding: 10px 0;
      border-bottom: 1px dashed rgba($accent-aqua, 0.2);
      display: flex;
      align-items: flex-start;
      gap: 8px;
      .trait-icon {
        width: 6px;
        height: 6px;
        border-radius: 50%;
        background: $accent-aqua;
        margin-top: 8px;
        box-shadow: 0 0 10px $accent-aqua;
      }
    }
  }

  /* 双栏布局 */
  .two-col {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 24px;
    .text-block {
      h3 {
        color: $accent-lavender;
        margin: 0 0 8px;
      }
      p {
        line-height: 1.8;
        opacity: 0.9;
      }
    }
  }
  .text-block.full {
    p {
      line-height: 1.8;
    }
  }
  .quote {
    font-style: italic;
    border-left: 4px solid $accent-aqua;
    padding-left: 20px;
    margin: 16px 0 0;
  }

  /* 信物网格 */
  .items-grid {
  .items-container {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 24px;
    margin-top: 20px;
  }

  .item-card {
    background: rgba($deep-bg, 0.5);
    backdrop-filter: blur(8px);
    border: 1px solid $glass-edge;
    border-radius: 60% 60% 30% 30% / 60% 60% 30% 30%;
    padding: 20px 18px 20px 18px;
    position: relative;
    overflow: hidden;
    transition: all 0.4s cubic-bezier(0.2, 0.9, 0.3, 1);
    display: flex;
    flex-direction: column;

    &::before {
      content: '';
      position: absolute;
      inset: 0;
      background: radial-gradient(circle at 20% 20%, rgba($accent-aqua, 0.1), transparent 70%);
      opacity: 0;
      transition: opacity 0.4s;
      pointer-events: none;
    }

    &:hover {
      transform: translateY(-10px) scale(1.02);
      border-color: rgba($accent-aqua, 0.5);
      box-shadow: 0 30px 50px rgba($accent-aqua, 0.2);
      &::before { opacity: 1; }
    }

    .item-img {
      position: relative;
      width: 100%;
      aspect-ratio: 1 / 1.1;
      border-radius: 50% 50% 30% 30%;
      overflow: hidden;
      margin-bottom: 16px;
      border: 2px solid rgba($accent-aqua, 0.2);
      box-shadow: 0 0 20px rgba($accent-aqua, 0.2);
      transition: all 0.3s;

      img {
        width: 100%;
        height: 100%;
        object-fit: cover;
        transition: transform 0.5s ease, filter 0.5s ease;
        filter: saturate(0.9) brightness(0.9);
      }

      .img-tentacles {
        position: absolute;
        bottom: -8px;
        left: 15%;
        width: 70%;
        height: 16px;
        background: radial-gradient(ellipse at center, rgba($accent-aqua, 0.3), transparent 70%);
        filter: blur(4px);
        border-radius: 50%;
        transition: all 0.3s;
      }

      &:hover {
        border-color: $accent-aqua;
        box-shadow: 0 0 30px $accent-aqua;

        img {
          transform: scale(1.08) rotate(2deg);
          filter: saturate(1.1) brightness(1.05);
        }
        .img-tentacles {
          height: 24px;
          background: radial-gradient(ellipse at center, rgba($accent-aqua, 0.5), transparent 70%);
        }
      }
    }

    .item-info {
      flex: 1;
      display: flex;
      flex-direction: column;

      h4 {
        margin: 0 0 10px;
        font-size: 1.2rem;
        font-weight: 500;
        color: $accent-lavender;
        display: flex;
        align-items: center;
        gap: 8px;

        &::before {
          content: '◆';
          color: $accent-aqua;
          font-size: 0.8rem;
          opacity: 0.7;
        }
      }

      .item-desc {
        position: relative;
        line-height: 1.6;
        color: rgba($text-light, 0.9);
        font-size: 0.9rem;

        p {
          margin: 0;
          transition: opacity 0.2s;
        }

        .hover-desc {
          position: absolute;
          top: 0;
          left: 0;
          opacity: 0;
          transform: translateY(10px);
          pointer-events: none;
          transition: opacity 0.3s, transform 0.3s;
          color: rgba($accent-aqua, 0.9);
          font-style: italic;
        }
      }
    }

    &:hover .item-desc p:first-child {
      opacity: 0;
    }
    &:hover .item-desc .hover-desc {
      opacity: 1;
      transform: translateY(0);
    }

    .item-depth {
      height: 20px;
      background: linear-gradient(to bottom, transparent, rgba($accent-deep-blue, 0.2));
      margin: 12px -18px -20px -18px;
      border-radius: 0 0 30% 30%;
    }
  }

  /* 移动端适配 */
  @media (max-width: 920px) {
    .items-container {
      grid-template-columns: repeat(2, 1fr);
      gap: 16px;
    }
  }

  @media (max-width: 520px) {
    .items-container {
      grid-template-columns: 1fr;
    }
    .item-card .item-desc .hover-desc {
      position: static;
      opacity: 1;
      transform: none;
      margin-top: 6px;
    }
    .item-card .item-desc p:first-child {
      opacity: 1;
    }
  }
}

  /* 彩蛋板块 */
  .easter-eggs .eggs-container {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 24px;
    .egg-item {
      background: rgba($deep-bg, 0.5);
      border: 1px solid $glass-edge;
      border-radius: 40px 40px 30px 30px;
      padding: 20px;
      .egg-time {
        display: inline-block;
        background: $accent-deep-blue;
        border: 1px solid $accent-aqua;
        border-radius: 60px;
        padding: 4px 12px;
        margin-bottom: 12px;
        font-size: 0.9rem;
        color: $accent-aqua;
      }
      h4 {
        color: $accent-lavender;
        margin: 0 0 8px;
      }
      p {
        margin: 0;
        line-height: 1.6;
      }
    }
    .egg-note {
      grid-column: span 2;
      text-align: center;
      font-size: 0.9rem;
      opacity: 0.7;
      font-style: italic;
    }
  }

  /* 共鸣链 */
  .timeline {
    .tl-wrap {
      position: relative;
      padding: 20px 0 20px 60px;
    }
    .vl {
      position: absolute;
      left: 30px;
      top: 20px;
      bottom: 20px;
      width: 2px;
      background: linear-gradient(180deg, $accent-aqua, $accent-lavender);
      border-radius: 2px;
      box-shadow: 0 0 15px $accent-aqua;
    }
    .tl-list {
      list-style: none;
      padding: 0;
      .tl-item {
        margin-bottom: 24px;
        .tl-card {
          background: rgba($deep-bg, 0.5);
          border: 1px solid $glass-edge;
          border-radius: 40px 40px 30px 30px;
          padding: 20px;
          transition: all 0.3s;
          &:hover {
            transform: translateX(10px);
          }
          .tl-title {
            color: $accent-lavender;
            margin: 0 0 8px;
            font-size: 1.1rem;
          }
          .tl-desc {
            margin: 0;
            line-height: 1.6;
          }
        }
      }
    }
  }

  /* 故事板块 */
  .ll-stories {
    .card-body {
      min-height: 300px;
      padding: 20px 0;
    }
    .story-head {
      color: $accent-aqua;
      font-size: 1.4rem;
      margin: 0 0 16px;
    }
    .story-content {
      line-height: 1.8;
      white-space: pre-wrap;
    }
    .card-foot {
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 20px;
      margin-top: 20px;
      .nav-btn {
        background: transparent;
        border: 1px solid $glass-edge;
        border-radius: 40px;
        padding: 8px 20px;
        color: $text-light;
        cursor: pointer;
        transition: all 0.2s;
        &:hover {
          border-color: $accent-aqua;
          background: rgba($accent-aqua, 0.1);
        }
      }
      .pager {
        color: $accent-aqua;
        font-size: 0.9rem;
      }
    }
  }

  /* 页脚 */
  .page-footer {
    margin-top: 40px;
    text-align: center;
    .footer-tentacles {
      display: flex;
      justify-content: center;
      gap: 16px;
      margin-bottom: 16px;
      span {
        width: 3px;
        height: 40px;
        background: linear-gradient(to top, $accent-aqua, transparent);
        animation: tentacleSway 3s infinite alternate;
        &:nth-child(1) {
          height: 50px;
          animation-delay: 0s;
        }
        &:nth-child(2) {
          height: 35px;
          animation-delay: 0.5s;
          background: $accent-lavender;
        }
        &:nth-child(3) {
          height: 60px;
          animation-delay: 0.2s;
        }
        &:nth-child(4) {
          height: 45px;
          animation-delay: 0.7s;
          background: $accent-pink;
        }
        &:nth-child(5) {
          height: 30px;
          animation-delay: 0.9s;
        }
      }
    }
    .footer-quote {
      font-style: italic;
      color: $accent-lavender;
      opacity: 0.8;
    }
  }

  /* 动画 */
  @keyframes float {
    0% {
      transform: translate(0, 0) scale(1);
      opacity: 0.2;
    }
    50% {
      transform: translate(-20px, -30px) scale(1.2);
      opacity: 0.5;
    }
    100% {
      transform: translate(20px, 20px) scale(0.9);
      opacity: 0.2;
    }
  }
  @keyframes tentacleSway {
    0% {
      transform: translateY(0) scaleY(1);
      opacity: 0.4;
    }
    100% {
      transform: translateY(-15px) scaleY(1.3);
      opacity: 0.9;
    }
  }
  @keyframes pulse {
    0% {
      opacity: 0.6;
      transform: scale(1);
    }
    50% {
      opacity: 1;
      transform: scale(1.3);
    }
    100% {
      opacity: 0.6;
      transform: scale(1);
    }
  }
  .fade-slide-enter-active,
  .fade-slide-leave-active {
    transition: all 0.4s ease;
  }
  .fade-slide-enter-from {
    opacity: 0;
    transform: translateY(20px);
  }
  .fade-slide-leave-to {
    opacity: 0;
    transform: translateY(-20px);
  }

  /* 移动端 */
  @media (max-width: 720px) {
    padding: 60px 12px 40px;

    .hero {
      grid-template-columns: 1fr;
      gap: 16px;
    }
    .two-col {
      grid-template-columns: 1fr;
    }

    .easter-eggs .eggs-container {
      grid-template-columns: 1fr;
      .egg-note {
        grid-column: span 1;
      }
    }
    .coral-tentacles {
      display: none;
    }
    .timeline-section .timeline-item {
      padding-left: 50px;
    }
  }
}
</style>
