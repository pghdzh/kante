<template>
  <div class="megumi-page" aria-live="polite">
    <main class="container" ref="mainContainer">
      <!-- HERO 区域 -->
      <header class="hero" role="banner">
        <div class="left-col">
          <button class="avatar-card">
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
            </div>
          </button>
        </div>
        <div class="right-col" ref="metaCard">
          <div class="hero-panel">
            <h1 class="name">
              坎特蕾拉 <span class="jp">（翡萨烈现任家主）</span>
            </h1>
            <p class="subtitle">于深海梦境中编织旋律的幻惑歌者</p>
            <p class="lead">
              神秘、高贵、美艳、剧毒……她居于山巅的冠冕，而山崖间流淌着的，都
              是由她编织的幻梦。
            </p>
          </div>
        </div>
      </header>

      <!-- 内容卡片区 -->
      <section id="personality" class="card personality">
        <div class="card-head">
          <h2>核心机制</h2>
        </div>
        <ul class="traits">
          <li>
            【迷离】：血量上方上限5点的特殊能量条（最上方）。拥有【迷离】时，Z进入蜃境状态，A消耗【迷离】获取【颤栗】。
          </li>
          <li>
            【迷离】获取方式：变奏技能，普通状态下E，R，第三段普攻命中，水中。
          </li>
          <li>
            【颤栗】：血量上方上限3点的特殊能量点（中间）。拥有3点【颤栗】时，E施放【感知汲取】，让敌人进入迷梦。
          </li>
          <li>【颤栗】获取方式：蜃境状态下普攻命中。</li>
          <li>
            【迷梦】：降低目标的行动速度，持续6.5秒。目标受到伤害时，将触发一次惊醒，并清除迷梦。队友伤害无法触发惊醒，并清除迷梦。
          </li>
          <li>【迷梦】触发方式：蜃境状态下的普通E和强化E。</li>
          <li style="font-weight: bold">输出流程：</li>
          <li>基础流程：AAA-EQ-R-ZE-AAA-E【感知汲取】-AAA-E</li>
          <li>
            技能缩写：普攻 = A，重击 = Z，共鸣技能 = E，共鸣解放 = R，声骸 = Q
          </li>
        </ul>
      </section>

      <section id="overview" class="card overview">
        <div class="card-head">
          <h2>角色背景与形象</h2>
        </div>
        <div class="two-col">
          <div class="text-block">
            <h3>背景</h3>
            <p>
              翡萨烈家族是黎那汐塔最为古老的家族之一，和隐海修会来往密切。世人对翡萨烈的固有印象，是他们高贵而神秘，且善于使毒弄权，但鲜少有人知道翡萨烈的真正目的。作为现任翡萨烈家主的坎特蕾拉，长居于一座伫立于悬崖之上的孤寂幽暗的古堡——波蒂维诺堡。飞瀑穿流，毒花环绕，淡紫色的雾气氤氲在朦胧的夜色中，为状如冠冕的古堡蒙上一层诡秘的面纱
              。传闻，她时常在古堡中独舞，舞步所及之处的每一寸空气，似乎都弥漫着令人沉溺的温暖
              。
            </p>
          </div>
          <div class="text-block">
            <h3>外貌／着装</h3>
            <p>
              坎特蕾拉有着一头紫丝绸般的光泽长发，如瀑布一样散落在脑后。蓝紫色的眼睛里氤氲着朦胧的雾气，使她看起来有一种似梦非梦的神采
              ，眼眸深不见底，眼中似有幽海翻涌，无人能看透她那幽深似海的眼眸。身材曼妙优美，胸口有一颗深蓝色的蓝宝石吊坠
              。
              湛蓝的宝石跃然装点于坎特蕾拉的衣裙、洋伞、饰物之上，将她衬托得更加优雅神秘。如碧海浪涛般深邃的蓝宝石，是独属于翡萨烈家主的证明。身着的衣裙上流动着绚丽的色彩，这些幻彩随着光线不同，会折射出不同的色泽，就像蛰伏深海的剧毒之物，美丽、危险、迷幻
              。
            </p>
          </div>
        </div>
      </section>

      <section id="resonance-status" class="card overview">
        <div class="card-head">
          <h2>共鸣状况</h2>
        </div>
        <div class="two-col">
          <div class="text-block">
            <p>
              坎特蕾拉代表着翡萨烈家族的最高毒药异能，毒药异能是翡萨烈家族中极为少见的幻觉毒，区别于一般翡萨烈成员常见的致死毒，幻觉毒并不会立刻致死。当水母群从坎特蕾拉的伞中游出，她的身边会形成一片美轮美奂的幻影之海。中毒之人先是会看到幻象——可能是内心最深处的渴望，抑或是坎特蕾拉编织的谎言。若是沉沦幻海太久，先是麻痹，进而昏厥，最严重时，才会致命。
              坎特蕾拉对自身异能状况把握得非常清晰，她的毒药异能炉火纯青，最精妙时，可使人看到幻觉，而不造成任何实质损伤。翡萨烈家族的毒药异能是一把双刃剑，发动异能时往往自己也会受到一定损伤，但从坎特蕾拉一贯优雅得体的行为表现来看，貌似并没有受到“幻梦”太大的影响。
            </p>
          </div>
        </div>
      </section>

      <!-- 新增三大主题区域 -->
      <section id="food" class="card special-section">
        <div class="card-head">
          <h2>「珊瑚骨」洋伞</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/item1.png" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>坎特蕾拉随身携带的洋伞。</p>
            <p>
              伞盖形状圆润，宛如一只软嘟嘟的水母，长长的触手悠闲地随风飘荡。而支撑起“水母”的伞骨则由深海珊瑚打磨而成，弧度优美、光泽温润、材质坚硬无比。握柄处镶嵌着一颗蓝宝石，象征着翡萨烈家主威严的权力。
            </p>
            <p>
              坎特蕾拉不推崇暴力，但若是不得不动手……她会转动洋伞。霎时，伞面流光溢彩，美轮美奂的水母群从伞下涌出，翩飞成一片幻影之海。
              让敌人自愿坠入美丽危险的幻海——这是现任翡萨烈家主坎特蕾拉的立世之道。而这柄洋伞和它的主人一样，看似温柔似水，实则坚不可摧。
            </p>
          </div>
        </div>
      </section>
      <section id="shattered-crystal" class="card special-section">
        <div class="card-head">
          <h2>「美梦」坩埚</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/item2.png" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>
              「海中女巫」坎特蕾拉用于熬制秘药的坩埚。
              瓶身镶嵌着一枚深邃的蓝宝石，这是属于坎特蕾拉的凭证。
            </p>
            <p>
              她经常会由着性子，随手丢进意想不到的“药材”——水母折断的触须，鱼儿脱落的鳞片，伤心之人的泪水，甚至还有萦绕耳畔的低语……
            </p>
            <p>
              不管她丢进什么，最后都能熬出一锅美梦。梦中的甘甜化开现实的咸苦，轻柔地安抚着身陷深渊的人们。
            </p>
          </div>
        </div>
      </section>

      <section id="cyclic-clock" class="card special-section">
        <div class="card-head">
          <h2>秘药瓶</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/item3.png" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>
              坎特蕾拉从不离身的秘药瓶。瓶身镶嵌着一枚深邃的蓝宝石，和她佩戴的饰品一样，都是翡萨烈家主的证明。瓶内存放着她精心调配的秘药，用于驱逐身体的疲惫，或者平息惊扰的梦魇。
            </p>
            <p>不过，自从你将她带离幽海，她已经很久没有梦魇了。</p>
            <p>
              如今，镶嵌着蓝宝石的秘药瓶中盛放着坎特蕾拉难得的真心与柔情——微光穿透深海，映照出隐现于汹涌浪潮之间的温柔呢喃……她将自己亲手托付给涉过暗潮、抵达海底的你。
            </p>
          </div>
        </div>
      </section>

      <section id="collective-album" class="card special-section">
        <div class="card-head">
          <h2>秘制帕尼尼</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/food.png" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>坎特蕾拉亲手制作的帕尼尼。</p>
            <p>烘烤得恰到好处的酥脆面包，塞满精心调味的海鲜内馅——</p>
            <p>一口下去，丰润的滋味顺着咽喉滑进肠胃，满足感填满腹腔。</p>
            <p>和秘药的玄妙不同，是一种简单温暖的味道。</p>
          </div>
        </div>
      </section>

      <section
        id="timeline"
        class="card timeline"
        aria-labelledby="timeline-heading"
      >
        <div class="card-head">
          <h2 id="timeline-heading">共鸣链详解</h2>
        </div>
        <div class="tl-wrap" role="list">
          <div class="vl" aria-hidden="true"></div>
          <ol class="tl-list">
            <li
              v-for="(ev, i) in timeline"
              :key="i"
              class="tl-item"
              role="listitem"
              tabindex="0"
              :aria-label="``"
            >
              <div class="tl-card">
                <h3 class="tl-title">第 {{ i + 1 }} 链 · {{ ev.title }}</h3>
                <p class="tl-desc">{{ ev.desc }}</p>
              </div>
            </li>
          </ol>
        </div>
      </section>

      <section class="ll-stories" ref="storyRef">
        <div class="card" role="region" aria-labelledby="stories-title">
          <header class="card-head">
            <h2 id="stories-title" class="card-title">坎特蕾拉纪事</h2>
          </header>

          <main class="card-body">
            <transition name="fade-slide" mode="out-in">
              <article
                :key="items[current].id"
                :id="`panel-${items[current].id}`"
                class="story"
                role="tabpanel"
                :aria-labelledby="`tab-${items[current].id}`"
              >
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
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

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
    desc: `普攻蛰幻的伤害倍率提升80%。施放共鸣解放陷溺时，坎特蕾拉的伤害无视目标30%防御，持续10秒。
迷梦的前1.2秒，目标受到伤害时，若此次伤害没有附加迷梦，则目标不会触发惊醒。`,
  },
]);

const reducedMotion =
  window.matchMedia &&
  window.matchMedia("(prefers-reduced-motion: reduce)").matches;

const items = [
  {
    id: "1",
    title: "圆桌会议",
    content: `
   光线穿过彩窗，在猩红的沙发上投下一片星星点点的斑驳。

雾紫的房间内点了几盏昏黄的小灯，圆桌边坐着几个模糊的人影，有的施施然坐在光明处，有的悠悠然浸没在黑暗里。

领头的那位发话了——至少从座次看是这样没错。这位女士有着一头紫丝绸般的光泽长发，蓝紫色的眼睛里氤氲着朦胧的雾气，使她看起来有一种似梦非梦的神采。她慵懒地靠着沙发，漫不经心地拨弄着胸口的蓝宝石吊坠——每当她陷入纠结的境地，总是会不自觉这么做。

“今日召集大家前来，是想一起探讨一下，究竟应该停留在何处？「幻梦」，抑或现实？”

“我们沿着梦的边界遨游，是留在虚幻的此岸，还是向前一步，游向现实的彼岸？幻梦中有我们熟悉的一切，虚像绚烂而轻盈，如同泡沫一般。而去往现实的路上，或许会遭遇意想不到的危险……但也可能看到从未见过的风景。”

“要我说，你还是太软弱了，坎特蕾拉。哦不对，现在应该叫你家主大人了，虽然在我看来，你还远远算不上称职。”黑暗里传来慵懒而笃定的声音，同样瀑布般的长发，同样深蓝的吊坠，同样曼妙的身段——只是这位女士似乎更加年长，更加强势，她深紫色的眼眸闪烁着不容质疑的光芒，“有什么好选的，直接把你的「紫绒梦」倒进宅子后面的瀑布，让整个拉古那都陷入由你掌控的幻梦——这样，我们就可以去任何地方，任何地方都属于我们。”

“无效回答。哦不对，硬要说的话，你是投了「幻梦」一票。好的，虽然听起来有些癫狂且不切实际，但这毕竟是讨论，为了公平公正起见，我还是记上了。”一旁端坐着的少女拿起笔记本——上面布满了奇奇怪怪的涂画，她好不容易找了个被秘药配方挤得几乎快看不见的空隙，煞有介事地划上一横。“我弃权，因为这样的讨论不见得有什么意义。我想，你大概已经有了答案……而对我来说，哪里都一样。”少女合上笔记本，抖了抖洁白的披肩，光线照到她碧蓝的眼眸中，若暗若明。

“为什么要这么复杂呢？遵从本心不好吗？坎特蕾拉姐姐，你很喜欢看窗前的飞鸟，也很喜欢外面自由的生活，那就，遵从你的本心就可以了呀。”稚声稚气的小女孩一整个趴在沙发上，用手去抓光线里飞舞的灰尘。

“这是一个慎重的决定，请大家严肃讨论。”坎特蕾拉似乎有些无奈，“就从你开始，重新好好说一下自己的选择，以及相应的理由。” `,
  },
  {
    id: "2",
    title: "荆棘冠",
    content: `
   深紫色的、淬毒的冠冕，刚戴上的时候会有些刺痛，但额头沁出的鲜血会将它滋润得更加鲜亮、更有光泽。

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

反正，「幻梦」就在血管里流淌，无论愿不愿意，都无法从身体中剥离。 `,
  },

  {
    id: "3",
    title: "水母灯",
    content: `
   昏暗、气味奇特的房间里，一盏微弱、昏黄的小灯，照着一本陈旧的笔记本，还有一口古怪的坩埚。

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
    content: `
    从有记忆起，我就一直被养在又大、又空旷的宅邸里。我从屋子的一头走到另一头，地板上的每一块瓷砖、架子上的每一个药瓶，我都很熟悉。整个屋子都灰扑扑的，没什么劲，只有小鸟飞过窗前的时候，我顺着洁白的、上下扑棱的翅膀看去——小小的圆点在天空中越飞越远，远处的天空蓝澄澄的，林子绿油油的，很好看。

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

那……投票的话，我投现实一票，也叫做「梦的彼岸」，对吧？大人总是喜欢这种玄乎的说法。但，你知道内心最渴望的是什么吧？不仅在梦中，也可以在现实里推开门扉，和大家一起跟着洁白的飞鸟四处转转，去寻找让所有人都幸福的结局。 `,
  },

  {
    id: "5",
    title: "清明梦",
    content: `
   庄严魅惑的声音，清冷坚定的声音，纯真童稚的声音，在雾紫的房间内穿梭交叠，形成梦幻般的声浪。

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

她抬起蓝紫色的眼睛，朦胧的雾气已经散去，平静海面般的眼眸泛起涟漪。
    `,
  },
];
const current = ref(0);

function select(index) {
  current.value = index;
  // 将焦点移回当前 tab（便于无障碍与键盘导航）
  const id = `tab-${items[index].id}`;
  const el = document.getElementById(id);
  if (el) el.focus();
}
const storyRef = ref<HTMLElement | null>(null);

function prev() {
  current.value = (current.value - 1 + items.length) % items.length;
  select(current.value);
  storyRef.value.scrollIntoView({ behavior: "smooth", block: "start" });
}

function next() {
  current.value = (current.value + 1) % items.length;
  select(current.value);
  storyRef.value.scrollIntoView({ behavior: "smooth", block: "start" });
}

onMounted(() => {
  if (!reducedMotion) {
    const tl = gsap.timeline({ defaults: { ease: "power3.out" } });
    tl.from(".avatar-card", {
      duration: 0.8,
      y: 18,
      opacity: 0,
      scale: 0.99,
    }).from(
      ".right-col > .hero-panel .name",
      { duration: 0.9, x: 24, opacity: 0 },
      "-=0.45"
    );

    gsap.utils.toArray(".tl-item").forEach((it: any, idx: number) => {
      gsap.from(it.querySelector(".tl-card"), {
        scrollTrigger: {
          trigger: it,
          start: "top 85%",
          toggleActions: "play none none reverse",
        },
        opacity: 0,
        y: 22,
        duration: 0.7,
        delay: idx * 0.06,
      });
    });

    gsap.to(".avatar-card", {
      scale: 1.007,
      duration: 2.8,
      yoyo: true,
      repeat: -1,
      ease: "sine.inOut",
    });
  } else {
    document.documentElement.classList.add("reduced-motion");
  }
});
</script>

<style scoped lang="scss">
/* 坎特蕾拉风格 —— 紫蓝海感（水母/幻海） + 毒药/治愈双重高光 */
$bg-deep: #04060b; // 深海夜色
$deep-2: #081022; // 次深底
$accent-1: #6f5ce6; // 暗紫主光
$accent-2: #5fe0ff; // 冷海蓝高光（湿光）
$venom: #ff7aa3; // 毒色点缀（保留少量）
$heal-glow: rgba(95, 224, 255, 0.12);
$text-main: #eae9ee;
$text-muted: rgba(234, 233, 238, 0.78);
$card-bg: rgba(6, 8, 12, 0.46);
$card-border: rgba(184, 107, 224, 0.06);
$soft-shadow: rgba(0, 0, 0, 0.6);
$inner-glow: rgba(184, 107, 224, 0.03);

.megumi-page {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(180deg, $bg-deep 0%, $deep-2 72%);
  font-family: "Noto Sans SC", system-ui, -apple-system, "Segoe UI", Roboto,
    Arial;
  color: $text-main;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  padding: clamp(20px, 4vw, 80px);
  box-sizing: border-box;
  padding-top: 60px;

  &.reduced-motion * {
    transition: none !important;
    animation: none !important;
  }

  .container {
    position: relative;
    z-index: 10;
    max-width: 1100px;
    margin: 0 auto;
    padding: clamp(12px, 2vw, 40px);
  }
}

/* ========== HERO 区域（整体暗化 + 血玫点光） ========== */
.hero {
  position: relative;
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 28px;
  align-items: start;
  padding: 12px;
  margin-bottom: 12px;

  /* 乐谱画布 —— 在 hero 背后增加五线谱移动感 */
  &::before {
    content: "";
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;
    background-image: repeating-linear-gradient(
      to bottom,
      rgba(255, 255, 255, 0.02) 0px,
      rgba(255, 255, 255, 0.02) 1px,
      transparent 1px,
      transparent 16px
    );
    opacity: 0.06;
    mix-blend-mode: overlay;
    transform: translateY(-4%);
    animation: staff-drift 16s linear infinite;
  }

  .left-col {
    display: flex;
    flex-direction: column;
    align-items: center;
    z-index: 6;
  }

  .avatar-card {
    background: linear-gradient(
      180deg,
      rgba(255, 255, 255, 0.01),
      rgba(184, 107, 224, 0.01)
    );
    border-radius: 16px;
    padding: 12px;
    display: grid;
    justify-items: center;
    gap: 8px;
    border: 1px solid $card-border;
    box-shadow: 0 14px 44px $soft-shadow, inset 0 1px 0 $inner-glow;
    cursor: pointer;
    transition: transform 0.28s ease, box-shadow 0.28s ease;
    width: 100%;


  }

  .avatar-wrap {
    width: 100%;
    display: block;
  }

  .avatar {
    width: 100%;
    height: auto;
    border-radius: 12px;
    display: block;
    box-shadow: 0 12px 34px rgba(0, 0, 0, 0.5), 0 0 18px rgba($accent-1, 0.03);
    transition: transform 0.5s ease, filter 0.5s ease;
    background-color: rgba(0, 0, 0, 0.04);
    filter: saturate(0.8) contrast(0.95) brightness(0.92);
  }

  .avatar-card:hover .avatar {
    transform: translateY(-4px) scale(1.02);
    filter: saturate(0.7) contrast(0.9) brightness(0.9);
  }

  .avatar-meta {
    display: flex;
    gap: 8px;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
  }

  .avatar-badge {
    font-size: 0.9rem;
    color: $text-main;
    background: rgba(0, 0, 0, 0.18);
    padding: 6px 10px;
    border-radius: 999px;
    border: 1px solid rgba($accent-2, 0.05);
    font-weight: 600;
  }

  .right-col {
    display: flex;
    flex-direction: column;
    gap: 12px;
    z-index: 6;
  }

  .hero-panel {
    position: relative;
    z-index: 6;
    padding: 20px;
    border-radius: 14px;
    background: $card-bg;
    backdrop-filter: blur(8px) saturate(0.9);
    border: 1px solid $card-border;
    box-shadow: 0 12px 50px rgba(0, 0, 0, 0.6), inset 0 1px 0 $inner-glow;
  }

  .name {
    font-size: clamp(1.8rem, 3vw, 2.8rem);
    margin: 0;
    font-weight: 900;
    letter-spacing: 1px;
    line-height: 1;
    display: inline-block;
    /* 标题改为 暗紫 → 血玫 渐变 */
    background: linear-gradient(90deg, $accent-1 0%, $accent-2 70%);
    -webkit-background-clip: text;
    background-clip: text;
    -webkit-text-fill-color: transparent;
    color: transparent;
    -webkit-text-stroke: 0.2px rgba(0, 0, 0, 0.22);
    text-shadow: 0 10px 36px rgba(0, 0, 0, 0.65);
    position: relative;
  }

  .jp {
    font-size: 0.95rem;
    color: $accent-1;
    margin-left: 8px;
    font-weight: 400;
  }

  .subtitle {
    margin: 6px 0 0;
    font-size: 1.1rem;
    color: $accent-1;
    font-weight: 600;
    line-height: 1.4;
  }

  .lead {
    margin: 10px 0 0;
    color: $text-main;
    opacity: 0.96;
    line-height: 1.7;
    max-width: 72ch;
  }
}

/* ========== 通用卡片（更暗、更锐利的边与血色提示） ========== */
.card {
  margin-top: 24px;
  background: rgba(6, 6, 8, 0.5);
  border-radius: 14px;
  padding: 24px;
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(6px);
  color: $text-main;
  border: 1px solid $card-border;
  transition: transform 0.28s ease, box-shadow 0.28s ease;

  &:hover {
    transform: translateY(-8px) scale(1.01);
    box-shadow: 0 30px 90px rgba(0, 0, 0, 0.7);
  }

  h2 {
    margin: 0 0 12px 0;
    font-size: 1.3rem;
    color: $accent-2;
  }

  .card-head {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
  }
}

/* personality 列表样式（暗紫细线） */
.personality .traits li {
  padding: 12px 0;
  border-bottom: 1px dashed rgba($accent-1, 0.06);
  font-size: 1rem;
  color: rgba($text-main, 0.92);
}

/* overview 双栏布局（保持） */
.overview .two-col {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
  margin-top: 12px;
}

.overview .text-block {
  flex: 1 1 40%;
  min-width: 240px;
  line-height: 1.6;
  color: rgba($text-main, 0.96);
}

/* special-section：加入音符漂浮和暗紫边框 */
.special-section {
  position: relative;
  overflow: hidden;

  .content {
    display: flex;
    gap: 24px;
    align-items: center;
    margin-top: 16px;
  }

  .img-area {
    flex: 0 0 200px;

    img {
      width: 100%;
      height: auto;
      border-radius: 12px;
      box-shadow: 0 14px 44px rgba(0, 0, 0, 0.6), 0 0 12px rgba($accent-1, 0.02);
      border: 1px solid rgba($accent-1, 0.03);
    }
  }

  .text-block {
    flex: 1;
    line-height: 1.6;
    color: rgba($text-main, 0.95);
  }
}

/* ll-stories — 暗底 + 微血色分割线 */
.ll-stories {
  .card {
    width: min(1200px, 100%);
    border-radius: 12px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.6),
      inset 0 1px 0 rgba($accent-1, 0.02);
    color: $text-main;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    background: linear-gradient(
      180deg,
      rgba(8, 10, 12, 0.12),
      rgba(8, 6, 10, 0.06)
    );
  }

  .card-head {
    padding: 14px 16px;
    border-bottom: 1px solid rgba($accent-2, 0.03);
    display: flex;
    flex-direction: column;
    gap: 12px;

    .card-title {
      font-size: 24px;
      font-weight: 700;
      color: $accent-1;
      letter-spacing: 0.6px;
    }
  }

  .card-body {
    padding: 18px 16px;
    min-height: 160px;
    position: relative;
    background: linear-gradient(180deg, rgba(0, 0, 0, 0), rgba(8, 8, 10, 0.02));
  }

  .story {
    color: rgba($text-main, 0.96);

    .story-head {
      font-size: 18px;
      color: $accent-2;
      font-weight: 700;
      margin-bottom: 8px;
    }

    .story-content {
      line-height: 1.7;
      color: rgba($text-main, 0.92);
      white-space: pre-wrap;
    }
  }

  .card-foot {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
    padding: 10px 14px;
    border-top: 1px solid rgba($accent-1, 0.02);
    background: linear-gradient(
      180deg,
      rgba(0, 0, 0, 0.02),
      rgba(0, 0, 0, 0.04)
    );

    .nav-btn {
      background: transparent;
      border: 1px solid rgba($accent-1, 0.06);
      color: $accent-2;
      padding: 6px 10px;
      border-radius: 6px;
      cursor: pointer;
      transition: background 180ms ease;
      &:hover {
        background: rgba($accent-2, 0.04);
      }
    }

    .pager {
      color: rgba($text-main, 0.72);
      font-size: 13px;
      font-weight: 600;
    }
  }

  /* 切换动画保留，但更暗 */
  .fade-slide-enter-active,
  .fade-slide-leave-active {
    transition: all 320ms cubic-bezier(0.2, 0.9, 0.2, 1);
  }

  .fade-slide-enter-from {
    opacity: 0;
    transform: translateY(6px);
  }

  .fade-slide-enter-to {
    opacity: 1;
    transform: translateY(0);
  }

  .fade-slide-leave-from {
    opacity: 1;
    transform: translateY(0);
  }

  .fade-slide-leave-to {
    opacity: 0;
    transform: translateY(-6px);
  }
}

/* timeline（暗紫竖线 + 微光） */
.timeline {
  position: relative;

  .tl-wrap {
    position: relative;
    padding: 24px 12px 12px 96px;
  }

  .vl {
    position: absolute;
    left: 80px;
    top: 32px;
    bottom: 20px;
    width: 4px;
    border-radius: 6px;
    background: linear-gradient(
      180deg,
      rgba($accent-2, 0.12),
      rgba($accent-1, 0.1)
    );
    box-shadow: 0 6px 18px rgba($accent-1, 0.06),
      inset 0 0 10px rgba($accent-1, 0.02);
    opacity: 0.95;
  }

  .tl-list {
    list-style: none;
    margin: 0;
    padding: 0;
  }

  .tl-item {
    position: relative;
    display: flex;
    gap: 18px;
    align-items: flex-start;
    margin: 20px 0;
    transition: transform 0.28s ease, box-shadow 0.28s ease;
  }

  .tl-card {
    background: rgba(8, 6, 10, 0.36);
    border-radius: 12px;
    padding: 16px 20px;
    box-shadow: 0 12px 36px rgba(0, 0, 0, 0.55);
    width: calc(100% - 130px);
    border-left: 4px solid transparent;
    transition: transform 0.28s ease, box-shadow 0.28s ease,
      border-color 0.28s ease;
    position: relative;
    z-index: 2;
  }

  .tl-item:hover .tl-card,
  .tl-item:focus-within .tl-card {
    transform: translateY(-8px);
    box-shadow: 0 24px 72px rgba(0, 0, 0, 0.6);
    border-left-color: rgba($accent-2, 0.18);
  }

  .tl-title {
    margin: 0 0 6px 0;
    font-size: 1.1rem;
    color: $accent-1;
    display: flex;
    align-items: baseline;
    gap: 8px;
  }

  .tl-desc {
    margin: 0;
    color: rgba($text-main, 0.92);
    white-space: pre-wrap;
  }
}

/* 浮动音符（可在模板中渲染若干 .floating-note） */
.floating-note {
  position: absolute;
  font-size: 14px;
  color: $accent-2;
  opacity: 0.95;
  transform-origin: center;
  animation: note-float 4.6s ease-in-out infinite;
  filter: drop-shadow(0 6px 18px rgba($accent-2, 0.06));
}

/* 动画关键帧 */
@keyframes staff-drift {
  0% {
    transform: translateY(-4%);
    opacity: 0.9;
  }
  50% {
    transform: translateY(6%);
    opacity: 0.95;
  }
  100% {
    transform: translateY(-4%);
    opacity: 0.9;
  }
}

@keyframes note-float {
  0% {
    transform: translateY(0) rotate(-6deg) scale(0.95);
    opacity: 0.85;
  }
  50% {
    transform: translateY(-12px) rotate(3deg) scale(1);
    opacity: 1;
  }
  100% {
    transform: translateY(0) rotate(-6deg) scale(0.95);
    opacity: 0.85;
  }
}

/* 响应式：小屏适配（保留原逻辑，仅小幅调整文字色调与阴影） */
@media (max-width: 980px) {
  .hero {
    grid-template-columns: 1fr;
    gap: 18px;
    padding: 8px;
  }

  .avatar-card {
    width: 140px;
    padding: 8px;
  }

  .hero-panel {
    padding: 14px;
  }

  .name {
    font-size: 1.8rem;
  }

  .subtitle {
    font-size: 1rem;
  }

  .lead {
    font-size: 0.95rem;
  }

  .card {
    padding: 18px;
    border-radius: 12px;
  }

  .overview .two-col {
    flex-direction: column;
  }

  .special-section .content {
    flex-direction: column;
  }

  .ll-stories {
    .card {
      border-radius: 10px;
    }

    .card-head {
      padding: 12px 12px;
    }

    .card-body {
      padding: 14px 12px;
      min-height: 180px;
    }

    .card-foot {
      padding: 8px 12px;
    }
  }

  .timeline {
    overflow: visible;
  }

  .timeline .tl-wrap {
    padding-left: 14px !important;
    padding-right: 14px;
    min-height: 0;
  }

  .timeline .vl {
    display: none !important;
  }

  .timeline .tl-list {
    overflow: visible;
  }

  .timeline .tl-item {
    display: block !important;
    margin: 14px 0 !important;
    width: 100%;
  }

  .timeline .tl-card {
    width: 100% !important;
    padding: 14px !important;
    border-left: 0 !important;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.45);
    transform: none !important;
  }
}
</style>
