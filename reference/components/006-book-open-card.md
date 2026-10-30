# 006 · 像书一样翻开的卡片
**A card that opens a little, like a small book**

> 分类：组件 Components · 原帖：[2026-10-30](https://x.com/Kubai087/status/2116198373040759107) · 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖说明与 Prompt 的改写整理，数值与公式取自原帖；「实现要点」代码为本仓库示例。没点名的选项用下文标 * 的默认值。

## 原理

生成式界面里的卡片最容易长得一样。做法是：**静止时它必须和现在这张卡片完全一样**；只有悬停或键盘聚焦时，才露出它其实是一本薄书——封面掀起一角，后面跟着几页，再是一块封底。

同一套翻页运动可以套三种皮：普通产品卡片（Default）、精装书（Skeuomorphic）、极简文档册（Minimal）。

## 规则 / 数值

### 改哪里

- 找到点名的卡片（或所指网格 / 列表里正在用的那张），**就地改造**。沿用它的结构、设计 token、圆角、阴影，以及项目里已有的弹簧 / 动效工具。
- 占位、内边距、内容和四周布局都不变。卡片外面的东西不许被挤动。
- 只有明确要求「新做卡片」时，才新建一张约 **280×280** 的方卡，并套用选定风格。

### 三种风格（运动相同，只换皮）

| | Default * | Skeuomorphic | Minimal |
|---|---|---|---|
| 表面 | 纯白，或卡片表面 token；无纹理 | 深色封面上的细布纹 | 平涂：米白、浅灰或石墨，一卡一色 |
| 边缘 | 1px 发丝线，用边框色 | 受光倒角：上沿亮、下沿暗 | 很淡的倒角或发丝线 |
| 阴影 | 中性冷灰，**半强度（×0.5）** | 偏暖的中性灰，全强度（×1）；不要橙 | 中性冷灰（强度原帖未单列，见下方选项） |
| 封面上的字 | 产品无衬线：标题 24/600，说明 15 弱化，元信息 14 弱化 | 仅左下角一行衬线标题 | 顶部等宽的章节号 + 版本，下接一条发丝线；两行无衬线标题；底部小灰标签 |
| 封面内容 | 现有卡片 UI：48px 图标块、右上角 ↗、标题、说明、发丝线下的元信息 | 满幅图片或动态 ASCII，顶部薄高光、底部轻暗 | 一条灰色线稿，只有一处用墨水色 |
| 墙（页面底） | 正常页面背景 | 暖灰泥墙，细颗粒，每本书一盏追光 | 冷灰，淡 24px 点阵，光从上方来 |
| 内页 | 白页，靠书脊略暗；小标题 + 4–6 行灰字 | 带颗粒的奶油纸；斜体章名、两端对齐；切口有薄页边 | 冷白；无衬线标题 + 灰行 |

要「跟我的产品走」时：表面、边缘、阴影、字体、墙都取自 token，封面保持卡片今天的样子，不要换成上面某一列的装饰。

Skeuomorphic 与 Minimal 的封面加一条顶边高光和一点底边暗部。Default 不加。

### 选项（* 为默认）

| 选项 | 取值 |
|---|---|
| 幅度 | subtle（A = **24°**）· book（A = **42°**）* · dramatic（A = **60°**） |
| 内页数 N | 2 · 3 · 4 * · 6 |
| 装订边 | 左 * · 右 · 上 |
| 触发 | 悬停 * · 按住时张开 · 滚入视口（约 60% 可见时张开一次，约 1.2s 后合上）。键盘聚焦始终会张开 |
| 内页内容 | 空白纸 · 几行字 * · 卡片的下一项真实预览（后续字段、短列表、小图） |
| 封面 | 现有卡片 UI * · 图片 · 动态 ASCII · 纯色 |
| 光从哪来 | 上方 * · 左上 |
| 阴影强度 | 无 · 轻 ×0.5 · 正常 ×1 * · 深 ×1.4。倍率乘在接触影和环境影的不透明度上。Default 风格自带「轻」 |
| 圆角 | 跟 token * · 连续曲率（超椭圆） |
| 手感 | snappy（k **320**，阻尼 **0.9**）· default（k **240**，阻尼 **0.86**）* · soft（k **160**，阻尼 **0.8**）。都几乎不过冲 |
| 减弱动效 | 微抬 2px * · 完全不动 |
| 触摸 | 按住张开 * · 滚入视口 · 关闭翻页。无论哪一档，第一次点击就要能激活卡片 |

### 层叠

在卡片这个链接 / 按钮内部，放一个 `transform-style: preserve-3d` 的书身。书身里、同一块方形上绝对定位地叠：

1. 封面，n = 0（保留卡片现有内容）
2. 内页，n = 1…N
3. 封底，n = N + 1，**角度永远是 0**

内页和封底是装饰：`aria-hidden`，没有链接，不能聚焦。

- 透视放在**不动的外层**上，约 **1100px**。原点靠装订边：左 `30% 50%`，右 `70% 50%`，上 `50% 30%`。
- 每一层的 `transform-origin` 在装订边：左 `0 50%`，右 `100% 50%`，上 `50% 0`。`backface-visibility: hidden`。
- 用 `translateZ((N + 1 − n) × 0.6px)` 把层在深度上错开，避免 z-fighting。
- 内页比封面内缩：切口一侧 **8px**，书脊一侧 **4px**。内页圆角 = 封面圆角 − 8。
- 封底：Default / Minimal 这种平卡，相对封面内缩约 **6px**，圆角 = 封面圆角 − 6，避免封面透视变短后封底从角上探出来。Skeuomorphic 的封底与封面齐平，颜色是封面色压暗约 **28%**。

### 翻开与合上

层 n 朝读者转过的角度、以及它起步的延迟：

> **angle(n) = A × 0.7ⁿ**  
> **张开延迟 = 28ms × n**  
> **合上延迟 = 28ms × (N − n)**

封面最大、最早；越里面的页转得越少、越晚。合上时倒过来：最里面的页先回到 0°。

方向：左装订 `rotateY(−angle)`，右装订 `rotateY(+angle)`，上装订 `rotateX(+angle)`。

每一层一条自己的弹簧（见「手感」），大约 **0.3s** 停稳。指针中途离开时，从**当前角度和速度**重新瞄向目标，不许跳变。

项目里没有弹簧时，用 CSS 过渡代替：`transform` 约 **420ms**，曲线 `cubic-bezier(0.3, 1.2, 0.5, 1)`（轻微过冲）。`transition-delay` 写在目标状态上——张开态 `28ms × n`，静止态 `28ms × (N − n)`——这样合上自然倒序。

### 光与阴影

光从上方来，影子落在墙的下方。（选「左上」时，同一组形状改向左下；原帖没有另给一套偏移像素。）

影子是卡片**后面**的两块实心形状，各模糊一次，用透明度交叉，**不要用 `box-shadow`**。`box-shadow` 是一圈空心环，封面翻走之后会剩一条描边。

静止时贴着卡片，像普通卡片的投影：

| 层 | 静止 | 张开后的另一套（预先模糊好） |
|---|---|---|
| 接触影 | 卡片下半部，左右各内收约 4px；模糊约 4px，下移约 2px，不透明度约 22% | 模糊约 12px，下移约 12px |
| 环境影 | 模糊约 14px，下移约 6px，不透明度约 12% | 模糊约 38px，下移约 26px |

张开时交叉到更长、更柔的那一对，并用 `transform` 再下移约 **14px**、加宽约 **4%**（跟 o 走，不要改宽高）。用一个进度 **o ∈ [0, 1]** 驱动，弹簧 k **200**、阻尼 **0.9**：静止对的不透明度 = `1 − o`，张开对 = `o`。

颜色是靠近墙面的中性灰（暖墙偏暖、冷墙偏冷）。不要纯黑，不要高饱和。强度见选项表：轻 ×0.5、正常 ×1、深 ×1.4；「无」则不画这两层。

### 页间明暗

每一页有一条从书脊出发的叠影渐变：暗 **32%** → 55% 处 **8%** → 85% 处 **0**。它的不透明度：

> **clamp(gap / 6, 0, 1) × max(0, 1 − gap / 30)**

`gap` 是前一页角度减去本页角度（度）。前一页刚掀起时本页发暗，两页分开后重新被照亮。

### 圆角与边缘

封面、内页、封底都裁到**同一条形状路径**上，发丝线或倒角沿着这条路径描（例如叠一条 SVG 描边）。不要把 CSS `border` 或 inset `box-shadow` 和 `clip-path` 混用：两者在角上对不齐，页分开时会露出第二道边。

- 跟 token：用 token 圆角；内层圆角 = 外层 − 内缩（与 [004](../shape/004-concentric-radius-superellipse.md) 同一条规则）。
- 连续曲率：超椭圆 `|x|ⁿ + |y|ⁿ = 1`，**n = 2^1.25**，也就是 `corner-shape: superellipse(1.25)` 那条曲线。按每一层的尺寸生成 `clip-path: path(...)`，描边复用同一条路径，尺寸变化时重算。

### 交互与无障碍

- 命中区域是**静止时的外轮廓**。悬停监听在不动的外层上；所有会转的层 `pointer-events: none`。否则封面扫过边缘会让目标来回进出，卡片闪一下开、闪一下合。
- 整张卡片是指针手势。
- 整张卡片仍然是原来的那一个 `<a>` 或 `<button>`，无障碍名称不变。内页不进入无障碍树。
- `:focus-visible` 与悬停一样张开。焦点环画在外层上，沿着静止形状，不要跟着封面飞。
- `prefers-reduced-motion`：不做扇形、不旋转。默认只把封面抬起约 **2px**，并把阴影交叉到张开那一对。选项为「完全不动」时连这 2px 也去掉。
- 触摸没有悬停。用上面的触摸选项；默认是按住时张开，但**同一次点击必须能激活**，不能变成「第一下只翻开、第二下才进去」。

### 性能

- 只动画 `transform` 和 `opacity`。不要动画模糊半径、`box-shadow`、`clip-path`、宽高或 `top` / `left`。
- 模糊只做一次：两套影子预先模糊好，再交叉淡入淡出。
- 所有卡片共用**一个** `requestAnimationFrame`。弹簧已经停住的卡片跳过；全部停住就停掉循环。
- 某个元素的样式字符串没变就不要写回（按元素记住上一次）。
- 不要在帧里读布局。量一次，再用 `ResizeObserver` 在尺寸变化时重测。
- 动态封面画布最多约 **30fps**；离开视口（`IntersectionObserver`）或标签页隐藏时暂停。

### 动态 ASCII 封面（只有选了这种封面才做）

- 一小格一个颜色，格子大约 **6×9px**。
- 按亮度从 `" .:-=+*#%@"` 里取字符（开头有一个空格）。
- 底下铺一层约 **40%** 亮度的同色块，上面再画彩色字符，空隙读起来像半调网点。
- 字符从预先画好的字形图集里贴，不要每帧做文本排版。动作慢，并且无缝循环。

参数化的 Dither / ASCII 画面见 [003](../effects/003-dither-ascii-effect.md)；这里只覆盖「书封上的一小块循环 ASCII」。

### 做完后交代

列出改过和新增的文件；写明用了哪种风格、哪些选项，并标出哪些是默认值；如果静止时已经和原来的卡片不一样，明确指出来。

## 实现要点（示例实现，非原帖代码）

```ts
const BOOK = {
  angle: 42,       // subtle 24 · book 42 · dramatic 60
  decay: 0.7,
  staggerMs: 28,
  pages: 4,        // 内页 N；封底 n = N + 1，不旋转
  perspective: 1100,
  spring: { k: 240, damping: 0.86 },       // snappy 320/0.9 · soft 160/0.8
  shadowSpring: { k: 200, damping: 0.9 },
} as const;

/** 张开时封面先动；合上时延迟改成 28ms × (N − n)，最里页先回正。 */
function leafMotion(n: number, pageCount: number, opening: boolean) {
  const angle = BOOK.angle * Math.pow(BOOK.decay, n);
  const delay = (opening ? n : pageCount - n) * BOOK.staggerMs;
  return { angle, delay };
}

/** gap = 前一页角度 − 本页角度（度）。刚分开时最暗，拉开后回到 0。 */
function leafShade(gapDeg: number) {
  const rise = Math.min(1, Math.max(0, gapDeg / 6));
  const fall = Math.max(0, 1 - gapDeg / 30);
  return rise * fall;
}
```

```css
.book-card {                       /* 静止外层：也是唯一的 <a> 或 <button> */
  perspective: 1100px;
  perspective-origin: 30% 50%;     /* 右装订 70% 50%；上装订 50% 30% */
  cursor: pointer;
}
.book-card__body { transform-style: preserve-3d; }
.book-card__leaf {
  position: absolute;
  inset: 0;
  transform-origin: left center;   /* 右装订 right center；上装订 center top */
  backface-visibility: hidden;
  pointer-events: none;            /* 命中永远落在外层，封面扫过边缘不会闪 */
  transition: transform 420ms cubic-bezier(0.3, 1.2, 0.5, 1);
}
.book-card__page { inset: 0 8px 8px 4px; }          /* 书脊侧只缩 4px */
.book-card__back { inset: 6px; }                    /* 平卡封底；精装书则齐平 */

/* 张开：--angle 已是 A×0.7^n 的带符号角度；--z = (N+1−n)×0.6px */
.book-card:is(:hover, :focus-visible) .book-card__leaf {
  transform: translateZ(var(--z)) rotateY(var(--angle));
  transition-delay: calc(var(--n) * 28ms);
}
.book-card:not(:hover, :focus-visible) .book-card__leaf {
  transition-delay: calc((var(--n-pages) - var(--n)) * 28ms);
}

/* 两套影子都在书身外面，filter 写死，只交叉 opacity。不要 box-shadow。 */
.book-shadow__rest { opacity: calc(1 - var(--open)); }
.book-shadow__open {
  opacity: var(--open);
  transform: translateY(calc(var(--open) * 14px)) scaleX(calc(1 + var(--open) * 0.04));
}

@media (prefers-reduced-motion: reduce) {
  .book-card__leaf { transform: none; transition: none; }
  .book-card:is(:hover, :focus-visible) .book-card__cover { transform: translateY(-2px); }
}
```

有弹簧工具时，用手感那一档的 k / 阻尼逐层积分，并把「离开时从当前角度和速度重瞄」接进去；CSS 过渡只是没有弹簧时的替身。阴影的 `--open` 走自己的那条弹簧，不要跟某一页绑在一起。

## 可直接用的 Prompt（精简版）

> Retrofit the card I name so its rest state stays identical to today. On hover or keyboard focus it opens slightly like a book: cover, 4 pages, and a back board that never rotates, hinged on the left, perspective about 1100px toward the hinge. Leaf n turns A × 0.7^n degrees (A = 42°) after a delay of 28ms × n, and closes in reverse — delay 28ms × (N − n) — retargeting from the current angle instead of jumping. Cast the shadow as two pre-blurred solid shapes crossfaded by one 0–1 progress; do not animate blur and do not use box-shadow. Clip every layer to one path and stroke its edge on that same path. Keep a single link or button; hide pages from assistive tech. Under reduced motion, lift the cover about 2px and crossfade the shadow only. Touch has no hover, and the first tap must still activate the card. Animate transform and opacity only. Use the Default skin unless I ask for Skeuomorphic or Minimal; any option I leave unset stays at its default.

原版 Prompt 见[原帖](https://x.com/Kubai087/status/2116198373040759107)。

## 常见错误

- 静止时内页或封底已经探出封面 → 平时就不是「一张普通卡片」。
- 用 `box-shadow` 当地影 → 封面翻走后剩一圈空心描边。
- 动画模糊半径、`clip-path`、宽高或 `top` / `left` → 掉帧。影子应预先模糊两套，只交叉透明度。
- 悬停打在会动的封面上 → 边缘来回进出，卡片闪烁开合。
- 合上不倒序，或指针中途离开就跳回 0° → 没有「由里向外合上」的感觉。
- `border-radius` 和 `clip-path` 各画各的边 → 页分开时角上露出第二道边。
- 内页能被读屏或 Tab 到 → 装饰层必须离开无障碍树。
- 触摸设备依赖 `:hover`，第一下点击被翻页吃掉。
- 开了减弱动效仍在旋转扇页。
- 每张卡片一个 rAF，或全部停住后循环还在跑。

## 出处

- 原帖：https://x.com/Kubai087/status/2116198373040759107
- 作者：[@Kubai087](https://x.com/Kubai087)
