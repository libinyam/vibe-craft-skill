# 001 · 拟物按钮：有点击手感的五层材质
**Tactile skeuomorphic buttons with a satisfying click**

> 分类：组件 Components · 原帖：[中文 2026-10-25](https://x.com/Kubai087/status/2114378773655326873) · [English 2026-10-27](https://x.com/Kubai087/status/2115247182853120484) · 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖 Prompt 的改写与整理，数值取自原帖；「实现要点」中的代码是本仓库的示例实现。原帖附 10 秒演示视频，视觉效果以视频为准。

## 原理

平面按钮"没手感"，是因为它缺少一块实体按键该有的光影：光从正上方打下来，按键顶面略亮、上沿有一条反光、下沿有一条暗线，底下有贴地的接触阴影和柔和的抬升阴影；按下去时这些关系反转。把这几层做出来、并且克制（中性灰、无品牌色、无发光、无大模糊），按钮就有了"按键"的质感。参考气质：Lumno 浏览器插件、Cloudflare 控制台——冷静、精确。

## 规则 / 数值

### 五个层级（由强到弱）
| variant | 外观 |
|---|---|
| `primary` | 近黑的石板色面，渐变 `#404859 → #273042`，白色文字 |
| `secondary` | 凸起的白色按键，`#FFFFFF → #F4F5F7`，深灰文字 |
| `tertiary` | 浅灰按键，`#F3F4F6 → #E7E9EC`，深灰文字 |
| `ghost` | 静止时没有面；悬停出现很淡的灰底；按下时是一个凹陷的"井" |
| `text` | 灰色文字 + 细下划线 + 箭头，悬停时箭头向右轻推 |

### 所有实心按钮共用的五层材质（只换数值，结构不变 → 做成 token）
1. **面 Face**：180° 线性渐变，顶部比底部亮约 6–8%。
2. **边 Edge**：1px 实线描边，比面的底色再深一级。primary `#12161F`，secondary `rgba(15,23,42,.14)`，tertiary `rgba(15,23,42,.10)`。
3. **顶部高光**：`inset 0 1px 0` 白色。石板面约 18% 不透明，浅色面 85–100%。
4. **底部暗线**：`inset 0 -1px 0` 黑色。石板面 22%，浅色面 6%。
5. **两层投影**：贴地接触影 `0 1px 2px` + 柔和抬升影 `0 4px 10px -3px`。石板面分别约 18% / 28%，浅色面约为其三分之一。

**关键细节**：所有 inset 层都写在按钮自身的 `box-shadow` 里，并排在外投影之前；不要用伪元素叠一层高光。原因是元素自身的 inset 阴影会自动沿内圆角（外圆角 − 边框宽）走，任何状态下转角都对得齐。

### 状态
- **Hover**：面提亮一级；抬升影扩大到 `0 8px 16px -5px`。
- **Pressed / active**：
  - `translateY(1px)`；
  - 渐变翻转（上暗下亮）；
  - 外投影收缩为 `0 1px 1px`；
  - 顶部高光变成内阴影 `inset 0 1px 2px` 黑色（石板 40%，浅色 12–16%）；
  - 浅色面在下方保留一条 `0 1px 0 rgba(255,255,255,.9)` 的白线，像按进纸里的键。
- **层数对齐**：静止态和按下态的 inset 层数量必须一致（不够就补 `inset 0 0 0 transparent`），这样阴影能过渡而不是跳变。
- **Focus-visible / disabled**：沿用项目组件库已有约定。
- **时长**：transform 120ms，阴影 140ms，颜色 160ms。

### 尺寸（基准：14px 文字）
- 高 40px，左右 padding 18px，圆角 10px，字重 600，可选 16px 前置图标。
- 石板面文字加 `text-shadow: 0 -1px 0 rgba(0,0,0,.28)`。
- **所有长度随字号缩放**：定义 `u = 字号 / 14`，高度、padding、圆角、边框、图标、按压位移、每个阴影的偏移 / 模糊 / 扩散都写成 `u` 的倍数。
  - `md`：14px 文字，u = 1px
  - `lg`：16px 文字，u = 16/14 px（高约 45.7px、圆角约 11.4px、抬升影约 `0 4.6px 11.4px -3.4px`）

### 落地策略
项目已有组件库 / 设计系统时，不要新增组件：通过它自己的 variant 与 token 体系改造现有 Button，API、props、无障碍保持不变。否则新建一个带 `variant` 和 `size` 的可复用 Button。

## 实现要点（示例实现，非原帖代码）

```css
.btn {
  --u: calc(var(--btn-font, 14px) / 14);           /* 尺寸单位 */
  height: calc(40 * var(--u));
  padding: 0 calc(18 * var(--u));
  border-radius: calc(10 * var(--u));
  border: calc(1 * var(--u)) solid var(--btn-edge);
  font: 600 var(--btn-font, 14px)/1 var(--font-sans);
  color: var(--btn-ink);
  background: linear-gradient(180deg, var(--btn-top), var(--btn-bottom));
  /* 顺序：inset 高光、inset 暗线、接触影、抬升影 */
  box-shadow:
    inset 0 calc(1 * var(--u)) 0 var(--btn-hi),
    inset 0 calc(-1 * var(--u)) 0 var(--btn-lo),
    0 calc(1 * var(--u)) calc(2 * var(--u)) var(--btn-contact),
    0 calc(4 * var(--u)) calc(10 * var(--u)) calc(-3 * var(--u)) var(--btn-lift);
  transition:
    transform 120ms ease,
    box-shadow 140ms ease,
    background 160ms ease, color 160ms ease, border-color 160ms ease;
}
.btn--lg { --btn-font: 16px; }

.btn--primary {
  --btn-top: #404859; --btn-bottom: #273042; --btn-edge: #12161F; --btn-ink: #fff;
  --btn-hi: rgb(255 255 255 / .18); --btn-lo: rgb(0 0 0 / .22);
  --btn-contact: rgb(0 0 0 / .18); --btn-lift: rgb(0 0 0 / .28);
  text-shadow: 0 -1px 0 rgb(0 0 0 / .28);
}
.btn--secondary {
  --btn-top: #FFFFFF; --btn-bottom: #F4F5F7; --btn-edge: rgb(15 23 42 / .14); --btn-ink: #1F2937; /* "深灰"，具体色值为示例 */
  --btn-hi: rgb(255 255 255 / 1); --btn-lo: rgb(0 0 0 / .06);
  --btn-contact: rgb(0 0 0 / .06); --btn-lift: rgb(0 0 0 / .09);
}

.btn:hover {
  /* 面提亮一级（各 variant 自行覆盖 --btn-top/--btn-bottom），抬升影变大 */
  box-shadow:
    inset 0 calc(1 * var(--u)) 0 var(--btn-hi),
    inset 0 calc(-1 * var(--u)) 0 var(--btn-lo),
    0 calc(1 * var(--u)) calc(2 * var(--u)) var(--btn-contact),
    0 calc(8 * var(--u)) calc(16 * var(--u)) calc(-5 * var(--u)) var(--btn-lift);
}
.btn:active {
  transform: translateY(calc(1 * var(--u)));
  background: linear-gradient(0deg, var(--btn-top), var(--btn-bottom)); /* 翻转 */
  /* 同样 2 个 inset + 2 个外层，层数与静止态一致 */
  box-shadow:
    inset 0 calc(1 * var(--u)) calc(2 * var(--u)) var(--btn-press),  /* 石板 .40 / 浅色 .12–.16 */
    inset 0 0 0 transparent,
    0 calc(1 * var(--u)) calc(1 * var(--u)) var(--btn-contact),
    0 calc(1 * var(--u)) 0 var(--btn-paper, transparent);           /* 浅色面：rgba(255,255,255,.9) */
}
```

React / Tailwind 项目：把上面的变量放进 `cva`（class-variance-authority）的 variant 里，或在 `tailwind.config` 的 `boxShadow` 中注册 `btn-rest / btn-hover / btn-press` 三组阴影 token。

## 可直接用的 Prompt（精简版）

> Restyle (or create) a tactile, skeuomorphic Button with five variants — primary, secondary, tertiary, ghost, text. Calm and precise, neutral grays, lit from straight above, no gloss / glow / big blur. Every filled variant uses the same five layers as tokens: 180° face gradient (top 6–8% lighter), 1px edge one step darker, inset 1px top highlight, inset −1px bottom shade, two-layer drop shadow (contact + lift). Put all inset layers in the element's own box-shadow, before the drop layers. Pressed: translateY(1px), flip gradient, collapse drop shadow, highlight becomes an inner shadow; keep the same number of inset layers in every state. Timing 120/140/160ms (transform/shadow/color). Base size 14px label → 40px tall, 18px padding, 10px radius; scale every length by u = labelSize/14. If a component library exists, extend its Button instead of adding a new one.

原帖的完整英文 Prompt（含每个 variant 的全部数值）请看[原帖](https://x.com/Kubai087/status/2114378773655326873)。

## 常见错误
- 用 `::before` 叠高光层 → 圆角对不齐、按下态出现缝隙。
- 静止态和按下态 inset 层数不同 → 按下时阴影"跳"一下而不是过渡。
- 固定 1px / 固定阴影值做大尺寸按钮 → 大号按钮看起来材质变薄。
- 加了品牌色渐变、外发光或大模糊 → 立刻变成廉价的 Web 2.0 风格。
- 在已有设计系统旁边另起一个 Button → 两套按钮并存。

## 出处
- 原帖（中文）：https://x.com/Kubai087/status/2114378773655326873
- 原帖（English）：https://x.com/Kubai087/status/2115247182853120484
- 作者：[@Kubai087](https://x.com/Kubai087)
