# 013 · 有物理质感的冲击波（含扭曲与通道分离）
**Shockwave halo with lens distortion and channel separation**

> 分类：质感效果 Texture & Effects · 原帖：[介绍 2026-11-08](https://x.com/Kubai087/status/2119359092334743735) · [线程起点](https://x.com/Kubai087/status/2119359049854779874) · 作者 [@Kubai087](https://x.com/Kubai087) / 枯白
>
> 本文是对原帖说明与 Prompt 的改写整理，不是原文。数值与公式取自原帖；「实现要点」代码为本仓库示例。介绍帖附约 26 秒演示视频，观感以视频为准。
>
> 介绍帖正文写了六层和四组配色。完整 Prompt 在该帖之后的回复里，分成四段。这次没能从 X 列出每条回复的链接；四段正文是从作者同步到 [Bonjour](https://bonjour.bio/posts/6af04012afc4dc392433470f) 的评论读全的（Prompt 1/4–4/4），与介绍帖的六层描述一致。下面的规则按这四段整理。回复的单独链接以原帖线程为准。

## 原理

把两件已有的光效收成一道会推开页面的冲击波：iOS 27.3 屏幕边缘的跑马灯，加上 Dia 每次打开主页时从底部升起的光环。光环扫过时，页面像被一块透镜推开——外侧拉开、内侧挤紧——红和蓝只比绿色多偏或少偏一点，字仍然能读。

它是叠在现有产品上的反馈，不是另做一个演示页。静止时长相不变；光环只在播放的那一秒存在。

介绍帖把效果拆成六层，Prompt 把其中几层做成可关的选项。色环始终开着。

1. **页面快照。** 把当前页面收成一张图，后面的弯曲、色散、边缘折射都从这张图取样。Chrome 的 HTML-in-Canvas 可以当一条路径。
2. **透镜扭曲。** 整道光环是一块透镜，不是一环一环的水波。环的外侧把页面拉开，内侧把页面挤紧。
3. **通道分离。** 红通道的位移是绿色的 1.08 倍，蓝通道是 0.92 倍（介绍帖写的是比绿色多 8%、少 8%）。只在位移处露出一点色散，字保持清楚。
4. **色环。** 四道宽色环彼此交叠，随着扩散一起淡出，决定颜色和亮度。浅色模式是染色，深色模式是加光（screen）。
5. **边缘折射。** 光环只在选定的内容区里走。碰到边缘时，内容像透过玻璃斜边往边缘拉一点；窗框本身不动。工具栏下面不做这一层。
6. **边线反光。** 分割线、描边、芯片和卡片的轮廓在光环经过时朝色环的颜色提亮一下。深色模式更弱。文字和图标不发光。

四组配色：Aurora（蓝 → 紫 → 粉 → 桃）、Halo（橙 → 琥珀 → 金）、Frost（薄荷与水蓝，带一点青柠）、Spark（紫 → 金）。每组都有深色、浅色两套：深色更亮、更饱和，浅色更柔。

## 规则 / 数值

下面的数都是起点，不是必须原样落地。先看现有页面，再逐条问，每条都给出默认并等回答，不要一次问完就写代码。

1. **触发。** 默认是点按或聚焦 AI 输入框。也可以是快捷键、发送、AI 开始回答，或其他。正在播的时候又触发一次：重来，还是忽略。原帖没有默认。
2. **起点和范围。** 默认从底部的 AI 输入框荡开，盖住页面内容区。也可以从按钮、页面底边、点击点或某块面板的中心出发，范围可以是一块面板或整个窗口。工具栏永远不画。
3. **层。** 色环永远开。弯曲、通道分离、边缘折射、边线反光各问一次开或关。弯曲、分离和边缘折射都要页面快照，要说明在当前平台上的成本。
4. **颜色。** 四组里选一组，或用品牌色再导出四环和深浅两套。原帖没有写从品牌色推导四环的公式，不要自己补一套冒充原帖。
5. **输入框。** 默认不动。只有回答要改时，才做成下面的胶囊。

然后说明准备怎么接进现有栈（快照、绘制、成本和回退），等同意再写。

### 光环的形状和时间

默认把圆心放在页面下方：水平对齐输入框中心，在页面底边之下 650px。露出来的是从底边升起的一道宽弧。

- 起步时环的中线贴着页面底边（半径 650），1000ms 内再长 800px，结束时刚过页面顶边。
- `radius = 650 + 800 * (1 - 2^(-9t)) / (1 - 2^(-9))`，`t` 为秒。开头很快，靠近远端变慢。
- 强度与同一段时间：`strength = min(1, t / 0.03) * (1 - t^2.2)`。结尾归零，慢下来的那一段里淡掉。
- 触发当下开始。若输入框正在展开，则在展开开始后再等 40ms。
- 换起点时，时间曲线不变，只改起点半径和增长量，让光环从起点出发、结束时刚过区域最远的边。区域内的点：圆心放在该点，起始半径 0–40px。从一条边出发：圆心放到那条边外侧，大约离开边「区域宽度的三分之二」。
- 裁在选定区域里。工具栏下缘是直边，其余地方用该区域自己的圆角（演示里是 12px，产品用产品的）。绝不画到浏览器或应用的工具栏上。

### 四道色环

环宽 `w = 110px`。`x = (到圆心的距离 - radius) / w`，环心外侧为正。

- 由内到外四环，第 `i` 环（从 0 计）的中心在 `x = i - 1.5`，权重 `exp(-((x - c) / 0.8)^2)`。颜色是四环的加权平均，相邻环软交接。
- 四环共用一次淡出：`alpha = strength * exp(-1.1 * (x / 2)^2)`。
- 浅色（染色）：`mix(page, ring, alpha * 0.24) + ring * alpha * 0.24 * 0.2`。
- 深色（screen 加光）：`1 - (1 - page) * (1 - ring * alpha * 0.3 * 1.5)`。

色值由内到外：

| 配色 | 深色 | 浅色 |
|---|---|---|
| Aurora | `rgb(80 140 255)` · `rgb(160 110 255)` · `rgb(255 110 180)` · `rgb(255 190 140)` | `rgb(110 150 255)` · `rgb(180 130 255)` · `rgb(255 130 190)` · `rgb(255 180 120)` |
| Halo | `rgb(255 150 60)` · `rgb(255 190 70)` · `rgb(255 225 120)` · `rgb(255 245 210)` | `rgb(255 140 50)` · `rgb(255 175 60)` · `rgb(245 200 80)` · `rgb(255 225 150)` |
| Frost | `rgb(80 220 175)` · `rgb(70 195 225)` · `rgb(180 240 150)` · `rgb(235 255 245)` | `rgb(40 190 150)` · `rgb(50 170 210)` · `rgb(140 210 90)` · `rgb(180 235 215)` |
| Spark | `rgb(140 100 255)` · `rgb(200 130 255)` · `rgb(255 200 110)` · `rgb(255 235 170)` | `rgb(130 95 240)` · `rgb(185 125 245)` · `rgb(245 185 80)` · `rgb(250 215 120)` |

深浅切换时，光环的颜色和页面一起交叉淡变，不要硬切。

### 透镜、色散、边线、斜边

整圈是一块透镜。按环分别位移会看成水波，不要这么做。

- `z = x * 0.75`，沿离开圆心的方向 `push = 7px * 2.33 * z * exp(-z^2)`，取样点是当前位置减去这段位移。环中线外侧页面被拉开，内侧被挤紧，影响大约就是色环那么宽。
- 色散只留痕迹：红在 1.08 倍位移处取样，蓝在 0.92 倍处取样。绿走透镜本身的位移。字必须仍然干净。
- 页面边缘保持笔直：透镜在距页面边缘最后 56px 内收到 0。
- 边线反光读的是弯曲之后的位置，所以高光跟着线走。分割线、边框，以及芯片、卡片、节点的轮廓，朝色环颜色走，并先混 20% 白。强度是 `strength * exp(-1.4 * (x / 2)^2)`，再乘浅色 0.3 或深色 0.14。这是一道淡反光，不是重画一条边。文字和图标不提亮。
- 找线的办法：再截一张图，只把分割线和边框按它们自己的宽度画成白、底是黑，其余（字、图标、图片、填充、阴影）都不画。白的地方才提亮，于是 1px 的边仍然是 1px。没有这张图时（直接改 DOM）：每一帧按元素到光环前沿的距离，给它一个 0–1 的变量，用这个变量把边框色混向色环色。
- 玻璃斜边只在左右和底边，不在工具栏下方。光环碰到边缘时，最外 60px 改从更内侧取样，最多再往里 22px：`22 * (1 - d/60)^2 * 该处强度`，内容朝边缘拉开。光环与边缘相遇处加一条细光，`exp(-d / 7)`，颜色是色环混 45% 白。

### 节奏

- 若输入框要展开：先缩到 97% 共 90ms，再用 580ms 弹开，金属边的高光转 900ms。光环在展开开始 40ms 后出发，跑 1000ms。不改输入框时，光环就在触发时跑。
- `prefers-reduced-motion`：不要弯曲、不要色散、不要反光。光环停在原位，半强度，600ms 内淡入再淡出。
- 只在光环可见时绘制。静止时拿掉或藏起画布，不留一层空的合成。

### 胶囊输入框（仅当明确要改）

静止：300×44、圆角 22、白底（深色 `rgb(30 30 33)`）、1px 细线、一枚闪光图标、14px 占位 “Ask AI about this project”、右侧快捷键提示。放在内容区水平居中、距页面底 48px。

点按后缩到 97% 共 90ms，再从自身中心弹成 460×100 的卡片，圆角 16，底边留在原地。进度 `1 - exp(-8t) * cos(6t)`，580ms，会稍微越过大头再在结束时落稳。卡片内容按最终尺寸先排好，随着长大露出；从进度 30% 起淡入。高度和圆角走同一条进度。

打开后的卡片：一行字 15px / 行高 22px，距左 16px，先是插入符再是占位；一条细线；工具栏高 52px：圆形添加钮、短分隔、两个选择器（模型、模式），右侧语音和发送都是 32px。

阴影宽、软、远，不要压得很重：`0 0 0 1px rgb(15 23 42 / 0.09)`，`0 2px 4px rgb(15 23 42 / 0.04)`，`0 28px 64px -16px rgb(15 23 42 / 0.22)`。不要焦点环，焦点只表现为插入符。

金属边是 1px，浅色银色渐变 `linear-gradient(120deg, rgb(238 240 243), rgb(196 202 210) 28%, rgb(241 243 245) 50%, rgb(188 195 204) 74%, rgb(233 236 239))`。深色原帖只给了两端 `rgb(74 79 87)` 到 `rgb(51 55 61)`，中间色标没有写，不要补。顶上的高光是锥形渐变：0–280° 透明，325° 为 `rgb(255 255 255 / 0.8)`，360° 再透明。角度从 -60° 转到 300°，900ms ease-out；整条边的不透明度是 `sin(πt)`，最高 0.75。渐变只留在边框上（`mask` 用 content-box 挖空）。关闭回到胶囊，320ms ease-in-out。

### 怎么画

色环和边线反光只是一层覆盖，任何平台都能做。弯曲需要能按偏移去采样的页面图像。

- **Web（演示的做法）。** 页面上方一块 WebGL2 canvas，一次 fragment 里做完。像素比上限 2。画布不接指针，静止时移除或隐藏。
- **快照。** 有 HTML-in-Canvas 就用它（`drawElementImage()`，或当成 WebGL 纹理）：近期 Chromium 在 `canvas-draw-element` 旗标后，原帖写 Chrome 157 起有报道说默认打开；Firefox 和 Safari 还没有。否则用 SVG `foreignObject` 一类库（html-to-image、modern-screenshot），点击时截一次再播，图片和字体要同源或内联，根节点要有背景色，`backdrop-filter` 不可靠。html2canvas / html2canvas-pro 会自己重画 CSS，`backdrop-filter`、部分渐变、oklch 可能错，留到最后。没有快照：只在透明画布上画光环（浅色普通 alpha，深色 `mix-blend-mode: screen` 或 `plus-lighter`），跳过弯曲。仅 Chromium 还可以用指向 `feDisplacementMap` 的 `backdrop-filter` 做弯曲，原帖说代价高，没有给滤镜参数。
- **iOS 17+（SwiftUI）。** Metal stitchable 函数，`.layerEffect(shader, maxSampleOffset: 30×30)` 贴在页面视图上，可以按弯曲位置采样并加上颜色。半径和强度用 `TimelineView` 或可动画的值驱动。iOS 17 以下：一层 `AngularGradient` 加模糊的覆盖，没有弯曲。
- **Android 33+。** AGSL `RuntimeShader`，内容作为 uniform shader，`RenderEffect.createRuntimeShaderEffect` 设到页面根上（Compose 用 `Modifier.graphicsLayer` 的 `renderEffect`）。每帧更新 uniform。33 以下只做覆盖。
- **Flutter（Impeller）。** `FragmentProgram`，`ImageFilter.shader` 放进裁到页面上的 `BackdropFilter`。内容是 sampler 0，尺寸是第一个 `vec2`。用 `FlutterFragCoord()`，GLES 上翻转 y。先查 `ImageFilter.isShaderFilterSupported`，不支持就只做覆盖。
- **Rust。** UI 先画进离屏纹理，再用同一套着色器的 WGSL 做一次全屏采样。wgpu 用手写的全屏三角形和一块 uniform；Bevy 走自定义后处理节点，`post_process_write()` 拿源和目标；egui / iced 经渲染器的 wgpu 回调合成，它们的控件自己采不到底下的像素。
- **Electron / Tauri。** 走 Web。Electron 还可以 `webContents.capturePage()`。

### 自查

深色和浅色都看过：光环下的字可读、没有被撕开；工具栏上没有光环；区域圆角是干净的；弯曲没有比色环更宽。触发和起点是问过的那一组，页面其余行为与原来一致。光环不可见时没有绘制成本。

## 实现要点（示例实现，非原帖代码）

只把半径、强度和透镜位移收成函数。配色、色环权重和平台快照按上一节，不要在这里另写一套。

```ts
/** 本仓库示例。t 为秒。默认圆心在页面底边之下 650px。 */
export function haloRadius(t: number, start = 650, growth = 800) {
  const u = (1 - 2 ** (-9 * t)) / (1 - 2 ** -9);
  return start + growth * u;
}

export function haloStrength(t: number) {
  return Math.min(1, t / 0.03) * (1 - t ** 2.2);
}

/** x = (到圆心的距离 - 半径) / 110，向外为正。返回沿径向的位移，单位 px。 */
export function lensPushPx(x: number) {
  const z = x * 0.75;
  return 7 * 2.33 * z * Math.exp(-(z * z));
}

/** 通道分离：绿走 lensPushPx(x)，红走 1.08 倍，蓝走 0.92 倍。 */
export const CHANNEL_OFFSET = { r: 1.08, g: 1, b: 0.92 } as const;
```

## 可直接用的 Prompt（精简版）

> Add a shockwave halo inside my existing product, not a demo, and do not change how the page looks at rest. Look at the app first, then ask one question at a time, with the default each time: what triggers it; where it starts and which area it covers (toolbars stay clear); whether to enable the bend, the channel split, the edge bevel, and the border glints (the four color rings stay on); which of Aurora, Halo, Frost, Spark — or my brand colors — and whether to leave my AI input alone (the default) or turn it into the spring-open pill. Default geometry: center 650px below the page bottom, radius = 650 + 800 * (1 - 2^(-9t)) / (1 - 2^(-9)) over 1s, strength = min(1, t/0.03) * (1 - t^2.2), clipped to the content area. One lens, not ripples: outside stretches, inside squeezes; sample red at 1.08× the offset and blue at 0.92×. Fade the lens over the last 56px before the page edge; bevel the last 60px at the sides and bottom. Borders glint, text and icons do not. Light mode tints, dark mode screens. Draw only while it runs. Reduced motion: no bend, split, or glints; fade in place at half strength over 600ms. Use the original prompt for the four palettes, the ring weights, the pill, and the per-platform snapshot path.

完整 Prompt 在[介绍帖](https://x.com/Kubai087/status/2119359092334743735)之后的回复里。本次读到的四段全文见作者的 [Bonjour 同步](https://bonjour.bio/posts/6af04012afc4dc392433470f)。

## 常见错误

- 还没问触发、起点、哪些层、配色，就把演示页的胶囊和四组色一起做上。
- 一环一环地位移，光环变成水波纹。
- 色散超过约 8%，或让文字一起发亮，字被撕开或糊掉。
- 光环画进工具栏，或在页面边缘还在弯，边线不再是直的。
- 没有快照却硬做弯曲；或静止时仍留着一块全屏画布。
- 深浅切换时把光环颜色硬切，不跟页面一起淡。
- 减少动态效果时仍在弯页面。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2119359092334743735
- 线程起点（正文在「1.」处截断，完整说明见介绍帖）：https://x.com/Kubai087/status/2119359049854779874
- Prompt 四段的同步：https://bonjour.bio/posts/6af04012afc4dc392433470f （X 回复链接本次未能逐条列出）
- 作者：[@Kubai087](https://x.com/Kubai087)
