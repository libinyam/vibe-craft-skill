# 021 · 景深环形卡片与流体渐变
**Ring cards with depth and a fluid gradient**

> 分类：质感效果 Texture & Effects · 原帖：[2026-11-19](https://x.com/Kubai087/status/2123579578098536608) · Prompt：[作者回复](https://x.com/Kubai087/status/2123580564272034300) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对介绍帖和作者回复里 Prompt 的改写，不是原文，也没有照抄原版 Prompt。下面的尺寸、透视、曲线和滑块默认值来自介绍帖与那条 Prompt，都是建议，先按这些值做出和演示一致的效果，再用滑块改。帖子时间是 2026-11-20 00:50:00 UTC，也就是洛杉矶时间 2026-11-19 16:50，按本仓库约定记为 2026-11-19。Prompt 回复在 2026-11-20 00:53:55 UTC。没有英文原帖。原帖附约 17 秒演示视频；只在视频里出现、没有写成文字的画面，这里不写。完整 Prompt 和可调演示以作者站点同一条为准：[景深环形卡片与流体渐变](https://vibedesign.tips/zh/021-ring-cards/)（英文页题为 Ring cards with depth and a fluid gradient）。本文中文标题用介绍帖，英文标题用官方英文页。「实现要点」代码为本仓库示例。

## 原理

这是一组有空间感的卡片轮播：同一排卡片排在一条弧上，改曲率的正负，就能在环内、环外之间切换。切换时整排一起弯过去，不是瞬间换一个角度。

环内时，人站在环中间。两侧的卡片更近、更大，向里包围，一屏能看到五张。画面左右各有一条玻璃边，越靠外缘偏移越大，卡片边缘向外张开，并带一点柔和的色散。

环外时，人在环外平视。正中那张清楚，两侧和后排逐渐模糊、淡掉。

卡片正面是缓慢流动的四色渐变。切到环外时，卡片从中间向两侧依次翻面，露出背面的图片。

透视、玻璃边、色散、景深、流速和圆角都做成滑块。默认值就是视频里的效果，可以当场改。

Prompt 要求先看现有项目，一次只问一题，汇总后再动手，不要先做一页演示再往产品里搬。

## 规则 / 数值

### 0. 先看现有项目

问任何问题之前，先在现有代码里弄清楚，并用几行写出来：

- 画面怎么画：DOM、canvas、WebGL、原生视图树，还是 GPU。
- 框架和版本。
- 图片在哪里。
- 有没有现成的滑块或表单控件。
- 有没有暗色模式，以及减弱动效的设置。

然后才开始问。一次只问一题。选项写成短列表，标出推荐，等人回答。

### 一次一题

1. **放在哪。** 单独一页或一个区块（推荐），或指定的组件。
2. **卡片装什么。** 正面流体渐变、背面用项目里的图片（推荐；并说明图片从哪来）；只要渐变；只要图片（这时不要翻面，两种视角都直接显示图片）。
3. **一开始是哪种视角。** 环内（推荐）或环外；以及能不能切换（推荐用两项的分段选择）。
4. **参数面板。** 给所有人看、只在开发时显示，还是藏在一个开关后面。

汇总答案，说明将在当前技术栈上怎么做（见第 9 节），等人点头再写。

### 1. 圆环

卡片沿一条弧排开，弧长刚好等于整排的长度，所以首尾相接。

| 量 | 默认 | 说明 |
|---|---|---|
| 张数 N | 22 | |
| 卡片尺寸 | 180 × 300 | |
| 间隙 | 18 | 步长 step = 宽 + 间隙 = 198 |
| 半径 R | N × step / (2π) ≈ 693 | 周长等于整排长度 |

第 i 张卡在弧上的位置：`s = wrap(i × step − offset)`，折进 `[−N × step / 2, N × step / 2)`。`offset` 是整排已经转过的距离。正对观众的那张是 `round(offset / step) mod N`。

用一个带符号的曲率摆放每张卡：`c = bend / R`。

- `c = 0` 时：`x = s`，`z = 0`，`a = 0`（排成一条直线）。
- 否则：`a = s × c`，`x = sin(a) / c`，`z = (1 − cos(a)) / c`。`x` 横向，`z` 朝向观众，卡片绕竖直中线转过角度 `a`。
- `bend = 1` 是环内（弧弯向观众），`bend = −1` 是环外。

切换视角时，把 `bend` 从一端缓动到另一端，时长 1100ms，曲线 `cubic-bezier(0.65, 0, 0.35, 1)`，让人看见整排从一只环弯成另一只。

### 2. 相机

透视 `P` 是眼睛到正前方那张卡的距离，切换时跟着 `bend` 走：

`P = mix(pIn, pOut, (1 − bend) / 2)`

| 量 | 默认 | 说明 |
|---|---|---|
| pIn | 860 | 比环心稍靠后，弧会弯，但两侧不会被拉得太远。若 `P = R`，眼睛正好在环心 |
| pOut | 1500 | 退得更远，环向外转得比较缓 |

眼睛平视，对着卡片的半高。点 `(x, y, z)` 落到屏幕上是 `x × P / (P − z)`、`y × P / (P − z)`，原点在画面中心。

一张卡只要有任何一部分离眼睛不到 140 个单位，或整张已经离开画面，就不要画。

默认这组数下：环内能看见五张——中间三张完整，外侧两张伸进玻璃边；环外能看见整圈，后排从近处卡片的缝里和外侧露出来。

### 3. 卡面

圆角用连续曲率：半径 32px，超椭圆指数 4，和 CSS `corner-shape: superellipse(2)` 同一条线。在着色器里，到轮廓的距离是：

`d = abs(p) − (w, h) / 2 + r`；`m = max(d, 0)`；`dist = (m.x⁴ + m.y⁴)¹ᐟ⁴ + min(max(d.x, d.y), 0) − r`

正面是四色流体渐变（第 4 节）。背面是图片，要镜像绘制，翻过来才是正的。

不要投影，不要倒影，卡片侧过去时也不要压暗——在白底页面上只会显得脏。远近靠尺寸、模糊和淡出（第 6 节）。

按下时缩到 0.95，用 140ms；弹回用 320ms，都绕中心。

### 4. 流体渐变

用 3 层 fBm 值噪声。每一层之后，把采样点乘矩阵 `(0.8, −0.6; 0.6, 0.8)`，再放大 2.02 倍、加上 `(1.7, 9.2)`，振幅减半。

卡面上的点 `p` 以中心为原点：`uv = p / w × 0.9 + seed`，`seed = i × 3.7 + 0.43`，每张卡不一样。

流动是让噪声场绕圈：两个相位相差 2.1 的偏移，场以每秒 0.2 个噪声格的速度转（Flow 滑块，默认 20%）。因为会回到起点，循环播放不会跳帧。演示里一圈等于动画循环的长度，17.07 秒。

再做两次扭曲后再取颜色。Prompt 里的公式是：

- `w1 = (fbm(uv + o1), fbm(uv + (5.2, 1.3) − o1))`
- `w2 = (fbm(uv + 1.5 × w1 + (1.7, 9.2) + o2), fbm(uv + 1.5 × w1 + (8.3, 2.8) − o2))`
- `f = fbm(uv × 0.9 + 1.7 × w2)`

上色：先在 c0、c1 之间按 `smoothstep(0.3, 0.7, f)` 混合，再按 `smoothstep(0.42, 0.72, w2.x) × 0.9` 靠向 c2，再按 `smoothstep(0.5, 0.8, w1.y) × 0.7` 靠向 c3。顶部加一层向白的柔光，`smoothstep(0.1, −0.6, y / h) × 0.12`。再加 ±1.75% 的轻微颗粒，模糊时淡掉。

每张卡一套四色，例如：

| 名称 | 四色 |
|---|---|
| Tangerine | `#ff7a3d` `#ff3d77` `#ffc46b` `#7a2cff` |
| Lagoon | `#00c2c7` `#2f6bff` `#9ff3e0` `#0b2a6f` |
| Peony | `#ff8fb1` `#ffd1dc` `#b84dff` `#ff5a5f` |
| Citrus | `#ffe14d` `#7ee081` `#ff9f1c` `#2ec4b6` |
| Iris | `#6a5cff` `#b18cff` `#ff7ad9` `#1b1464` |
| Aurora | `#2af598` `#009efd` `#b3ffab` `#12123a` |

也可以从产品品牌色推导，先把四色拿给人看。

### 5. 环内：玻璃边

先把卡片画进一张纹理，再做第二次绘制铺到屏幕上。画面左右各一条玻璃带，默认宽 160px。

像素到所在那一侧的距离是 `d`（`d < rim`）时，去取更靠里的画面：`k = (1 − d / rim)^2.4`，横移 `0.4 × rim × k`，方向朝画面中心，并沿着朝向眼睛的那条线（所以也会往眼睛的高度靠）。卡片的上下边在玻璃下向外张开，从玻璃带内侧的零开始，向外平滑加大。

保持 `0.4 × 2.4 < 1`：带子会把画面往外缘挤，但不会折回来。一折，玻璃里就会出现重复的卡片，或卡片之间的缝。

色散：同一个偏移抽七次，红偏得最多、蓝最少。`t = i / 6`，`i = 0…6`，偏移乘 `1 + spread × (1 − 2t)`，`spread = 0.25 × k`（Dispersion 滑块，默认 25%）。每次的权重是 `w(t) = clamp((1.5 − 3t, 1 − 1.5 × |2t − 1|, 3t − 1.5), 0, 1)`，再除以权重和，色边才会平滑。

玻璃上有一层很淡的亮，向白靠 `k³ × 0.12`。

玻璃边只属于环内：`bend` 回到 0 时淡出；`rim = 0` 则关掉。

### 6. 环外：景深

模糊（单位是屏幕像素）取下面两者里较大的那个。默认模糊 14px。

- 横向：`blur × smoothstep(110, 画面半宽, |screen x|)^1.4`。中间 220px 保持清楚，向两侧平滑加大。
- 远近：`blur × smoothstep(0.25, 0.8, back)^1.2`，其中 `back = −z / (2R)`，正前方为 0，后排为 1。

模糊必须按像素算，不要用 backdrop-blur 去套会动的内容，那会闪。一张卡的覆盖按屏幕上的盒式模糊来算，横竖分开：横向 `clamp((min(dl, b) + min(dr, b)) / (2b), 0, 1)`，`dl`、`dr` 是到卡片左右边的屏幕距离；上下同理。透明度是两者的 smoothstep 相乘，并被同样模糊半径软化过的圆角上限住。比模糊还窄的卡片到不了全覆盖，会淡掉，这和真的模糊一样。

每张卡四周要留 `最大模糊 + 4` 个屏幕像素的边，侧得很厉害时，模糊的边才有地方画。

内容本身也要跟着糊：比当前模糊更细的噪声层淡掉；图片读更小的 mipmap，级别为 `log2(max(1, b × 每屏幕像素的纹素数))`。

后排向页面底色淡出：向白靠 `fade × smoothstep(0.3, 1, back)`，默认淡出 50%。

### 7. 翻面和拖动

切到环外时，每张卡绕竖直中线翻面，从正前方那张向两侧依次翻开。整段 `flip` 从 0 到 1，用 1800ms，曲线 `cubic-bezier(0.3, 0, 0.6, 1)`。每张卡的滞后 `lag = min(|s| / (6 × step), 1)`，转角 `π × smoothstep(0, 1, flip × 1.55 − lag × 0.55)`。翻完后，前半圈是图片，后半圈仍是渐变。切回来就翻回去。

拖动时，整排跟指针走，一像素对一像素。松手后按松开时的速度滑行（目标 = `offset − velocity × 260ms`），再吸到最近的一张，时长 `clamp(|距离| × 1.6, 420, 1100)` ms，曲线 `cubic-bezier(0.25, 0.6, 0.2, 1)`。点一张卡，用 760ms 把它转到正前。

用力一甩（演示里用来打开环内）是 16 张卡、2800ms，曲线 `cubic-bezier(0.4, 0, 0.12, 1)`。速度是一下子提上来的，不是直接满速，循环如果从这一下开始，才不会跳。

新的动作会打断上一个。减弱动效：渐变停住，视角立刻切换，滑动最多 200ms。

### 8. 参数滑块

每一行：名称、滑块、带单位的当前值。环内、环外两套行不同。拖动滑块立刻改画面，并停掉演示动画。复位把所有值恢复成默认。

环内：

| 参数 | 默认 | 范围 |
|---|---|---|
| Perspective | 860px | 600–1400 |
| Glass rim | 160px | 0–240 |
| Dispersion | 25% | 0–60 |
| Flow | 20% | 0–60 |
| Corner | 32px | 0–60 |

环外：

| 参数 | 默认 | 范围 |
|---|---|---|
| Perspective | 1500px | 900–2400 |
| Depth blur | 14px | 0–30 |
| Far fade | 50% | 0–100 |
| Flow | 20% | 同环内 |
| Corner | 32px | 同环内 |

项目里已有滑块组件就用现成的；没有就用普通的 range，轨道有填充，手柄竖直，旁边一个数字框。

改 Flow 时，每一帧往流动的角度上加，不要去乘时间，渐变才不会跳。

### 9. 各平台怎么落

这些是建议，先对照当前技术栈和版本再定。

- **Web**（演示用的做法）：WebGL 2，两遍绘制。每张卡是一条竖直的列带（横向 96 条，两边再各多一条给边距），在顶点着色器里摆好；透视下竖线仍是直的，所以每列只要顶、底两个点。图片放进带 mipmap 的纹理数组。按从远到近、预乘透明画进一张纹理，再做玻璃那一遍到屏幕。一个 `requestAnimationFrame` 循环；2 倍像素密度时，笔记本上保持 60 fps。
- **iOS**：Metal（MTKView）同样两遍；或 SwiftUI，用 Metal `[[stitchable]]` `layerEffect` 做玻璃，`visualEffect` 做卡片。
- **Android**：OpenGL ES 3 或 Vulkan；API 33 以上可以用 AGSL RuntimeShader 在 Compose 里画渐变和玻璃。
- **Flutter**：`FragmentProgram` 着色器画渐变和玻璃，卡片用 `CustomPainter` 的 `drawVertices`。
- **没有 GPU 路径**：CSS 3D 变换摆卡片（`preserve-3d`，同样的 x、z、a），流体改用 CSS 渐变凑合。不做玻璃那一遍；景深用几层台阶模糊代替按像素模糊。

### 10. 做完核对

- 环内：外侧卡片在玻璃里平滑张开，色边柔和，没有折回，没有重复的卡片；一屏五张。
- 环外：看起来像平视的一只环；中间清楚，模糊向两侧和后排平滑加大；转动时不闪，侧得很厉害的卡片没有硬边。
- 翻面从中间向两边走；图片方向是正的；后半圈仍是渐变。
- 每个滑块都会立刻改画面，数值和第 8 节一致，复位能回到默认。
- 中档手机上 60 fps；减弱动效、浅色 / 深色页面、面板的键盘操作都要查过。

## 实现要点（示例实现，非原帖代码）

示例只把圆环摆放、透视投影、视角切换、翻面滞后和景深模糊写成函数。玻璃色散和流体噪声按第 4、5 节的公式在着色器里实现，不在这里再抄一遍。接在现有页面或组件上，不要先做一页脱离产品的演示。

```ts
/** 默认值来自原帖 Prompt，都是建议，不是硬性规定。 */
export const RING = {
  n: 22,
  cardW: 180,
  cardH: 300,
  gap: 18,
  pIn: 860,
  pOut: 1500,
  eyeClearance: 140,
  switchMs: 1100,
  switchEase: "cubic-bezier(0.65, 0, 0.35, 1)",
  rim: 160,
  rimExp: 2.4,
  rimK: 0.4,
  dispersion: 0.25,
  blur: 14,
  fade: 0.5,
  sharpHalf: 110,
  corner: 32,
  pressScale: 0.95,
  pressInMs: 140,
  pressOutMs: 320,
  flipMs: 1800,
  flipEase: "cubic-bezier(0.3, 0, 0.6, 1)",
  dragCoastMs: 260,
  snapMinMs: 420,
  snapMaxMs: 1100,
  snapMsPerUnit: 1.6,
  snapEase: "cubic-bezier(0.25, 0.6, 0.2, 1)",
  bringFrontMs: 760,
  flickCards: 16,
  flickMs: 2800,
  flickEase: "cubic-bezier(0.4, 0, 0.12, 1)",
  reducedSlideMs: 200,
  flowCellsPerSec: 0.2,
  loopSec: 17.07,
} as const;

export const step = RING.cardW + RING.gap; // 198
export const radius = (RING.n * step) / (2 * Math.PI); // ≈ 693

export function wrapArc(s: number): number {
  const half = (RING.n * step) / 2;
  return ((s + half) % (RING.n * step) + RING.n * step) % (RING.n * step) - half;
}

export function placeCard(i: number, offset: number, bend: number) {
  const s = wrapArc(i * step - offset);
  const c = bend / radius;
  if (Math.abs(c) < 1e-8) return { s, x: s, z: 0, a: 0 };
  const a = s * c;
  return { s, x: Math.sin(a) / c, z: (1 - Math.cos(a)) / c, a };
}

export function perspective(bend: number) {
  return RING.pIn + (RING.pOut - RING.pIn) * ((1 - bend) / 2);
}

export function project(x: number, y: number, z: number, p: number) {
  const d = p / (p - z);
  return { x: x * d, y: y * d };
}

export function frontIndex(offset: number) {
  return ((Math.round(offset / step) % RING.n) + RING.n) % RING.n;
}

/** flip 从 0 到 1。切到环外时从正前向两侧依次翻开。 */
export function flipAngle(s: number, flip: number) {
  const lag = Math.min(Math.abs(s) / (6 * step), 1);
  const t = Math.min(Math.max(flip * 1.55 - lag * 0.55, 0), 1);
  const sm = t * t * (3 - 2 * t);
  return Math.PI * sm;
}

export function depthBack(z: number) {
  return -z / (2 * radius);
}

export function depthBlur(screenX: number, z: number, halfWidth: number) {
  const across = Math.pow(smoothstep(RING.sharpHalf, halfWidth, Math.abs(screenX)), 1.4);
  const along = Math.pow(smoothstep(0.25, 0.8, depthBack(z)), 1.2);
  return RING.blur * Math.max(across, along);
}

function smoothstep(e0: number, e1: number, x: number) {
  const t = Math.min(Math.max((x - e0) / (e1 - e0), 0), 1);
  return t * t * (3 - 2 * t);
}

export function snapDuration(distance: number) {
  return Math.min(Math.max(Math.abs(distance) * RING.snapMsPerUnit, RING.snapMinMs), RING.snapMaxMs);
}

export function glassK(d: number, rim = RING.rim) {
  if (d >= rim || rim <= 0) return 0;
  return Math.pow(1 - d / rim, RING.rimExp);
}
```

`wrapArc` 只是把弧长折进半圈；模运算不是新的设计值。`flipAngle` 里的 `t * t * (3 - 2 * t)` 是 Prompt 所写 `smoothstep(0, 1, …)` 的展开，也不是新值。减弱动效时不要跑流动，视角切换不要动画，滑动不要超过 200ms。

## 可直接用的 Prompt（精简版）

> Build a ring of cards in my project, in two views, using the numbers below as defaults, not requirements. Inside: the viewer stands in the middle; cards near the edges come closer and grow; each side of the frame has a 160px glass rim that bends them and splits their colors; five cards are in view at perspective 860. Outside: the ring is seen at eye level from outside at perspective 1500; the front card stays sharp; the sides and far side soften with a 14px depth blur and a 50% fade. Switching views eases bend from 1 to −1 over 1100ms, cubic-bezier(0.65, 0, 0.35, 1), so the row visibly bends from one ring into the other. Switching to Outside turns the cards over from the front outward over 1800ms, from flowing four-color gradients to pictures. Every card's gradient flows slowly (0.2 noise cells a second). A settings panel shows the key numbers as sliders, set to these defaults, and updates live.
>
> N = 22, 180×300, gap 18, step 198, R = N × step / (2π) ≈ 693. Place each card with signed curvature c = bend / R. No shadow, no reflection, no darkening as cards turn away. Corners are a superellipse of exponent 4 over a 32px radius (CSS corner-shape: superellipse(2)). The back picture is mirrored so it reads correctly when turned. Keep 0.4 × 2.4 under 1 so the glass never folds. Never use backdrop-blur on moving content. New motion stops the one before it. Reduced motion: no flow, an instant switch, slides of 200ms at most.
>
> Before building, look at my project: how it draws, framework and versions, where the pictures live, slider or form components, dark mode, and reduced motion. Report that in a few lines. Then ask, one at a time, with the recommended answer marked: (1) its own page or section (recommended), or a component I name; (2) flowing gradients on the front and my pictures on the back (recommended; tell me where the pictures come from), gradients only, or pictures only (then skip the turn-over); (3) opens in Inside (recommended) or Outside, and whether people can switch (a two-option segmented control, recommended); (4) the settings panel shown to everyone, only in development, or behind a toggle. Sum up my answers, say how you will build it on my platform, and wait for my go-ahead. Read the full parameter tables, noise and glass formulas, drag and flick timing, and platform notes in this skill's reference file; do not invent missing values.

上面是改写，不是原文。完整 Prompt 在[作者回复](https://x.com/Kubai087/status/2123580564272034300)里，作者站点同一条为：https://vibedesign.tips/zh/021-ring-cards/

## 常见错误

- 环内、环外做成两套互不相干的布局，切换时整排没有弯过去。
- 环内看不见五张，或外侧两张没有伸进玻璃边。
- 玻璃带把画面折回来，出现重复的卡片或缝。`0.4 × 2.4` 没有小于 1。
- 色散是生硬的红蓝边，不是七次采样、按权重混出来的平滑色边。
- 环外用 backdrop-blur 套在会动的卡片上，转动时画面在闪。
- 中间 220px 没有保持清楚，或后排没有按距离模糊、淡出。
- 侧得很厉害的卡片出现硬边，四周没留模糊边距。
- 给卡片加了投影、倒影，或侧过去时压暗，白底上显得脏。
- 背面图片没有镜像，翻过来是反的。
- 翻面不是从正前向两侧依次翻，或后半圈也变成了图片。
- 渐变循环一圈会跳一下；改 Flow 时用乘法去乘时间，当场跳变。
- 滑块改了画面不跟着变，或没有复位到默认。
- 没先问放在哪、装什么、默认视角和面板给谁看，就先做了一页演示。
- 有现成滑块不用，另造一套。
- 忽略减弱动效：流动还在跑，视角切换还在做 1100ms 动画。
- 把只在视频里出现、没有写成文字的画面补成新规则。
- 给参数表没有写的颜色、曲线或尺寸再补一套。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2123579578098536608 （2026-11-20 00:50:00 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-19）
- Prompt 回复：https://x.com/Kubai087/status/2123580564272034300 （2026-11-20 00:53:55 UTC）
- 作者站点同一条（完整 Prompt 和可调演示）：https://vibedesign.tips/zh/021-ring-cards/
- 英文页同一条：https://vibedesign.tips/021-ring-cards/ （标题 Ring cards with depth and a fluid gradient）
- 作者在后续帖说明效果类技巧会在站点上开放参数滑块：https://x.com/Kubai087/status/2123640171841368384
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 没有英文原帖。原帖附约 17 秒演示视频。只在视频里出现的画面不当成新规则。
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例里的 TypeScript 常量和 `placeCard` / `project` / `flipAngle` 等函数是本仓库的。
