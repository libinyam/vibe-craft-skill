# 012 · 消息气泡的材质与动效（探索版）
**Message bubble materials and motion (exploratory)**

> 分类：组件 Components · 原帖：[中文 2026-11-05](https://x.com/Kubai087/status/2118508433704169726) · [Prompt](https://x.com/Kubai087/status/2118508837796037053) · 作者 [@Kubai087](https://x.com/Kubai087) / 枯白
>
> 本文是对原帖说明与 Prompt 的改写整理，数值取自原帖；「实现要点」代码为本仓库示例。渐变色标、天空步进、颗粒矩阵与重做 demo 的外壳样式见原帖 Prompt，这里不照录。

## 原理

一块纯色气泡会让 Agent / AI 对话看起来像通用模板。这里给气泡六种材质，思考、消息进入、从中间删掉都跟着材质走，界面才有自己的性格。

六种材质共用同一套气泡盒子，也共用「整列各自用弹簧让位」的运动。变的是涂料、字体、背景，以及发送、展开、退场的性格。两条用法：重做一条独立的对话 demo（一种材质演完这三段，再用 tab 换材质），或只改造已有气泡。动手前先问走哪条、用哪种材质。

## 规则 / 数值

### 先问，再决定改哪里

1. 先问：重做 demo，还是改造现有聊天；用哪一种或哪几种材质。得到回答再写代码。
2. 改造的默认范围只有气泡：材质（下面的盒子与六种涂料）和动效（整列让位、发送、回复展开与逐字、长按与退场）。消息数据、组件 props、加载方式、所有图标都保持原样。
3. 下面三处默认不动。每一处单独问一次，得到同意才改：输入栏（输入框和发送按钮）、消息菜单（外观、条目、开关方式）、正在输入的指示（换成「长回复」里的点阵）。光标只属于重做 demo。
4. 改造开始时先摸清：对话列、气泡组件、消息增删改的位置，以及输入栏、菜单、输入指示现在长什么样。记下列如何排布和滚动、已有哪些动画。标出这三类问题：消息进出时整列跳动、气泡在动画里重新折行、任何在动画 `top` / `height` / `margin` 的地方。
5. 先列出发现（改造）或将要做的东西（重做）和要用的材质，做完再列出改过的每个文件。对不上这套方案时先问。

### 气泡盒子（六种材质相同）

- 字 28px，行高 38px，`letter-spacing: -0.01em`，padding `17px 26px`，描边 2px（材质没有给边色就透明），圆角 38px。单行是胶囊，折行后是圆角矩形。只用普通圆弧，不要超椭圆，也不要 `corner-shape`。发送按钮是圆的。
- 自己在右，对方（AI）在左，两侧留 48px。长消息在固定宽度里折行（原帖 demo 是 680px）。文字从一开始就按这个宽度排，不要等动画过程中再变宽重排。
- 基础字体 Inter。Pixel、Aqua、拟物会换字体；Gradient 会把字距收到 `-0.015em`。
- 本仓库整理：按默认 `content-box`，单行内容高 38px，上下 padding 各 17px，上下边框各 2px，总高 76px，半径 38px 正好是胶囊。不要另写一个把胶囊压扁的固定高度。

### 成组与尾巴

同一发送者连续几条算一组：组内相距 4px，组与组相距 16px。每条标成单条、组内第一条、中间或最后一条。有消息加入或离开时，用 200ms 的 `border-radius` 过渡把组的形状改过去。

- **Cirrus、Aqua、拟物**：两条相接的地方，只把发送者一侧的角收到 10px。外侧和两端仍是 38px。
- **Solid、Gradient**：每条都保持全圆。只有组内最后一条带尾巴，用两个伪元素放在发送者一侧的底角。鼓包向外伸出 12px，旁边用页面底色挖成一弯。变成「组内最后一条」时，尾巴 160ms 淡入。原帖给的是自己这一侧（`right`）；对方气泡做镜像。
- **Pixel** 不靠 DOM 填这个盒子：见下一节，格子按 16px 圆角（相接处 4px）测形状，最后一条是阶梯尾巴。

### 六种材质

涂料、字体、背景做成 token。`.me` 是自己，`.them` 是对方。改造时用气泡上的值；背景只在这种材质离了它就读不出来时才动（Cirrus 要那片天，Gradient 要深色底），而且要先问。

**Cirrus**（轻、干净）：没有描边、没有投影的薄磨砂，底下是蓝天白云。

- 气泡 `background: none`、`box-shadow: none`、`border-color: transparent`，`backdrop-filter: blur(26px) saturate(130%)`。
- 颜色铺在 `::before` 上（`inset: -2px`，圆角继承），字叠在上面。底层是 `--grain` 再叠 `--tint`。自己的 tint 是 `rgb(46 128 232 / 0.34)`，对方是 `rgb(104 134 168 / 0.34)`，字都是白的。
- 颗粒是 160px 的 SVG 平铺：`feTurbulence`，`fractalNoise`，`baseFrequency="0.9"`，`numOctaves="2"`，`stitchTiles="stitch"`，再经 `feColorMatrix` 收成 9% 的中灰。矩阵原文在原帖。
- 天空只渲染一次，尺寸是页面的两倍，之后当图片用：头顶深蓝，下半是成片白云。图片左右各漂 100px，一个来回一圈。视线、步进、fbm 和太阳的参数在原帖，按原帖抄。

**Pixel**（复古游戏）：画在固定 4px 格子的 canvas 上，不用 DOM 填色。

- 每帧用格子中心去测气泡当前的形状：位置、缩放、尺寸、圆角 16px（组内相接处 4px）。组内最后一条有阶梯尾巴。在气泡自己的坐标系里，原点是发送者一侧的底角，x 朝外：`-10 ≤ x ≤ 14`，`-16 ≤ y ≤ 0`，并且 `y ≥ -16 + (x + 10) * 16 / 24`。
- 落在形状里的格子填色；自己在内、邻格在外的是一格宽的描边。描边 `rgb(14 15 34)`，自己的填充 `rgb(93 112 232)`，对方 `rgb(244 243 236)`。变暗时把填充往背景混。
- 每格留残影：`trace = max(trace * exp(-dt / 0.06s), 当前覆盖)`。trace 高于该格的 4×4 Bayer 阈值才画，阈值 `(value + 0.5) / 16`，矩阵顺序是 `0 8 2 10 / 12 4 14 6 / 3 11 1 9 / 15 7 13 5`。trace 低于 1 时，按 `0.25 + 0.75 * trace` 把原色往背景混：拖影大小不变，只是更淡。canvas 用 `image-rendering: pixelated` 放大。
- 背景 `rgb(27 30 52)`，横竖每 4px 一条 1px、3.5% 的白线。
- 字仍在 DOM：`'Pixelify Sans'`，`letter-spacing: 0.02em`，位置吸到 4px 网格。自己是白字，`text-shadow: 0 4px 0 rgb(14 15 34 / 0.35)`；对方 `rgb(14 15 34)`。

**Aqua**（俏皮、带点怀旧）：老 Mac 那种果冻，顶上有一条高光。

- 字体 `'Lucida Grande'`，`letter-spacing: -0.01em`。面用 `--face`，边用 `--edge`。
- 阴影：底部内发光 `inset 0 -14px 18px -8px var(--glow)`，顶部内高光 `inset 0 2px 2px rgb(255 255 255 / 0.5)`，外投影 `0 4px 8px -2px rgb(0 30 80 / 0.32)`。
- 自己是纵向蓝玻璃渐变（亮、中段折暗、底部再亮），`--edge: rgb(21 80 160)`，`--glow: rgb(160 228 255 / 0.95)`，白字，`text-shadow: 0 -1px 1px rgb(0 40 110 / 0.45)`。对方是纵向浅灰渐变，`--edge: rgb(141 148 159)`，`--glow: rgb(255 255 255 / 0.95)`，字 `rgb(29 29 31)`，`text-shadow: 0 1px 0 rgb(255 255 255 / 0.7)`。每一档色标在原帖。
- 高光条 `::before`：`top: 3px`，左右各让出 16px，高 `min(46%, 36px)`，圆角 `30px 30px 18px 18px`，白的竖向渐变从 0.92 落到 0.12。
- 背景是横条纹：`rgb(246 246 246)` 与 `rgb(233 233 233)` 各 3px 交替。

**Gradient**（品牌色）：整列对话共用一条纵向渐变，气泡只露出自己所在高度的那一段，往上走颜色会变。做法接近 Telegram。

- 自己的背景尺寸是 `100% var(--thread-height)`，位置 `0 calc(-1 * var(--top))`。`--top` 是这条气泡当前在列里的偏移，移动时每帧重写。尾巴取气泡脚下那一段。色标在原帖（青绿）。字是白的。
- 对方底 `rgb(38 38 42)`，字 `rgb(244 244 245)`。
- 气泡 `font-weight: 500`，`letter-spacing: -0.015em`。页面背景 `rgb(0 0 0)`。

**Skeuomorphic**（暖、可触摸）：和 001 的拟物按钮同一套配方（面、边、顶光、底暗线、投影）。阴影数字是这一条自己的，不要套 001 的值。

- 字体 `'Avenir Next'`，`font-weight: 500`。
- 自己：边 `rgb(18 22 31)`，面从 `rgb(72 81 100)` 到 `rgb(39 48 66)`，白字，`text-shadow: 0 -2px 0 rgb(0 0 0 / 0.28)`。投影 `inset 0 2px 0 rgb(255 255 255 / 0.18), inset 0 2px 6px rgb(255 255 255 / 0.08), inset 0 -2px 0 rgb(0 0 0 / 0.22), 0 2px 4px rgb(15 23 42 / 0.18), 0 10px 22px -8px rgb(15 23 42 / 0.36)`。
- 对方：边 `rgb(15 23 42 / 0.14)`，面从白到 `rgb(242 243 245)`，字 `rgb(31 41 55)`，`text-shadow: 0 1px 0 rgb(255 255 255)`。投影 `inset 0 2px 0 rgb(255 255 255), inset 0 -2px 0 rgb(15 23 42 / 0.06), 0 2px 4px rgb(15 23 42 / 0.06), 0 8px 18px -8px rgb(15 23 42 / 0.18)`。
- 页面背景 `rgb(245 242 233)`。

**Solid**（默认对照）：最好读，也最省。

- 自己 `rgb(10 132 255)`，白字。对方 `rgb(233 233 235)`，黑字。无投影，页面白底。

### 整列一起动

对话从输入栏上方、自下往上排。每条的位置用 `transform: translateY` 写。消息增加、删掉或变高时，算出每条的新位置，让它从现在所在的地方用自己的弹簧跟过去。其它气泡不需要另外一段动画。列表必须留在正常文档流里时，用同样的思路做 FLIP。

弹簧（response，damping）：

| 材质 | response | damping |
|---|---|---|
| Cirrus | 0.5s | 0.88 |
| Pixel | 0.42s | 0.8 |
| Aqua | 0.46s | 0.74 |
| Gradient | 0.52s | 0.92 |
| Skeuomorphic | 0.34s | 0.84 |
| Solid | 0.42s | 0.86 |

不要动画 `top`、`height`、`margin`。两条气泡不要重叠。

### 发送

气泡从落点下方大约 40px 升起，按材质进入：

| 材质 | 入场 |
|---|---|
| Cirrus | 磨砂从无到有：backdrop blur 与 tint 在 560ms `ease-out` 内到满值；缩放 0.94→1，420ms；淡入 200ms |
| Pixel | 缩放 0.3→1，380ms，`cubic-bezier(.22, 1, .36, 1)`，并且落在格子上；200ms 内经抖动淡入；字几乎到最终尺寸才出现 |
| Aqua | 果冻。缩放 0.5→1，弹簧 response 0.5s、damping 0.5；宽高彼此错开，一开始差 12%，大约 0.2s 内消掉。高度不要超过自身的 1.04，以免顶到上一条 |
| Gradient | 从失焦对上焦：模糊 16px→0，缩放 0.96→1，440ms，同一条 cubic-bezier；淡入 300ms |
| Skeuomorphic | 像一颗键。缩放 1.1→1，弹簧 response 0.36s、damping 0.55，会先沉到 1 以下再回到 1 |
| Solid | 缩放 0.9→1，380ms，同一条 cubic-bezier；淡入 120ms |

### 长回复

正在输入的点阵是原帖用来代替现有输入指示的做法。改造时默认留着原来的指示；要换成点阵，先单独问。重做 demo，或对方同意之后，用下面这套。

点阵单独放着，外面不要套气泡，大约 48px 见方，位置在回复即将开始的那个角。每种材质在上面跑一小段循环，亮度是时间的连续函数，不要做成亮/灭两态。会动的那些用短快门（1/30s 到 1/20s）对一帧取平均，做出运动模糊。点是平的，一种材质一种颜色，要在该材质的背景上站得住；静止时淡，亮起时实。不要高光，也不要倒角。

| 材质 | 网格 | 循环 |
|---|---|---|
| Cirrus | 6×6 | 热度沿慢圆放出，一圈 2.2s。过去 1 秒留下的高斯叠在一起，越旧越宽、越淡 |
| Pixel | 6×6 | 摆锤挂在上沿中点，角度 `0.9 · cos(2πt / 1.1s)`。锤亮、杆淡，四个亮度档，方点 |
| Aqua | 5×5 | 每 0.9s 一滴。涟漪向外走并衰减：半径减时间之后的指数衰减余弦 |
| Gradient | 6×6 | 两盏一样亮的光绕中心转，一圈 1.3s，格子上从薄荷铺到青绿 |
| Skeuomorphic | 4×4 | 弹跳的球。两次弹起间隔 0.62s，中间是抛物线，同时匀速横移，碰到边就折返 |
| Solid | 5×5 | 拨一下弦：基频，加上更轻的二倍频 |

回复落地时，气泡出现在回复开始的那个角，并从那个角展开到消息的尺寸。点阵若在场，同时淡出。

- Cirrus：偏软的弹簧，640ms。
- Aqua：弹簧会略微超过最终尺寸，760ms。
- Skeuomorphic：短弹簧，520ms。
- 其余：520–600ms，`cubic-bezier(.22, 1, .36, 1)`。

文字按最终宽度先排好，用裁切露出，折行在动画里不能变。上方的气泡跟着「正在展开的这条」的当前外框走，不要跟着弹簧的过冲目标走，这样才不会叠在一起。

字从展开开始 160ms 之后逐个进来：

| 材质 | 单字 |
|---|---|
| Cirrus | 上移 10px 并淡入。间隔 18ms，每个 360ms |
| Pixel | 不淡入、不缩放，间隔 26ms 直接出现 |
| Aqua | 从 0.4 弹出，`cubic-bezier(.34, 1.56, .64, 1)`。间隔 16ms，每个 300ms |
| Gradient | 从 8px 模糊里变清楚，同时上移 6px。间隔 14ms，每个 320ms |
| Skeuomorphic | 从 1.14 盖下去。间隔 22ms，每个 160ms |
| Solid | 淡入。间隔 14ms，每个 240ms |

### 从中间删掉一条

长按先把这条压到 0.94–0.97，再抬到 1.02–1.04；其余气泡暗到 35%–50%（Gradient 再模糊 4px）。菜单在气泡下方、靠气泡所在的一侧，条目是 Copy、Regenerate、Read aloud、Delete（Delete 为红）。

菜单的外观、条目和开关动画不在改造的默认范围里。下面的入场只在对方同意改菜单，或在重做 demo 时使用：Cirrus 磨砂渐显；Pixel 从顶部分五步阶梯擦开；Aqua 从 0.7 弹出；Gradient 从 12px 模糊对上焦；Skeuomorphic 从上方 14px 用短弹簧落下；Solid 从 0.88 缩放到位。

长按和气泡退场属于默认要做的动效。确认删除后，菜单在 160–200ms `ease-in` 内收起，气泡按材质退场：

| 材质 | 退场 | 时长 |
|---|---|---|
| Cirrus | 先化回透明玻璃，再淡掉 | 420ms |
| Pixel | 拆成格子，按打散的顺序逐格立刻消失，格子本身不淡出，不留拖影 | 360ms |
| Aqua | 胀到 1.14 再 pop | 320ms |
| Gradient | 溶进 24px 模糊，同时放大到 1.12 | 380ms |
| Skeuomorphic | 先压到 0.94，再下沉 36px 淡出，不要加模糊 | 320ms |
| Solid | 缩到 0.7，带 8px 模糊 | 260ms |

一开始退场就把这条从布局里拿掉。上方的消息填进空位，下方不动。离空位最近的那条大约 60ms 后先动，再往上每一条比它下面那条晚 50–80ms。落到位置时如果还在往下，把速度反向，只留下 20%–45%（Aqua 留得最多，Gradient 最少），轻轻弹一下，不要陷进下面那条。空位两侧若是同一个发送者，合成一组。

### 守住

- 只动画 `transform`、`opacity`、`filter`（玻璃用 `backdrop-filter`）。
- 新的操作从当前状态打断正在跑的动画，不要排队等它结束。中途换材质：整列淡出，再以新材质从当时的进度淡入。
- `prefers-reduced-motion`：留下淡入淡出，去掉位移、缩放和模糊；点阵停在一帧；整列直接放到最终位置。
- 新消息到来时，只要用户没有往上滚动，对话就钉在底部。

## 实现要点（示例实现，非原帖代码）

下面只把共用盒子、成组圆角、纯色和尾巴串成结构，并标成**本仓库示例**。伪元素的 `content` / `position`、类名、`top: 0` 和 `sans-serif` 回退是本仓库补的；几何与色值取自原帖。Aqua 与 Gradient 的渐变色标、Cirrus 的天空与颗粒矩阵、Pixel 的逐格测试不要在这里补全，按原帖抄。

```css
/* 本仓库示例 */
.thread { position: relative; }
.bubble {
  position: absolute;
  top: 0;
  left: 48px;
  max-width: 680px;
  box-sizing: content-box;
  padding: 17px 26px;
  border: 2px solid transparent;
  border-radius: 38px;
  font: 28px/38px Inter, sans-serif;
  letter-spacing: -0.01em;
  transform: translateY(var(--y)); /* --y 含组内 4px、组间 16px；不要动画 top */
  transition: border-radius 200ms;
}
.bubble.me { left: auto; right: 48px; }

.bubble.solid.me {
  --bubble: rgb(10 132 255);
  background: var(--bubble);
  color: rgb(255 255 255);
}
.bubble.solid.them {
  --bubble: rgb(233 233 235);
  background: var(--bubble);
  color: rgb(0 0 0);
}

/* Cirrus / Aqua / 拟物：只收发送者一侧、两条相接的角 */
.bubble.tight.me.is-first  { border-bottom-right-radius: 10px; }
.bubble.tight.me.is-middle { border-top-right-radius: 10px; border-bottom-right-radius: 10px; }
.bubble.tight.me.is-last   { border-top-right-radius: 10px; }

/* Solid / Gradient：组内最后一条的尾巴。--bubble 为气泡色，--page 为页面底色。
   成为最后一条时透明度从 0 淡到 1，160ms。对方气泡把 right 镜像到 left。 */
.bubble.has-tail::before,
.bubble.has-tail::after {
  content: "";
  position: absolute;
  opacity: 1;
  transition: opacity 160ms;
}
.bubble.me.has-tail::before {
  bottom: -2px; right: -14px; width: 64px; height: 38px;
  background: var(--bubble);
  border-bottom-left-radius: 18px 14px;
  border-bottom-right-radius: 3px;
}
.bubble.me.has-tail::after {
  bottom: -2px; right: -34px; width: 32px; height: 42px;
  background: var(--page);
  border-bottom-left-radius: 20px 24px;
}
```

```ts
/** 本仓库示例：整列弹簧。response 为秒。发送、展开用的是另一组弹簧，见上文，不要混用。 */
export const BUBBLE_SPRING = {
  cirrus:       { response: 0.5,  damping: 0.88 },
  pixel:        { response: 0.42, damping: 0.8 },
  aqua:         { response: 0.46, damping: 0.74 },
  gradient:     { response: 0.52, damping: 0.92 },
  skeuomorphic: { response: 0.34, damping: 0.84 },
  solid:        { response: 0.42, damping: 0.86 },
} as const;
```

## 可直接用的 Prompt（精简版）

> Before any code, ask which path and which material(s). Rebuild: a standalone AI-chat demo that plays send, a long reply, and delete-from-the-middle in one material, with tabs to switch among Cirrus, Pixel, Aqua, Gradient, Skeuomorphic, and Solid. Retrofit: restyle only the existing bubbles — the shared box and the chosen material's paint and motion — and keep message data, component props, how messages load, and every icon. Do not touch the composer, the message menu, or the typing indicator unless I agree to each one separately. The cursor exists only in the rebuild demo. Shared box: 28px type on a 38px line, letter-spacing -0.01em, padding 17px 26px, 2px border, 38px circular radius (no superellipse), mine on the right and theirs on the left, 48px from the sides, wrap at a fixed width (680px in the demo) laid out at that width from the start. Groups sit 4px apart inside and 16px between; Cirrus, Aqua, and skeuomorphic close the meeting corners on the sender's side to 10px over 200ms; Solid and Gradient stay round and fade a tail onto the last bubble over 160ms. Move the thread as one: each bubble springs its own translateY (response/damping per material), never top, height, or margin, and never overlaps. Send rises from about 40px below in that material's way. A long reply opens from the corner where it starts, words arriving one by one from 160ms in; replace the typing indicator with the dot matrix only if I say yes. Delete from the middle with that material's exit, then let the messages above fall into the gap and bounce with 20–45% of their downward speed. Animate transform, opacity, and filter only; interrupt from the current state; fade through a material change; honor prefers-reduced-motion; keep the thread pinned to the bottom unless I have scrolled up. Copy every paint value, sky march, grain matrix, and motion curve from the original prompt — do not invent substitutes.

完整 Prompt 见[原帖](https://x.com/Kubai087/status/2118508837796037053)。介绍帖见[中文](https://x.com/Kubai087/status/2118508433704169726)。

## 常见错误

- 还没问清是重做还是改造、用哪种材质，就开始改。
- 改造时动了输入栏、消息菜单、输入指示或图标。
- 用 `top` / `height` / `margin` 做动画，或让两条气泡叠在一起。
- 缩放或展开时让文字重新折行。
- 消息进出时整列跳动，而不是每条从当前位置用自己的弹簧跟上。
- 下一次操作要等上一段动画播完。换材质时硬切，而不是整列淡出再从当前进度淡入。
- 忽略 `prefers-reduced-motion`：位移、缩放、模糊还在，点阵也还在循环。
- 用户已经向上滚动，新消息仍把视图拽回底部。
- 气泡用了超椭圆或 `corner-shape`。
- Pixel 用 DOM 背景色代替 4px 网格，或拖影不经过 Bayer 阈值。
- Cirrus 没有那片天空、Gradient 没有深色底就铺上气泡；换背景之前也没有问。
- 拟物气泡直接套用 001 的阴影数字。
- 把色和时长写死在组件里，而不是 token。

## 出处

- 原帖（中文）：https://x.com/Kubai087/status/2118508433704169726
- 原帖 Prompt：https://x.com/Kubai087/status/2118508837796037053
- 作者：[@Kubai087](https://x.com/Kubai087) / 枯白
