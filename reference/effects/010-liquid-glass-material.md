# 010 · 液态玻璃材质
**Liquid glass for cards, players, and search fields**

> 分类：质感效果 Texture & Effects · 原帖：[2026-11-04](https://x.com/Kubai087/status/2117782193108082923) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对原帖说明的改写整理，不是原文，也没有照抄原版 Prompt。数值取自原帖。原帖附约 39 秒演示视频，观感以视频为准。原帖写明 Prompt 随附，完整 Prompt 以原帖为准。「实现要点」代码为本仓库示例。「性能与降级」一节是本仓库补充，原帖没有写回退。

## 原理

液态玻璃不是一块半透明白底。它是叠在真实背景上的一组薄层：底下有一层只管体积的投影；背景被轻轻模糊，同时拉高饱和度和亮度，所以玻璃是透的而不是发灰；只有边缘一圈把背景往里推，越靠边推得越多，中间是平的；红比蓝多弯一点，边上才有色散；再罩一层半透明着色，浓淡跟着背景有多花走；左上和右下有高光，向内再缩一圈，厚度才站得住。文字也不用一块死色，浅色玻璃和深色玻璃各用一种混合，让字沾上一点背景。

这套材质是作者做 Lumno 时整理出来的，用来换已经有的卡片、播放器和搜索框。浅色、深色各写一套着色浓度，不要共用一个透明度。

原帖称它拆成 6 个复合层，正文接着列了 7 条。前 5 条是玻璃叠层，第 6 条是文字的混合方式，第 7 条是按背景亮度在两套方案之间切换。按这个实现，不要再补出第七层玻璃。

## 规则 / 数值

套到已经存在的卡片、播放器、搜索框上。项目里已有表面 / 卡片组件时，做成 variant 或 token，保留原来的 API、键盘行为和无障碍。所有数值收成命名常量或 CSS 变量。

### 叠层（从底到顶）

原帖按这个顺序写。投影和高光的具体像素原帖没给，不要编一套数值冒充原帖；沿用该组件已有的阴影和高光 token。组件还没有时，用项目里现成的 surface 阴影，并在注释里标明。

1. **投影。** 单独一层 drop shadow，只负责体积。不要拿发光或大模糊代替它。
2. **模糊与提亮。** `backdrop-filter` 模糊背景，同时提高饱和度和亮度。通透感来自饱和度和亮度，模糊本身只有 2px。
3. **边缘折射。** 用 SVG 位移贴图把边缘一圈的背景往玻璃内侧推。离外缘越近，位移越大；距边达到边宽之后，位移为 0，中间保持平整。红色通道比蓝色通道多弯，形成色散。
4. **着色。** 一层半透明色罩。不透明度落在该方案的区间里，并随背景复杂程度变化。原帖没有写「更花 → 更实」还是反过来，也没有给计算公式。
5. **边缘光。** 左上、右下各一道高光，再向内缩一圈加第二道，用来交代厚度。颜色、宽度、透明度原帖都没给。
6. **文字着色。** 浅色玻璃上的文字用 `multiply`（正片叠底）。深色玻璃上用 `overlay`（叠加），把字提亮并带上一点背景色。原帖没有另给一个提亮百分比，不要再加没出处的 `brightness()`。

### 两套方案

折射参数两套共用。着色、模糊、饱和度、亮度按方案分开。

| | 浅色方案 | 深色方案 |
|---|---|---|
| 着色 | 白 | `rgb(20 20 24)` |
| 着色不透明度 | **6%–22%** | **16%–38%** |
| 模糊 | **2px** | **2px** |
| 饱和度 | **180%** | **160%** |
| 亮度 | **1.06** | **0.8** |
| 边宽 | **44px** | **44px** |
| 边缘偏移 | **1.1 × 边宽** | **1.1 × 边宽** |
| 色散 | **±15%** | **±15%** |

边缘偏移是边宽的 1.1 倍（默认 44 × 1.1 = 48.4px），用在最外缘。色散 ±15% 是红、蓝相对这条偏移的偏离：红比蓝多弯。原帖没有给三通道各自的系数。示例里把红色幅度取 `边缘偏移 × 1.15`、蓝色取 `边缘偏移 × 0.85`，绿色落在两者之间，并标明这是拆分方式，不是原帖公式。

着色不透明度必须留在上表的区间内。原帖没给复杂度公式时，可以取区间中值；若要随背景变化，把取舍写进注释，不要写成原帖算法。

### 按背景亮度切换

在玻璃背后取多个点，求平均亮度 L。**L < 0.5 用浅色方案，否则用深色方案。** 这是原帖写明的方向：背景偏暗切浅色玻璃，背景偏亮切深色玻璃。不要按页面主题名对调——亮色页面的底通常 L ≥ 0.5，走的是深色方案。

采样点的个数、位置，以及用相对亮度还是通道平均值，原帖都没写。示例用相对亮度，并标明是示例。

### 边宽在矮控件上（本仓库整理）

边宽默认 44px。搜索框这类矮控件如果短边小于 88px，平整的中间会消失，整块都在折射。此时把边宽收到短边的 40% 以内，边缘偏移仍取当时边宽的 1.1 倍。这是本仓库的取舍，不是原帖参数。

## 实现要点（示例实现，非原帖代码）

常量与切换。亮度公式是示例：原帖只要求「多个采样点的平均亮度，低于 0.5 用浅色」。

```ts
export const LIQUID_GLASS = {
  rim: 44,
  edgeOffsetRatio: 1.1,
  dispersion: 0.15,
  lightBelow: 0.5,
  light: { blur: 2, saturate: 1.8, brightness: 1.06, tint: 'rgb(255 255 255)', alpha: [0.06, 0.22] },
  dark:  { blur: 2, saturate: 1.6, brightness: 0.8,  tint: 'rgb(20 20 24)',    alpha: [0.16, 0.38] },
} as const;

/** 本仓库示例。原帖没写复杂度公式；这里取「更花 → 更不透明」，并夹在原帖区间内。 */
export function tintAlpha(complexity: number, variant: 'light' | 'dark'): number {
  const [min, max] = LIQUID_GLASS[variant].alpha;
  const t = Math.min(1, Math.max(0, complexity));
  return min + (max - min) * t;
}

/** 距外缘的距离；≥ 边宽时返回 0，中间不推。 */
export function edgePush(distanceFromEdge: number, rim = LIQUID_GLASS.rim): number {
  if (distanceFromEdge >= rim) return 0;
  const t = 1 - distanceFromEdge / rim;
  return t * rim * LIQUID_GLASS.edgeOffsetRatio;
}

export function channelOffset(distanceFromEdge: number, channel: 'r' | 'g' | 'b'): number {
  const base = edgePush(distanceFromEdge);
  const d = LIQUID_GLASS.dispersion;
  const gain = channel === 'r' ? 1 + d : channel === 'b' ? 1 - d : 1;
  return base * gain;
}

export function glassVariant(samples: Array<{ r: number; g: number; b: number }>): 'light' | 'dark' {
  if (samples.length === 0) throw new Error('glassVariant: sample the backdrop first');
  const avg = samples.reduce((sum, p) => {
    return sum + (0.2126 * p.r + 0.7152 * p.g + 0.0722 * p.b) / 255;
  }, 0) / samples.length;
  return avg < LIQUID_GLASS.lightBelow ? 'light' : 'dark';
}
```

表面本身放投影、着色和边缘光。模糊、饱和度、亮度放在 `backdrop-filter`。位移只作用在背景采样上，和文字、图标分层，避免把字也推弯。容器加 `isolation: isolate`，文字的混合就停在玻璃里面，不会乘到整页上。这两条是本仓库的落地方式。高光透明度是示意，换成组件已有的高光 token。`--glass-tint` 的默认值取区间中值（浅色 0.14，深色 0.27），原帖没有指定默认点。

```css
.liquid-glass {
  --rim: 44px;
  --edge-offset: calc(var(--rim) * 1.1);
  position: relative;
  isolation: isolate;
  color: var(--glass-ink);
  background: rgb(255 255 255 / var(--glass-tint, 0.14));
  box-shadow:
    var(--glass-shadow, var(--shadow-card)), /* 原帖未给投影数值，沿用已有 token */
    inset 1px 1px 0 rgb(255 255 255 / .45),  /* 左上高光：示例 */
    inset -1px -1px 0 rgb(255 255 255 / .16), /* 右下高光：示例 */
    inset 0 0 0 1px rgb(255 255 255 / .2);   /* 内缩第二道边缘：示例 */
  backdrop-filter: blur(2px) saturate(180%) brightness(1.06);
}
.liquid-glass--dark {
  background: rgb(20 20 24 / var(--glass-tint, 0.27));
  backdrop-filter: blur(2px) saturate(160%) brightness(0.8);
}
.liquid-glass__ink--light { mix-blend-mode: multiply; }
.liquid-glass__ink--dark  { mix-blend-mode: overlay; }
```

折射用 SVG `feDisplacementMap`，红、绿、蓝分开推。贴图用上面的 `edgePush` 来画：距边 ≥ 边宽的区域是中性灰（不位移），边缘的向量指向内侧。像素幅度放在 `scale` 里，不要再烤进贴图，否则会推两次。默认边宽下，最外缘的 scale 是红 55.66、绿 48.4、蓝 41.14，也就是 `48.4 × (1 ± 0.15)`，绿色取中间。边宽一变就用 `channelOffset` 重算。原帖没有给滤镜图和贴图像素公式，不要把这段比例写成原帖参数。

## 性能与降级（本仓库整理）

原帖评论里有人指出，不少液态玻璃到了生产环境性能会很差。原帖没有给降级。下面是本仓库的补充：

- 只铺在卡片、播放器、搜索框这种小表面上。不要给整页背景或长列表的每一行都开 `backdrop-filter` 加 SVG 位移。
- 同时存在的玻璃表面设一个上限；滚出视口的停掉采样和滤镜。
- 不支持 `backdrop-filter: blur(2px)` 时，退回项目已有的实色 surface，保留投影，关掉位移贴图。不要沿用 6%–22% 那种玻璃透明度，否则字会直接压在花背景上。
- `prefers-reduced-transparency: reduce` 时同样退回实色，关掉 `backdrop-filter` 和位移。

```css
@supports not (backdrop-filter: blur(2px)) {
  .liquid-glass,
  .liquid-glass--dark {
    background: var(--surface);
    backdrop-filter: none;
    filter: none;
  }
}
@media (prefers-reduced-transparency: reduce) {
  .liquid-glass,
  .liquid-glass--dark {
    background: var(--surface);
    backdrop-filter: none;
    filter: none;
  }
}
```

## 可直接用的 Prompt（精简版）

> Restyle the existing cards, players, and search fields with liquid glass. If a surface component exists, add a variant or tokens and keep its API. Build six treatments, not a flat translucent fill: (1) a drop shadow for volume, reusing the component's shadow token — the post gives no shadow pixels; (2) backdrop blur 2px plus saturate and brightness; (3) an SVG displacement map that pushes the backdrop inward only inside a 44px rim, stronger at the outer edge and flat past the rim, edge offset 1.1× the rim, red bent more than blue by ±15%; (4) a translucent tint whose alpha stays inside the recipe range and may vary with backdrop complexity — the post gives the range, not a formula; (5) highlights at the top-left and bottom-right plus a second rim set inward; (6) text uses multiply on the light recipe and overlay on the dark recipe, with no extra brightness percentage. Light recipe: white tint 6%–22%, blur 2px, saturate 180%, brightness 1.06. Dark recipe: rgb(20 20 24) at 16%–38%, blur 2px, saturate 160%, brightness 0.8. Sample several backdrop points, average the luminance, and use the light recipe only when that average is below 0.5. Put every number in named tokens. On controls shorter than 88px, shrink the rim so a flat middle remains (repo guidance). Limit the effect to small surfaces, and fall back to an opaque surface token when backdrop-filter is missing or prefers-reduced-transparency is set (that fallback is repo guidance, not from the post).

上面是改写，不是原文。完整 Prompt 见[原帖](https://x.com/Kubai087/status/2117782193108082923)（原帖写明 Prompt 随附）。

## 常见错误

- 只用 `rgba(255,255,255,.2)` 加一层大模糊 → 那是普通毛玻璃。这套材质的模糊只有 2px，通透靠饱和度和亮度，边缘还要有向内的折射。
- 浅色和深色共用一个着色透明度，或把不透明度写到区间外面。
- 位移贴图整张都在推，中间也在弯。距边达到边宽之后必须是平的。
- 红蓝位移一样 → 边上没有色散。
- 把位移方向做成往外推。背景要从边缘被推进玻璃内侧。
- 背景平均亮度已经低于 0.5，仍用深色方案；或看见亮色主题就套浅色方案。原帖是 L < 0.5 才用浅色。
- 文字用纯白或纯黑，浅色玻璃不用 multiply，深色玻璃不用 overlay。
- 给深色文字再加一个原帖没有的 `brightness()`。
- 自己编投影和高光的像素，写成原帖参数。
- 搜索框高度只有 40px 仍用 44px 边宽 → 没有平整的中间。
- 长列表每一行或整页都开 `backdrop-filter` 和 SVG 滤镜。
- 不支持 `backdrop-filter`，或用户偏好减少透明时，表面仍是一块低透明度玻璃，字压在花背景上。

## 出处

- 原帖：https://x.com/Kubai087/status/2117782193108082923
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 本文是改写整理，不是原帖，也不是原版 Prompt。性能与降级、矮控件的边宽收束是本仓库补充。
