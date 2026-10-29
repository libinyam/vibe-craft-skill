# 003 · Dither 与 ASCII 参数化效果
**Parametric ordered-dither & ASCII effect with a live control panel**

> 分类：质感效果 Texture & Effects · 原帖：[2026-10-27](https://x.com/Kubai087/status/2115091203465290087) · 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖说明与 Prompt 的改写整理，数值与公式取自原帖；「实现要点」代码为本仓库示例。原帖附约 21 秒演示视频，最终观感以视频为准。注：API 返回的原帖 Prompt 中有一处被吞掉的 HTML 标签（指定素材的那句，疑似 `<img>` / `<video>`），本文按上下文理解为"图片或视频元素"。

## 原理

给登录页、卡片、Hero 图加一层**可调**的纹理，而不是一张死图：

- **ASCII**：把画面切成网格，每格按亮度换成一个字符，从 `.` 到 `@` 越来越密。字号 = 大小，格子疏密 = 间距，明暗对比 = 采样强度；"取样区域"决定字符画的是暗部还是亮部。
- **Dither（有序抖动）**：把画面切成像素块，用 4×4 Bayer 矩阵当阈值，把每个颜色通道压成少数几级。像素块 = 大小，纹理尺度 = 间距，采样强度越高，色阶越少、颗粒越重。
- **遮罩**：两种效果都叠一层竖向遮罩，让画面退后——读起来像纹理而不是噪点。

## 规则 / 数值

### 推荐参数
| 参数 | 范围 | ASCII | Dither |
|---|---|---|---|
| 遮罩 Mask | 0–100 | **90** | **88** |
| 色调 Color | Original / 5 套色板 | Original | Original |
| 取样 Sample tones | Shadows / Highlights | **暗部** | （不显示） |
| 采样强度 Strength | 0–100 | **100** | **100** |
| 大小 Size | 0–100 | **37**（字号 7–24px） | **15**（像素块 2–9px） |
| 间距 Spacing | 0–100 | **0**（横 8–24px / 纵 9–26px 步长） | **0**（每个矩阵格 1–4 块） |

两个滤镜各自保存一套参数，切换滤镜时恢复该滤镜自己的值。

### 通用
- `range(v, a, b) = a + (b − a) · v/100`；亮度 `L = (0.299R + 0.587G + 0.114B) / 255`；`s = strength / 100`。
- 采样：把素材按 cover 方式画进一个长边约 480px 的小画布，用 `getImageData` 读取；采样点叠在白底上。选了色板时，先用 256 档查找表把每个像素的 L 映射到色板（四个色标）。
- 每帧均匀取约 12,000 个样本，取 L 的第 8 百分位为 low、第 92 百分位为 high，用于归一化。

### ASCII
- 字符表（由空到满）：`"  .,:;-=+xX08S#&@"`（开头两个空格）。
- 等宽字体，字号 `round(range(size, 7, 24))`px；容器宽 < 720px 时上限用 22。
- 网格步长不能比字形更紧：横向 ≥ 最宽字形 × 1.08；行高 ≥ `@` 高 × 1.12 且 ≥ 字号 × 1.04；第一个格子离边缘半个步长。
- 每格先算原始色调（暗部模式用 `1 − L`），再按 low/high 归一化，二者按 `0.72 + 0.14s` 混合，再过一条随强度变陡的 S 曲线（smoothstep）。色调 ≤ 0.02 的格子跳过；字符 = `chars[round(tone × 16)]`。
- 字色：取样颜色在 HSV 中提亮、略增饱和；透明度随色调与强度增加，夹在 0.06–0.94。（精确系数见原帖。）
- 绘制顺序：（重着色后的）原图 → 遮罩 → 字符。

### Dither（Bayer 4×4）
- 像素块 `max(2, round(range(size, 2, 9)))`（< 720px 时上限 7）；图案缩放 `max(1, round(range(spacing, 1, 4)))`；色阶 `max(2, round(range(strength, 8, 2)))`（强度 0 → 8 级，100 → 2 级）；混合 `blend = range(strength, 0.32, 1)`。
- 矩阵 `[[0,8,2,10],[12,4,14,6],[3,11,1,9],[15,7,13,5]]`，阈值 `(m + 0.5)/16`，按 `floor(x/scale), floor(y/scale)` 取。
- 每块取中心点颜色，先略微推高饱和度，再逐通道量化，然后按 `blend` 与原色混合。
- 写入尺寸为 `容器 / 像素块` 的小画布，再关闭平滑放大绘制。
- 绘制顺序：抖动后的图 → 遮罩在最上。

### 遮罩
- 竖向渐变，色标在顶部、42%、底部。亮色页用白色，基础不透明度 54% / 20% / 38%；暗色页用黑色，44% / 20% / 50%。
- 遮罩值 v ≤ 50 时按 `base · (v/50)^1.08` 减弱；v > 50 时按 `base + (100 − base) · ((v−50)/50)^1.08` 增强到不透明。

### 色板（色标位于 0 / 0.4 / 0.74 / 1）
| 名称 | 色标 |
|---|---|
| meadow | `#2B361F` `#677D45` `#AEBA78` `#E5E1BD` |
| blue | `#192A4D` `#326AC5` `#A2C2F1` `#EDF3FC` |
| amber | `#3C200F` `#A74C15` `#EF9A46` `#F7E7CA` |
| lavender | `#292144` `#6552AB` `#B3A5E4` `#F0ECFB` |
| mono | `#151515` `#575757` `#A6A6A6` `#ECECEC` |

切换滤镜或色板时，从上一帧交叉淡入 150ms。

### 渲染
- 在素材上方叠一个同尺寸、同圆角的 canvas；DPR 上限 1.6，画布上限约 2048×2048。
- 图片：参数变化时重绘（72ms 防抖）。
- 视频：播放且在视口内时每帧重绘（优先 `requestVideoFrameCallback`，回退 `requestAnimationFrame`），原 video 元素留在下面作为源。
- 只作用于指定素材，原地替换：保持其尺寸、位置、圆角和其上叠放的内容。

### 控制面板
- 放在素材旁边：下方有空间就放下方，否则放侧边，绝不遮挡素材；宽约 420px，居中对齐素材。
- 行顺序：遮罩 → 滤镜（ASCII | Dither 两个 tab）→ 色调（圆形色块）→ 取样（仅 ASCII）→ 采样强度 → 大小 → 间距（显示 %）。
- 每行 46px：136px 标签列 + 控件。滑杆：22px 高、8px 圆角、带填充段的轨道；9×30、3px 圆角的滑块；居中数值框；50 处有吸附刻度。色块：22px 圆，用该色板中间两个色标做 135° 渐变（Original 用素材的小裁切）；选中态外加带间隙的描边环。
- 拖动时实时生效；提供"恢复推荐值"按钮，恢复当前滤镜的推荐参数。
- 跟随页面亮 / 暗主题与项目 token。已有图像 / canvas 管线或滑杆、分段控件、面板组件时复用之，否则新建可复用的 `EffectLayer` + `EffectPanel`。

## 实现要点（示例实现，非原帖代码）

```ts
const BAYER4 = [[0,8,2,10],[12,4,14,6],[3,11,1,9],[15,7,13,5]];
const range = (v: number, a: number, b: number) => a + (b - a) * v / 100;

/** 有序抖动：对一个通道值 c(0–255) 量化 */
function ditherChannel(c: number, x: number, y: number, levels: number, scale: number, blend: number) {
  const m = BAYER4[Math.floor(y / scale) % 4][Math.floor(x / scale) % 4];
  const threshold = (m + 0.5) / 16;
  const scaled = (c / 255) * (levels - 1);
  const q = Math.floor(scaled) + ((scaled % 1) > threshold ? 1 : 0);
  return c + ((q / (levels - 1)) * 255 - c) * blend;
}

// 参数映射（strength/size/spacing 都是 0–100）
const levels = Math.max(2, Math.round(range(strength, 8, 2)));
const blend  = range(strength, 0.32, 1);
const block  = Math.max(2, Math.round(range(size, 2, boxWidth < 720 ? 7 : 9)));
const scale  = Math.max(1, Math.round(range(spacing, 1, 4)));

// 绘制：小画布写像素后放大
ctx.imageSmoothingEnabled = false;
ctx.drawImage(small, 0, 0, box.width, box.height);
```

```ts
const ASCII_CHARS = '  .,:;-=+xX08S#&@'; // 17 个字符，索引 0–16
const ch = ASCII_CHARS[Math.round(tone * 16)];
if (tone > 0.02 && ch !== ' ') ctx.fillText(ch, cx, cy);
```

## 可直接用的 Prompt（精简版）

> Add an ASCII / ordered-dither (Bayer 4×4) effect to the one image or video I name, in place (keep size, position, radius, overlays), with a live control panel beside it. One shared settings object { filter, mask, palette, inkTone, strength, size, spacing }, stored per filter. Recommended — ASCII: mask 90, shadows, strength 100, size 37, spacing 0; Dither: mask 88, strength 100, size 15, spacing 0. ASCII maps luminance to " .,:;-=+xX08S#&@" in a monospace grid; Dither quantizes each channel to 8→2 levels with a Bayer threshold and draws blocks scaled up without smoothing. Add a vertical veil mask (stops at top / 42% / bottom) so it reads as texture. Optional 5 palettes (meadow, blue, amber, lavender, mono) via a luminance gradient map. Canvas overlay at DPR ≤ 1.6; redraw images on change (72ms debounce), videos per frame. Panel rows: Mask, Filter tabs, Color swatches, Sample tones (ASCII only), Strength, Size, Spacing; live updates and "Reset to recommended". Follow the page theme and project tokens.

完整 Prompt（含色调曲线、字色、遮罩的全部公式）见[原帖](https://x.com/Kubai087/status/2115091203465290087)。

## 常见错误
- 不加遮罩 → 效果像噪点，抢走前景内容（登录表单）的注意力。
- 放大像素块时开着图像平滑 → 颗粒变糊。
- 不按 low/high 百分位归一化 → 偏暗或偏亮的素材几乎不出字符。
- 每帧对全分辨率素材做 `getImageData` → 卡顿（应先缩到长边约 480px 采样）。
- 视频不在视口内仍然逐帧重绘 → 白耗 CPU。
- 两个滤镜共用一套参数 → 切换后效果面目全非。

## 出处
- 原帖：https://x.com/Kubai087/status/2115091203465290087
- 作者：[@Kubai087](https://x.com/Kubai087)
