# 019 · 五种经典的卡片滚动交互
**Five classic scroll-driven card interactions**

> 分类：交互 Interaction · 原帖：[2026-11-16](https://x.com/Kubai087/status/2122492414598459820) · Prompt：[作者回复](https://x.com/Kubai087/status/2122496665571803155) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对介绍帖和作者 Prompt 回复的改写整理，不是原文，也没有照抄原版 Prompt。尺寸、距离和公式取自介绍帖与作者回复；只在演示视频里的细节不补。介绍帖时间为 2026-11-17 00:50:00 UTC，Prompt 回复为 2026-11-17 01:06:53 UTC，按本仓库约定把介绍帖记为 America/Los_Angeles 的 2026-11-16。没有英文原帖。原帖附演示视频。「实现要点」代码为本仓库示例。

## 原理

人滚动页面时，预期看到的是上下移动。这里五种效果把滚动变成横移、旋转，或一条别的路径，用在产品官网和个人作品集上。苹果官网常把这类滚动和序列帧放在一起。序列帧怎么做，原文没写。

五种都由滚动位置直接驱动。停在一半，效果就停在一半。往回滚，沿原路倒放。

- **层叠卡片（Stacked cards）**：卡片随滚动一层层叠上。下面的变暗、缩小，每张顶上留一条，前后才分得开。
- **卡组飞出（Card deck）**：最上面一张沿弧线向左或向右飞走，下一张升上来。轨迹按抛物线来。
- **横向展廊（Pinned Gallery）**：这一段先钉住。纵向滚动变成整行横移，运动方向和滚动方向不同。
- **封面轮播（Cover flow）**：经典的 3D 封面流。焦点在中间，两侧略微侧转，视线收在当前这一张上。
- **图片展开（Image Expand）**：小图随滚动铺满屏幕。标题和背景前后遮挡，移动速度也不一样，于是有远近。

改的是产品里已经存在的长页面。内容、组件、token、文案都留下。

## 规则 / 数值

下面的数都是建议。Prompt 第 2 到第 8 节是测过的一套，叫 Standard。第 10 节是预设。用户选哪套，就做哪套。

演示画幅是 **1000 × 896**。那一帧上的像素不要原样套到别的窗口上。演示视频里多出来的画面，本文不写。

### 先看产品

动手之前用几行说清现状，然后再发问。

- 页面各段的顺序、内容、卡片、图片、视频。固定或粘住的东西（页头、横幅、cookie 条）在每个断点上的高度。
- 每一段适合哪种效果。被人扫读、对比、搜索的段落保持原样。
- 现在怎么适配：断点、用的是 vh / svh / dvh 还是脚本读高度、矮窗口上已经坏了什么。
- 滚动发生在窗口还是内部容器。有没有平滑滚动库、锚点、现成的滚动效果。
- 平台和版本，圆角和动效的 token，有没有暗色。

### 一次问一题

看完再问。每次只问一题，选项写成短列表，标出关键数值和推荐项，然后等回答。

1. **哪一段用哪种。** 一页里每种最多一次，中间夹普通段落。
2. **长度：** Short、Standard（推荐）、Long。数值见预设表。
3. **手机和矮屏：** Standard（推荐）、和桌面相同，或手机上全部改成普通段落。Standard 是：手机上层叠、飞出、图片展开留下，并按视口缩放；展廊和封面改成带吸附点的原生横滑；视口高度低于 560px 时，什么都不钉住。找得到统计或受众对应的屏幕尺寸，先说出来。
4. **减弱动效：** 普通段落（推荐：列表、横排、一张通栏图），或保留钉住、去掉运动。
5. **引擎：** CSS scroll-driven animations，加一小段脚本做回退（推荐）；或项目里已经在用的滚动库。不要平滑滚动，不要劫持滚动。

五题问完，用一张表汇总选择，写明在这个平台上怎么做、代价和回退，等用户点头再改。代价按要写的代码、兼容和回退来说。原文没给工时。

「手机」没有单独的宽度断点。不要自己补一个 768 之类的界限。

### 哪种内容用哪种

- **层叠卡片：** 3 到 5 个并列、各自值得一屏的条目。步骤、功能、方案、章节。
- **卡组飞出：** 3 到 6 个一次只看一条的条目。精选、评价、要点。
- **横向展廊：** 5 到 12 个比屏幕宽、用来浏览的条目。作品、产品、发布。
- **封面轮播：** 5 到 9 个同形、一次聚焦一张的条目。专辑、书、方案、人物。
- **图片展开：** 一张强图，或一段视频，用来打开下一章。

### 五条共用的规则

每一段一个进度：

`p = clamp((scrollY - start) / distance, 0, 1)`

每个属性都是 p 的函数。不要做一段由滚动「触发」、然后自己跑完的定时动画。停在一半就停在一半。往回滚，精确倒放。不改变滚动速度，不吸附页面，不截获滚轮。

钉住是位置，不是脚本锁死滚动：`position: sticky` 放在一段比屏幕更高的区域里。多出来的高度，就是效果播放的距离。

只动画 `transform`、`opacity`、`clip-path`、`filter`。字留在页面里，任何 p 都要能读。

钉住期间，用一根细条或一排圆点表示读到哪。不要文字标签。

### 层叠卡片（Stacked cards）

演示画幅 1000 × 896 上的 Standard：

- 卡片 **936 × 720**。12 栏全部占满，再向外出血 24px；内边距也是 24px，内容因此回到栏上。圆角 **28px**。卡片彼此相距 **48px**。
- 每张比上一张低 **16px** 粘住：`top = 40px + i * 16px`。有固定页头，把页头高度加进这个 top。新卡片盖住旧卡片，每张露出 **16px**。
- 后面的卡片到来时，这一张后退。`k = clamp((scrollY - 它粘住时的 scrollY) / 752px, 0, 它后面还有几张)`。缩放 `1 - k * 0.05`，原点在顶边中心。上面压着三张时是 **0.85**。暗层是 `k * 0.12`。
- 卡片内部：名字和一行说明只占第 0 到第 5 栏，在栏内换行，不挤进图里。名字距卡片顶 **48px**，说明在名字下 **24px**。
- 每张卡片自己的低饱和底色，加一张图。演示是 stone、sage、clay，再加一张墨色。色值原文没给，不要从视频里取。不要投影。
- 最后一张粘住之后再停 **320px**，然后整叠一起离开。离开时的位移原文没写。
- 可选开场：段标题正好铺满栅格，不要裁切。第一张卡片盖住标题的下沿。标题以滚动的一半速度移动，并在第一张粘住之前的 **280px** 里淡出。

卡片改高之后，卡与卡粘住的滚动间距跟着变。原文没把这条写成「卡片高 + 32」。Standard 在 720px 高的卡片上，这段距离是 **752px**。示例不另造一条换算。

### 卡组飞出（Card deck）

- 演示是五张叠在标题下面，**432 × 456**（居中的 6 栏）。每张里：图向内收 **8px**，名字和一行说明向内收 **24px**。不要别的内容。条目可以是 3 到 6 个。
- 段高是一屏再加 **1920px**，内容钉住。前四张各用 **420px** 飞出，然后停 **240px**。最后一张留下。420 × 4 + 240 = 1920。
- 第 i 张：`t = clamp((scrollY - start) / 420 - i, 0, 1)`。左右交替飞出：`x = side * t * t * 1000px`，`y = t * (160px * t - 40px)`，`rotate = side * t * 16deg`，绕底边中心。飞出后藏起来。往回滚，沿同一条路径回来。先向左还是先向右，原文没指定。
- `y` 先为负（抬起）再为正（落下）。t = 1 时 y = 120px。按公式，最高大约只抬起 2.5px。这是算出来的，不是另设的抬起距离。
- 下面的牌每一位低 **18px**、小 **0.05**，暗层每位 **0.04**，最多叠三层。最上面一张飞走时，其余各升一位。

### 横向展廊（Pinned Gallery）

- 一行 5 到 12 张。演示是八张，宽 **280**（4 栏），间距 **24px**。白卡片，图向内收 8px，下面是名字和作者。标题在行上方，距行 **48px**。行下一条 **2px** 的条：轨道 6% 黑，填充 30%。30% 没写是黑还是不透明度，不要补成色值。
- 段高是一屏再加溢出。溢出 = 行宽 − 视口宽。演示里这段溢出是 **1520px**，不要当成所有页面的常数。内容 `sticky`，演示写的是 top 0、高度一屏。有固定页头时，top 改成页头高度。
- `p = (scrollY - start) / overflow`，行 `translateX(-p * overflow)`。不要缓动。滚到底，到最后一项就松开。
- 钉住距离控制在大约三屏以内。更多条目改用网格。

### 封面轮播（Cover flow）

- 5 到 9 张正方形封面，边长 **432**（居中的 6 栏），放在标题下。下面是圆点。封面上不要字。
- 段高是一屏再加每步 **320px**，内容钉住。步数是张数减 1，最后一张停在中间。`r = clamp((scrollY - start) / 320, 0, count - 1)`。`c = floor(r) + smoothstep(fract(r))`。smoothstep 让每张在中间停一下。原文只写了 smoothstep 这个名字，没写多项式。
- 第 i 张，`d = i - c`：`translateX = d * 260px`，`rotateY = clamp(d, -1, 1) * -30deg`，`scale = max(0.6, 1 - |d| * 0.2)`，暗层 `min(|d|, 3) * 0.15`。离焦点越近，越在上面。透视 **1400px**。超过 **3.5** 个位置就隐藏。圆点按 `1 - |i - c|` 变亮。
- 封面的变换原点原文没写。

### 图片展开（Image Expand）

- 段高是一屏再加 **1200px**，内容钉住：前 **900px** 长大，后 **300px** 停住。
- 开始时，标题和相框一起居中。相框 **584 × 352**（第 2 到第 9 栏），在标题下 **48px**，圆角 **8px**。
- 图片元素始终盖住视口。`clip-path: inset()` 先露出相框，再开到 0，圆角同时收到 0。图片自身从 **1.2** 收到 **1**，所以看起来是在变大。两者都走 `e = ease(p)`，曲线 `cubic-bezier(0.45, 0, 0.25, 1)`。这里的 p 用在长大的那 900px 上。不要去动画宽高。
- 开场那个标题以两倍速度淡出，并上升 **60px**。原文没写这 60px 是不是也走两倍速度。
- 景深：把主体从图片里抠成单独一层。演示抠的是山，怎么抠原文没写，按素材来。另一个标题放在这层后面，下沿被形状挡住。一行小字放在最前。开场标题和形状后面的标题是两处字，不要合成同一个元素。
- 展开时，图和形状从 **1.2** 收到 **1**。形状后的标题从 **1.25** 起，前面那一行从 **1.4** 起。原文对这两层只写了起始缩放，终点写明收到 1 的是图片。铺满之后，图和形状停住；形状后的标题按滚动的 **0.25** 上升，前面那一行按 **0.5** 上升。一层压着一层，速度又不同，读起来才有远近。
- 下一节滑上来盖住它。顶上圆角 **28px**，在它最后 **120px** 里收成直角。同时图片压暗到 **35%** 黑。原文没写压暗是不是刚好发生在这 120px 里。
- 视频同样处理：静音、循环，只在屏幕上时播放。

### 屏幕与视口高度

演示尺寸服务于 1000 × 896。按屏幕缩放。不要写一个默认窗口很高的固定高度。

- 钉住段的视口高度用 **100svh**。手机上 100vh 是工具栏藏起来之后的高度，内容会钻到工具栏下面。钉住的内容不用 **dvh**，工具栏一动它就会跳。第 4 到第 7 节里的「100vh + 距离」是简写，落地时视口用 100svh，多出来的仍是滚动距离。
- 在加载、转向、宽度变化时重新测量。大约 **120px** 以内、只有高度在变（工具栏露出或藏起）的情况忽略。那时重算，卡片会在滚动中途跳。
- 每个钉住段都要放进一屏：
  - 层叠卡片高 = `min(720px, 100svh - sticky top - (张数 - 1) * 16px - 32px)`。卡与卡粘住的距离跟着这个高度走。
  - 卡组：标题 + 48px + 卡片放进 `100svh - 64px`。高度不够时，先减图片的高。
  - 展廊卡片高 = `min(520px, 100svh - 标题 - 48px - 96px)`。演示写了宽 280，没写高。
  - 封面边长 = `min(432px, 48svh, 70vw)`。间距随边长一起变。
  - 图片相框宽 = `min(584px, 100vw - 2 * 页边)`，比例 **5:3**。inset 从相框和视口的差算出来。Standard 的 584 × 352 和严格的 5:3 差一点点，不要把 352 改成别的数。
- 每段滚动距离乘 `clamp(0.7, 视口高 / 896, 1.2)`。乘的是滚动距离：粘住间距、飞出行程、停留、封面步长、展开的生长和停留。卡片的布局尺寸走上面的「放进一屏」，不要再乘一次。
- 视口高度低于 **560px**（手机横过来、或很矮的窗口）时，什么都不钉住。改成列表、横排、一张通栏图。
- 字号跟宽度走：段标题 `clamp(36px, 7vw, 64px)`，卡片名字 `clamp(28px, 5vw, 36px)`。
- 出血的瞬间遵守 `env(safe-area-inset-*)`。固定页头从每一次「放进一屏」里扣掉，并加进 sticky 的 top。层叠的 `top = 40px + i * 16px` 同样加上页头高度。

手机上的 Standard 还有一条，和上面的高度规则一起用：层叠、飞出、图片展开留下并按视口缩放；展廊和封面改成原生横向滚动，带吸附点。这组吸附只在那条横滑上。不要拿它去吸附页面滚动，也不要截获滚轮。

### 排版

- 先落在产品自己的栏上。演示按 12 栏：层叠卡占满 12 栏，卡组和封面是居中的 6 栏，展廊卡宽 4 栏，展开图框在第 2 到第 9 栏。产品栏数不同，就落到现有栅格上。每个文字盒和它所占的栏一样宽，长名字换行，不冲进图里。
- 标题引出的内容，从标题下 **48px** 开始。钉住段里，标题和内容一起在视口中居中。标题居中，内容就居中；标题左对齐，内容就左对齐。
- 卡片以外，每段只有一个短标题。不要标签、侧边文案、计数或说明文字。卡片以内只有一种结构：图、名字、一行说明。展廊那一行是名字和作者，封面本身不放字。
- 两种字体、四档字号。演示是 Newsreader 64 和 36、Geist 18 和 16。产品已有字体，就用产品的，保持四档。大号衬线按首字母的侧承向左收。大约超过 **100px** 时用光学字号，让衬线不要太尖。侧承的像素原文没给。
- 除了列表行之间，不要分割线。不要投影。强调色整页最多用一次。一组图只用一种图片样式。同一张图不要在一页里出现两次。
- 锚点落在段首。键盘滚动和滚轮一样。钉住段里的条目要能用 Tab 走到。

暗色沿用产品 token。展廊轨道那档 6% 黑是演示值，暗色下原文没另给。

### 预设

用户没改时用 Standard。某一档只替换它写明的数。

| 组 | 各档 |
|---|---|
| 长度 Length | Short：层叠 560px，飞出 300px，展廊每滚动 1px 横移 1.25px，封面步长 240px，图片 600 + 200px。Standard（推荐）：752、420、1:1、320、900 + 300。Long：900、560、1:1 且两端各停 200px、420、1200 + 400 |
| 层叠深浅 Stack depth | Flat：每张缩 0.03，暗 0.08。Standard：0.05、0.12。Deep：0.07、0.16，露出条 24px |
| 飞出 Deck throw | Gentle：8°、700px、一律向右。Standard：16°、1000px、左右交替。Lively：24°、1200px、抬起 60px |
| 封面 Cover flow | Flat：不转，两侧 0.85。Standard：30°、两侧 0.8。Deep：45°、两侧 0.75、间距 180px |
| 图框起点 Picture start | Small：420 × 280。Standard：584 × 352。Wide band：通栏，高 340 |

长度那一行替换的是滚动距离：层叠的后退距离（Standard 752px）、每张飞出的行程（Standard 420px）、展廊的横移比、封面步长、图片的生长和停留。层叠最后的 320px 停留、飞出后的 240px 停留，预设表没写，不要跟着长度档一起改。Lively 的 60px 抬起、Deep 的 24px 露出条，原文没给新的完整公式，只换它点名的那一项。

### 怎么做

先对照项目已有的技术栈和版本。

- **Web：** 钉住用 `position: sticky`。优先用 scroll-driven animations（`animation-timeline: view()` 或具名时间线，加上 `animation-range`）。不支持时，或关键帧不够时，改用脚本。卡组每张自己的区间、封面在中间的停顿，关键帧不够。脚本是一个 passive 的 scroll 监听：每帧读一次 scrollY，用 resize 时量好的起点和距离算出这一段的 p，只写到这一段自己的自定义属性上。
- 项目里已经有 GSAP ScrollTrigger（scrub、pin）或 Motion（useScroll、useTransform），就用那一套，数值仍是上面这些。不要为了这个效果新装一个库。不要平滑滚动，不要劫持滚动。
- **iOS（SwiftUI）：** 在滚动视图里用 `.scrollTransition` 或 `.visualEffect`（iOS 18 可用 `onScrollGeometryChange`），配合 `scaleEffect`、`rotation3DEffect`、`offset`、`clipShape`。展廊和封面用横向 `ScrollView`，`.scrollTargetBehavior(.viewAligned)`。
- **Android（Compose）：** 用 `LazyListState` 取 p，`Modifier.graphicsLayer` 做缩放、rotationY、透明度、位移和裁切。展廊用带吸附的 `LazyRow`，或 `HorizontalPager`。
- **Flutter：** `CustomScrollView` 加 sliver，`ScrollController` 取 p，`Transform` 和 `ClipRRect`。展廊和封面用 `PageView`。

iOS 上把展廊和封面写成横向滚动，和「手机上这两项改成原生横滑」是同一方向。Web 桌面的 Standard 仍是钉住之后把纵向滚动变成横移，除非用户选了别的手机方案。

### 做完核对

- 每一段都慢滚、快滚、往回滚。没有跳变、闪烁或空档。
- 这些视口都看过：375×667、390×844、430×932（工具栏露出和藏起）、844×390、768×1024、1280×720、1440×900、1920×1080，以及一个高 600px 的桌面窗口。没有被裁切，没有卡片高过屏幕，没有内容钻进工具栏或固定页头。
- 文字缩放到 200%、减弱动效、键盘、亮色和暗色。中端手机上保持满帧。做出来的数值和用户确认的那张表一致。

## 实现要点（示例实现，非原帖代码）

常量是 Standard。视口缩放、smoothstep 的多项式、左右的正负号、飞出后用透明度藏起、圆点用透明度变亮、封面原点放在中心、开场标题的淡出和 60px 分开算、压暗跟圆角不是同一段，都在注释里标了，并记在文末出处。

```css
.pin {
  /* 多出来的高度是效果播放的距离。视口用 100svh，不用 100vh / dvh。 */
  height: calc(100svh + var(--distance));
}
.pin-sticky[data-sticky] {
  position: sticky;
  top: var(--sticky-top, 0px); /* 有固定页头时写成页头高度 */
  height: 100svh;
  /* 飞出的牌和两侧封面要画出这一屏。这里不加 overflow: hidden。 */
}
```

```ts
export const SCROLL_CARDS = {
  demoWidth: 1000,
  demoHeight: 896,
  distanceScaleMin: 0.7,
  distanceScaleMax: 1.2,
  ignoreHeightResize: 120,
  noPinBelow: 560,
  stack: {
    cardW: 936,
    cardH: 720,
    bleed: 24,
    radius: 28,
    apart: 48,
    firstTop: 40,
    strip: 16,
    recede: 752,
    scaleStep: 0.05,
    shadeStep: 0.12,
    hold: 320,
    titleLead: 280,
    titleSpeed: 0.5,
    nameTop: 48,
    lineGap: 24,
  },
  deck: {
    cardW: 432,
    cardH: 456,
    pictureInset: 8,
    textInset: 24,
    throwPx: 420,
    hold: 240,
    sectionExtra: 1920,
    flyX: 1000,
    flyDeg: 16,
    pileDrop: 18,
    pileScale: 0.05,
    pileShade: 0.04,
    pileMax: 3,
  },
  gallery: {
    cardW: 280,
    gap: 24,
    titleGap: 48,
    bar: 2,
    trackBlack: 0.06,
    // 原文写 fill 30%，没写是黑还是不透明度。这里不补色值。
    demoOverflow: 1520,
  },
  cover: {
    size: 432,
    step: 320,
    spread: 260,
    turnDeg: 30,
    perspective: 1400,
    hideBeyond: 3.5,
  },
  image: {
    grow: 900,
    hold: 300,
    frameW: 584,
    frameH: 352,
    frameBelowTitle: 48,
    radius: 8,
    pictureFrom: 1.2,
    titleFrom: 1.25,
    lineFrom: 1.4,
    titleRise: 60,
    titleParallax: 0.25,
    lineParallax: 0.5,
    nextRadius: 28,
    nextFlatten: 120,
    darken: 0.35,
    ease: [0.45, 0, 0.25, 1] as const,
  },
} as const;

const clamp = (n: number, a: number, b: number) => Math.min(b, Math.max(a, n));

/** CSS clamp(0.7, 视口高 / 896, 1.2)。只乘滚动距离。 */
export function distanceScale(viewportHeight: number): number {
  return clamp(viewportHeight / SCROLL_CARDS.demoHeight, SCROLL_CARDS.distanceScaleMin, SCROLL_CARDS.distanceScaleMax);
}

export function progress(scrollY: number, start: number, distance: number): number {
  if (distance <= 0) return 0;
  return clamp((scrollY - start) / distance, 0, 1);
}

/** 原文只写了 smoothstep 这个名字。这里用常见的三次 smoothstep。 */
export function smoothstep(t: number): number {
  const x = clamp(t, 0, 1);
  return x * x * (3 - 2 * x);
}

export function stackedCardHeight(svh: number, stickyTop: number, cards: number): number {
  const { cardH, strip } = SCROLL_CARDS.stack;
  return Math.min(cardH, svh - stickyTop - (cards - 1) * strip - 32);
}

/** k 的上限是后面还有几张。缩放原点在顶边中心。 */
export function stackedRecede(scrollY: number, stuckAt: number, cardsAfter: number, recede = SCROLL_CARDS.stack.recede) {
  const k = clamp((scrollY - stuckAt) / recede, 0, cardsAfter);
  return {
    k,
    scale: 1 - k * SCROLL_CARDS.stack.scaleStep,
    shade: k * SCROLL_CARDS.stack.shadeStep,
    origin: "top center" as const,
  };
}

/**
 * 偶数张向右（side = 1），奇数张向左。先向哪一侧原文没指定，这是本仓库的取法。
 * t 到 1 后用 opacity 藏起。原文只说藏起来。
 */
export function deckThrow(scrollY: number, start: number, index: number, step = SCROLL_CARDS.deck.throwPx) {
  const t = clamp((scrollY - start) / step - index, 0, 1);
  const side = index % 2 === 0 ? 1 : -1;
  const { flyX, flyDeg } = SCROLL_CARDS.deck;
  return {
    t,
    x: side * t * t * flyX,
    y: t * (160 * t - 40),
    rotate: side * t * flyDeg,
    origin: "bottom center" as const,
    opacity: t >= 1 ? 0 : 1,
  };
}

/** 最上面一张的 t 增加时，下面的深度下降。深过三层的不画。 */
export function deckPile(index: number, topIndex: number, topT: number) {
  const depth = index - topIndex - topT;
  if (depth < 0 || depth > SCROLL_CARDS.deck.pileMax) return null;
  return {
    y: depth * SCROLL_CARDS.deck.pileDrop,
    scale: 1 - depth * SCROLL_CARDS.deck.pileScale,
    shade: depth * SCROLL_CARDS.deck.pileShade,
  };
}

/** Standard 是 1:1，没有缓动。Short 的 1.25 由调用方改 ratio。 */
export function galleryX(scrollY: number, start: number, overflow: number, ratio = 1): number {
  const travel = overflow / ratio;
  const p = progress(scrollY, start, travel);
  return -p * overflow;
}

export function coverFocus(scrollY: number, start: number, count: number, step = SCROLL_CARDS.cover.step): number {
  const r = clamp((scrollY - start) / step, 0, Math.max(0, count - 1));
  return Math.floor(r) + smoothstep(r - Math.floor(r));
}

export function coverPlace(index: number, focus: number, size = SCROLL_CARDS.cover.size) {
  const d = index - focus;
  const { spread, turnDeg, hideBeyond } = SCROLL_CARDS.cover;
  const spreadPx = spread * (size / 432); // 间距随边长。原文没写比例，432 是 Standard 边长。
  const ad = Math.abs(d);
  return {
    hidden: ad > hideBeyond,
    x: d * spreadPx,
    rotateY: clamp(d, -1, 1) * -turnDeg,
    scale: Math.max(0.6, 1 - ad * 0.2),
    shade: Math.min(ad, 3) * 0.15,
    // 越近越高。具体 z-index 原文没给。
    z: Math.round((hideBeyond - ad) * 10),
    origin: "center" as const,
    dot: clamp(1 - Math.abs(index - focus), 0, 1),
  };
}

/** cubic-bezier(0.45, 0, 0.25, 1)。求值是本仓库示例。 */
export function easeStandard(t: number): number {
  const [x1, y1, x2, y2] = SCROLL_CARDS.image.ease;
  const cx = 3 * x1;
  const bx = 3 * (x2 - x1) - cx;
  const ax = 1 - cx - bx;
  const cy = 3 * y1;
  const by = 3 * (y2 - y1) - cy;
  const ay = 1 - cy - by;
  const sampleX = (u: number) => ((ax * u + bx) * u + cx) * u;
  const sampleY = (u: number) => ((ay * u + by) * u + cy) * u;
  const sampleDX = (u: number) => (3 * ax * u + 2 * bx) * u + cx;
  let u = clamp(t, 0, 1);
  for (let i = 0; i < 6; i++) {
    const dx = sampleDX(u);
    if (Math.abs(dx) < 1e-6) break;
    u = clamp(u - (sampleX(u) - t) / dx, 0, 1);
  }
  return sampleY(u);
}

type Box = { top: number; right: number; bottom: number; left: number };

/** inset 跟着 e 收到 0。圆角从 8px 收到 0。图片缩放从 1.2 收到 1。 */
export function imageOpen(pGrow: number, frame: Box, view: Box) {
  const e = easeStandard(clamp(pGrow, 0, 1));
  const open = 1 - e;
  return {
    e,
    inset: {
      top: (frame.top - view.top) * open,
      right: (view.right - frame.right) * open,
      bottom: (view.bottom - frame.bottom) * open,
      left: (frame.left - view.left) * open,
    },
    radius: SCROLL_CARDS.image.radius * open,
    pictureScale: SCROLL_CARDS.image.pictureFrom - (SCROLL_CARDS.image.pictureFrom - 1) * e,
    // 形状和图片同为 1.2 → 1。后面两层收到 1 是本仓库的读法，原文只写了起始值。
    shapeScale: SCROLL_CARDS.image.pictureFrom - (SCROLL_CARDS.image.pictureFrom - 1) * e,
    titleBehindScale: 1 + (SCROLL_CARDS.image.titleFrom - 1) * (1 - e),
    lineScale: 1 + (SCROLL_CARDS.image.lineFrom - 1) * (1 - e),
    // 两倍速度只用于淡出。60px 上升按 e，不按两倍。titleRise 是向上的像素。
    titleOpacity: clamp(1 - 2 * e, 0, 1),
    titleRise: SCROLL_CARDS.image.titleRise * e,
  };
}

/** 铺满之后的停留。上升写成负的 translateY。原文写 rises，没写坐标正负。 */
export function imageHold(holdScroll: number) {
  return {
    titleY: -SCROLL_CARDS.image.titleParallax * holdScroll,
    lineY: -SCROLL_CARDS.image.lineParallax * holdScroll,
  };
}
```

脚本回退每帧只读一次滚动位置。起点和距离在加载、转向、宽度变化时量，不在 scroll 里量布局。大约 120px 以内的纯高度变化直接忽略。

```ts
export function bindScrollProgress(section: HTMLElement, scroller: HTMLElement | Window = window) {
  const sticky = section.querySelector<HTMLElement>("[data-sticky]") ?? section;
  let start = 0;
  let distance = 1;
  let scheduled = false;
  let lastWidth = window.innerWidth;
  let lastHeight = window.innerHeight;

  const scrollY = () => (scroller === window ? window.scrollY : (scroller as HTMLElement).scrollTop);

  const measure = () => {
    const origin = scroller === window ? 0 : (scroller as HTMLElement).getBoundingClientRect().top;
    start = scrollY() + section.getBoundingClientRect().top - origin;
    // 整段比钉住的那一屏更高。多出来的才是滚动距离。不要用整段的 clientHeight 自减。
    distance = Math.max(1, section.offsetHeight - sticky.offsetHeight);
  };

  const write = () => {
    scheduled = false;
    section.style.setProperty("--p", progress(scrollY(), start, distance).toFixed(4));
  };

  const onScroll = () => {
    if (scheduled) return;
    scheduled = true;
    requestAnimationFrame(write);
  };

  const onResize = () => {
    const w = window.innerWidth;
    const h = window.innerHeight;
    const heightOnly = w === lastWidth && Math.abs(h - lastHeight) < SCROLL_CARDS.ignoreHeightResize;
    lastWidth = w;
    lastHeight = h;
    if (heightOnly) return;
    measure();
    write();
  };

  measure();
  write();
  scroller.addEventListener("scroll", onScroll, { passive: true });
  window.addEventListener("resize", onResize);
  window.addEventListener("orientationchange", () => {
    lastWidth = -1;
    onResize();
  });
}
```

大约 120px 以内、宽度没变的高度变化会跳过。宽度变了、转向了、或高度变化更大，就重新量。封面间距按 `size / 432` 去乘 260，是本仓库对「间距随边长变」的比例。图片 inset 乘在 `e` 上，是本仓库对「跟着 e 打开到 0」的写法。视口缩放的分母用视口高度；钉住块的高度从 100svh 的布局读，不从 `100vh` 读。

减弱动效若选了推荐项，这一段不要钉住：列表、横排，或一张通栏图。选「保留钉住、去掉运动」时，才留下 sticky、把 transform 停在 p = 0。

## 可直接用的 Prompt（精简版）

> Change card sections on a long page that already exists in this product. Keep its content, components, tokens, and copy, and do not add a demo page. Inspect the page first — section order, media, fixed or sticky bars and their heights, breakpoints, vh / svh / dvh, window scroll or an inner scroller, smooth-scroll libraries, anchors, motion tokens, dark mode — and say what you found in a few lines. Then ask one question at a time, each a short option list with the key values and the recommended one marked, and wait: which section gets which of the five interactions (at most one of each, ordinary sections between them; leave anything people scan, compare, or search); Length Short, Standard, or Long; phones and short screens; reduced motion; engine. Summarize the answers in one table, propose the build for this platform with cost and fallback, and wait for a go-ahead. Drive every property from scroll progress, so stopping halfway holds and scrolling back reverses it. Pin with sticky inside a taller section, using 100svh. Animate only transform, opacity, clip-path, and filter, and keep text readable. Use the tested Standard values unless a preset was chosen. Do not hijack scroll, change its speed, catch the wheel, enable smooth scrolling, or add a scroll library for this. Below 560px of viewport height, do not pin. Ignore height-only resizes under about 120px. Demo sizes are for a 1000×896 viewport. Do not invent anything that appears only in the demo video.

上面是改写，不是原文。完整 Prompt 见[作者回复](https://x.com/Kubai087/status/2122496665571803155)。

## 常见错误

- 另做一页演示，或换掉现有文案、组件和 token。
- 五题还没问完，汇总表也没点头，就开始改。
- 一次把五题一起问完。
- 一页里同一种效果做了两次，或几段效果紧挨着，中间没有普通段落。
- 把用来扫读、对比、搜索的段落也改成滚动效果。
- 滚动只负责启动一段定时动画。停在一半，动画仍会跑完；往回滚，不能沿原路倒放。
- 改变页面滚动速度，吸附整页，截获滚轮，或打开平滑滚动。
- 为了这个效果新装 GSAP、Motion，或其他滚动库。
- 钉住段用了 100vh 或 dvh。
- 视口高度低于 560px 仍在钉住。
- 工具栏造成的大约 120px 以内的纯高度变化也重新计算，卡片在滚动中途跳动。
- 写死一个默认窗口很高的高度。
- 动画了宽高、top、margin，而不是 transform、opacity、clip-path、filter。
- 图片展开靠拉宽高变大，而不是让铺满视口的图被 clip-path 打开。
- 某个进度下字读不清，或字被移出页面。
- 展廊钉住超过大约三屏。更多条目该进网格。
- 减弱动效仍在钉住并运动。用户选了推荐项时，应是列表、横排，或一张通栏图。
- 进度提示写成文字标签。细条或圆点就够。
- 从演示视频里补颜色、间距或时序。stone、sage、clay、墨色都没有色值。
- 把演示的 1520px，或 1000×896，当成所有屏幕的常数。
- 手机方案自己加了一个原文没有的宽度断点。
- 锚点没有落在段首。钉住段里的条目 Tab 不到。
- 同一张图在一页里用了两次，或给卡片加了投影。
- 出血内容钻进安全区，或钻进固定页头下面。
- 把开场标题和形状后面的标题做成同一个元素。
- 长度预设把层叠的 320px 停留、飞出的 240px 停留一起改掉。预设表没写这两项。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2122492414598459820 （2026-11-17 00:50:00 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-16）
- Prompt 回复：https://x.com/Kubai087/status/2122496665571803155 （2026-11-17 01:06:53 UTC）
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 本文是改写整理，不是介绍帖，也不是原版 Prompt。没有英文原帖。原帖附演示视频，只在视频里的细节没有写入。
- 示例里这些是本仓库取值，原文没有：smoothstep 用 `t²(3−2t)`；偶数张向右、奇数张向左；飞出后用 `opacity: 0` 藏起；圆点用透明度表示变亮；封面变换原点放在中心，z 用 `(3.5 - |d|) * 10`；封面间距按 `边长 / 432` 乘 260；开场标题淡出用 `1 - min(1, 2e)`，60px 上升只乘 `e`；形状后的标题和前一行小字也收到 1；停留阶段的上升写成负的 `translateY`；下一节的压暗铺在整段盖上来的进度上，28px 圆角只在最后 120px 收平；inset 和圆角都乘 `(1 - e)`；贝塞尔求值用六次牛顿法；滚动距离用整段高度减去 `[data-sticky]` 的高度。视口缩放的分母用视口高度，钉住块的高度从 100svh 的布局读取。
