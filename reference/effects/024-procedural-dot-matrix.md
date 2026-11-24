# 024 · 程序化生成点阵
**Dot-matrix loading and reveal animations for AI images**

> 分类：质感效果 Texture & Effects · 原帖：[2026-11-24](https://x.com/Kubai087/status/2125220063301271808) · Prompt：[作者回复](https://x.com/Kubai087/status/2125220175591145682) · 演示：[作者站点](https://vibedesign.tips/zh/024-image-loading/) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对介绍帖、作者站点同一条和附带 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。用可调点阵代替传统 Loading、入场可以带色散、鱼眼、边缘扭曲和冲击波，来自介绍帖。九种点阵、九种出图、共用的落定、时序、性能、调参面板和分平台做法，来自作者站点同一条和官方 Prompt，下面会标明。只在演示视频里的细节不当成新规则。介绍帖时间为 2026-11-24 13:28:42 UTC，Prompt 回复为 2026-11-24 13:29:09 UTC，演示回复是 https://x.com/Kubai087/status/2125234339374137635 。按本仓库约定，介绍帖记为 America/Los_Angeles 的 2026-11-24。没有英文原帖。介绍帖附约 51 秒演示视频。中文页题为「[AI 生图的点阵等待与出图动效](https://vibedesign.tips/zh/024-image-loading/)」，英文页题为 [Dot-matrix loading and reveal animations for AI images](https://vibedesign.tips/024-image-loading/)。本文中文标题用介绍帖，英文标题用英文页标题。「实现要点」代码为本仓库整理 / 示例实现。

## 原理

生图常常要等好几秒。画框中间只放一个转圈，等待是空的；生成结束直接换成成图，衔接又太硬。介绍帖的做法，是从 GPT 的生图组件得到启发，用噪声、数学函数，加上作者此前分享过的冲击波，做一套参数可调的点阵，用来展示内容正在生成，替换传统的 Loading 图形。

作者站点把等待和出图写成同一段：生成时，画框里铺一层会呼吸的点，上面一行状态字有一道光扫过；图片就绪后，从这些点里面显出来，再落到本身的尺寸和清晰度。点阵九种，出图九种，可以任意搭配。介绍帖点名的入场手段是色散、鱼眼、边缘扭曲和冲击波。色散和边缘折射写在每种出图共用的落定里，鱼眼是九种出图之一，冲击波只加在三种出图上。

Prompt 里的每个数都来自一份能跑的演示，是默认值，不是硬性规定。先按这些默认做出来，再用面板调。这是接到产品里已经有的生图流程上，不是另做一页演示。先看现有代码怎么请求、怎么接到结果、用什么绘制，用几行说清，然后一次问一件事，等人点头再写。产品原有的布局、配色和文案留下。

## 规则 / 数值

先看清楚，再建议。点头的那些才做。

### 先看产品里的生图

这些来自官方 Prompt。动手之前先找到：

- 请求从哪里发出，结果以 URL、blob、base64 还是流回来，有没有进度或部分图像，画框是哪个组件。
- 项目用什么画（DOM、canvas、WebGL，或原生视图树）、框架和版本、现成的滑块、暗色模式、减弱动效。

用几行把发现说完，再往下问。一次只问一件，并把推荐项标出来：

1. **放在哪。** 每一处会出现生成图的地方（推荐；把它们列出来），或只改点名的那几处。
2. **哪一组点阵和出图。** 全产品同一组（推荐：fBm breath 配 Halftone grow，也可以改从下面九组里选），每个位置一组，或每张新图换一组。
3. **面板谁能看见。** 只在开发环境（推荐），放在一个开关后面，或所有人都能看见。

把回答收成一段，说明会按产品所在平台怎么落地，然后等一句可以开始。没有这句，不写实现。

### 时序

下面的时钟和文案来自官方 Prompt。作者站点重写了其中的解码、300ms 和失败。

- 点阵用 500ms 淡入。
- 状态行先是 “Making the first draft…”，1.3s 后换成 “Refining the details…”，图片解码完成、开始显影时变成 “Done”。产品有自己的句子和语言，用产品的。这三句只是没有现成文案时的起点。
- 出图要等图片解码完（`image.decode()`，或该平台上等价的一步）再开始。网络响应到了、解码还没完，不出图。
- 出图默认 1400ms，曲线 `cubic-bezier(0.16, 1, 0.3, 1)`：进来快，落定慢。把缓动后的进度记为 R，从 0 到 1。
- 请求发出后 300ms 内就已经解码好的图，跳过点阵，直接淡入。否则 Loading 会闪一下。作者站点写明，这条是延续 017 那条「等待状态」的原则（英文页称为 Four loading patterns and when to use each）。本分支没有 017 的文件，这里只记这条原则。这次直接淡入的时长，Prompt 没有写；下面的 200ms 只用于减弱动效。
- 请求失败：点停在当前这一帧，说明出了什么，并给出重试。
- 一次多张（一批，或一张宫格）：每张用自己的延迟，时刻错开。Prompt 给的 3×3 例子是 0、560、220、820、380、1040、120、700、300ms。不要同一拍一起出来，也不要排成一次扫过。

### 画框、状态行和点

这些来自官方 Prompt。作者站点用自己的话重写了点距、暗角、深色光晕和边缘微光。

画框用这张图自己的比例，带圆角。演示里的一张卡片是 600×772，卡片圆角 28px，画框圆角 14px，两者同心。这是那张演示卡的默认，不是每张图都要做成 600×772。

状态行在画框上方，20px，颜色是偏淡的墨色。一道 96px 宽、两端柔和的满墨窗口，每 1.7s 扫过文字一次。窗口和窗口里的文字副本用 transform 反向移动，正文不跟着重绘。

点排成正方形网格，点距默认 20px，在画框里居中。每个点的值 v（0 到 1）由它的等待点阵决定，按点算一次，不按像素算。v 把半径从 0.55px 带到点距的 15%，不透明度从 0.1 带到 1。靠近画框边缘做暗角：`0.45 + 0.55 * smoothstep(8, 72, 到边缘的距离)`。

深色才有的点光晕：每个亮点一圈高斯，σ = 0.3 × 点距，强度 v²。把本格和周围八格加在一起，光晕跨格才不会出缝。

深色、且仍在等待时，画框边缘有两层光，周期错开，避免一起涨落。外层：宽的雾（`exp(-d / 12)`）加边缘一条细线（`exp(-d / 2.5)`），d 是从边缘往里的距离；每 2.4s 呼吸一次，两条较亮的弧每 8s 顺时针转一圈。内层：大约在往里 6px 处有一条软带（高斯宽 3.5px），再加一层很淡的宽雾；每 3.7s 呼吸一次，从自己涨落的反相点开始，三条弧每 11s 逆时针转一圈。出图开始后，这两层在前三分之一的 R 里淡掉。

浅色：浅底上的深色点，没有光晕，出图前缘的边也更收一点。Prompt 没有写「更收」的具体系数。

### 九种等待点阵

作者站点把九种分成三类，用来认效果。每一类下面的式子来自官方 Prompt。

共同的输入：p 是点中心，单位像素；t 是呼吸时钟，单位秒。呼吸滑块改变 t 走得有多快，相位要接得上，改速度时画面不能跳。n 是值噪声叠成的 fBm：四个八度，后一个频率翻倍、振幅减半，采样位置是 `p / scale`（scale 默认 180px），时间是 `t * 0.32`。

**噪波类**

- **fBm breath。** 四层分形噪声慢慢涨落，看起来像云，也是别的效果垫在下面的那层。`v = smoothstep(0.28, 0.74, n)`。
- **Domain warp。** 用一层噪声把另一层的采样点推开，再往一个方向漂，像烟。`drift = (0.22t, -0.1t)`；`w = (fbm(q * 1.1 + drift), fbm(q * 1.1 + drift + 5.2))`；`v = smoothstep(0.34, 0.7, fbm(q + 4(w - 0.5) + 1.6 * drift))`。q 按本节开头的采样来理解，就是点中心除以噪波尺度；这一条里 Prompt 直接写了 q，没有再定义一次。

**几何类**

站点的概括是：几何类都叠一层浅淡噪声，点与点的亮度不完全一样，才像「还在生成」。各条的混合比以 Prompt 为准。双螺旋那一条没有写出和 n 的混合比，这里不补。

- **Ripple。** 从中心扩出的环。`r = |p - 中心| / scale`，`w = 0.5 + 0.5 * sin(r * 5.2 * π / 2 - 2.6t)`，`v = smoothstep(0.15, 0.85, mix(n, w, 0.6) * (0.55 + 0.6n))`。
- **Spiral arms。** 三条旋臂。这一条没有单独定义 r 和 θ；按 Ripple 的写法，r 是到中心的距离除以 scale，θ 是方位角。`s = 0.5 + 0.5 * sin(3θ + 5.5r - 2.2t)`，`v = smoothstep(0.12, 0.85, mix(n, s, 0.55) * (0.6 + 0.5n))`。
- **Sine plasma。** 四条正弦加在一起。x、y 用噪波格子（p / scale）：`sin(2.6x + 0.9t) + sin(3.1y - 1.1t) + sin(2.2(x + y) + 0.7t) + sin(4.4r - 1.5t)`。第四项的 r 是同一套格子里到中心的距离。`v = smoothstep(0.2, 0.8, mix(0.5 + 0.125 * 和, n, 0.35))`。
- **Light band。** 一条宽 0.15 的高斯带，每 3.3s 斜着扫过一遍，底下是半强度的噪声。Prompt 没有再给这条带的方向角和式子。
- **Double helix。** 两股横跨画框、一边转一边走。`y = ±72 * sin(x / 44 - 1.8t)`，坐标系是「画框高度 = 306」，所以 72、44 和下面的宽度 10 不要直接当成像素。每股是宽度 10 的高斯，离得近的那股更亮，亮度随相位的 cos 走。两股之间每隔一列有一截连线（Prompt 写的是 base pairs）。
- **Snowflakes。** 六臂雪花，每条臂在一半长度处分叉。每片占一个边长 `z = max(0.27 * 画框高度, 11 * 点距)` 的方格，在格里随机落位，臂长 0.37z 到 0.5z，一边慢慢转一边以 18px/s 下落，带一点晃。后面是单点的细雪：每 `0.4z` 的方格一粒，下落速度是雪花的两倍，亮度 35% 到 60%。最底下是 12% 的噪声。把角度折进一圈的十二分之一，画一条臂和它的分叉就得到全部六条。臂至少跨四个点，否则看起来是一块斑。晃的幅度和自转的角速度，Prompt 没有写。

**随机类**

- **Static grain。** 每个点按自己的相位，在两个随机亮度之间平滑过渡，大约每秒 2.5 次，以 42% 混在 n 上。过渡用 smoothstep。不用平滑，点就会闪，看起来像坏掉的屏幕，而不是旧电视的雪花。

### 每种出图都有的落定

这些来自官方 Prompt，九种出图共用。作者站点把放大、玻璃边、色散和解码收成了同一节。

- **放大。** 绕画框中心采样，倍数是 `1 + 0.10 * (1 - R)`：进来时放大 10%，最后回到原大。画框本身不缩放。没有 GPU、改用 CSS 时，Prompt 允许用 `transform: scale` 做这一步，见后面的降级。
- **变清晰。** 开始时比清晰图软 3 个 mip，`bias = 3 * (1 - R)^0.8`，再收到锐利。Halftone 和 Denoise 自己有清晰度过程，这里不要再叠一层。
- **边缘像玻璃斜边。** 画框内侧最后 26px，采样点再往里偏 `bend * 2 * (1 - R)^0.7 * (1 - d / 26)^2` 像素，bend 默认 6。R = 0 且贴着边缘时，偏移是 12px，和作者站点写的「最边缘最多 12px」是同一个默认。边缘那条线本身不动，是画面被拉向边缘。出图前缘内侧 20px 用同一套折射：`bend * exp(-(d / 14)^2)`，沿前缘有一层慢的噪声起伏，幅度 ±67%，R 过了 0.85 就收掉。
- **色散只跟着位移走。** 采样点被动过的地方，红色再沿位移多走 30%，蓝色少走 30%。画面没动的地方不分色。不要在边缘另描一圈彩色线，分色属于画面本身。
- **前缘一条细白线。** 深色强度 0.55，浅色 0.22，同样在 R 过 0.85 后淡去。白线保持白色。

### 九种出图，以及和点阵的默认配对

出图用遮罩 a 把画面从点阵上打开，R 是缓动后的进度。默认配对和「怎么打开」作者站点与 Prompt 一致。任意点阵仍可以配任意出图。Halftone、Dissolve、Denoise、Cell bloom 最不挑点阵。

| 出图 | 默认点阵 | 怎么打开 |
|---|---|---|
| Fisheye 鱼眼 | Ripple | 圆从中心长开。前缘在半对角线的 `1.3R - 0.1` 处。圆内按鱼眼去读：`中心 + d * (1 - k + k * u^2)`，`u = 距离 / 前缘`，`k = 0.5 * (1 - R)`。前缘后面有一道会衰减的波，红蓝往边缘分开，最后都收回平。这道波的振幅和频率 Prompt 没有写 |
| Halftone 网点放大 | fBm breath | 每个点先取自己那一格的平均色，胀到点距的一半，圆相接；后半段从 4.5 个 mip 的软图对焦到清晰 |
| Unzip 拉开 | Double helix | 从水平中线向上下打开：`\|y\| < R * (半高 + 30px) + 22 * sin(x / 44 - 1.8t) * (1 - R)`。后面这项和双螺旋用同一套单位（画框高度 = 306），边会跟着螺旋浪，再收直 |
| Denoise 去噪 | Static grain | 从 6 个 mip 收到清晰，指数 `(1 - R)^1.4`。上面盖 1.5px 的颗粒，幅度 `±45% * (1 - R)^2`，每秒刷新 24 次，像扩散反过来走 |
| Spiral 螺旋擦除 | Spiral arms | 一条臂从中心扫开。`key = 0.72 * r / 半对角线 + 0.28 * 角度 / 2π`，`key < 1.12R - 0.06` 的地方打开 |
| Dissolve 溶解 | Domain warp | 一张平滑的两八度噪声，启动时以一半尺寸画一次。噪声低于 `1.3R - 0.15` 的地方打开，软边 ±0.03（噪声值，不是像素） |
| Interference 干涉 | Sine plasma | 用和 Sine plasma 同一组四正弦当阈值，并且继续在动。低于 `1.25R - 0.1` 的地方打开 |
| Stream 逐行 | Light band | 行从顶上往下到。前缘在高度的 `1.22R - 0.06`。前缘往上 16% 的带子仍然是粗块：接到已经完成的行处是 2px，到前缘长到 16px，像流式图片在填 |
| Cell bloom 细胞绽开 | Snowflakes | Voronoi 格子大约是画框高度的五分之一。每格在 R 的前 55% 里随机选一个时刻，从种子长出圆 |

### 冲击波（只在三种出图上）

Halftone grow、Denoise、Cell bloom 出图时，一道环从中心推到 1.25 倍半对角线，环宽是 0.12 倍半对角线。令 `x = (距离 - 环半径) / 环宽`，`z = 0.75x`，画面沿径向被推开 `bend * 1.6 * 2.33 * z * exp(-z^2) * A`：环的外侧被拉开，内侧被挤紧，色散仍按上一节的 30%。`A = smoothstep(0, 0.08, R) * (1 - smoothstep(0.55, 1, R))`，也就是开头很快起来，从 R = 0.55 收到结束。

环上有一层很淡的四色光，由内到外 `#508CFF`、`#A06EFF`、`#FF6EB4`、`#FFBE8C`，彼此软接。强度 `A * exp(-0.5 * x^2) * 0.16`（深色，screen，是加光）或 `0.12`（浅色，一层染色）。只染已经显出来的画面，不照到还没显影的点上。

### 如果产品本来就在陈列这些选项

这一段只在产品有选择器或设置页要展示全部点阵和出图时才用，来自官方 Prompt。点阵做一排标签，出图在下面排成一行，每组默认配对用一条淡的 S 形线连上。一张图进来时，一条亮线沿这根线从点阵走到出图，头上有一个点，出图的名字跟着亮，亮度是这段行程的 p²。这里的 p 是亮线自己的进度；Prompt 没有再定义它怎么从 0 走到 1，也不要把它和点中心那个 p 当成同一个量。

### 亮色和暗色

演示用的一套颜色，来自官方 Prompt。产品有自己的颜色，用产品的；点保持中性。

| 主题 | 页面 | 卡片 | 画框 | 点 |
|---|---|---|---|---|
| 深色 | `#0b0b0c` | `#161618` | `#0e0e0f` | `#f2f2f2` |
| 浅色 | `#f7f7f7` | `#ffffff` | `#f4f4f4` | `#1c1c1c` |

主题切换用 480ms 交叉淡化，点、画框和光晕跟着走。

### 做得便宜，并尊重减弱动效

这些来自官方 Prompt，作者站点做了同样的压缩。

- 点阵先算进一张小纹理，一个点一个 texel，再按画框的真实尺寸去采样。不要在每个屏幕像素上算噪声。
- 不会变的遮罩（溶解那张噪声）在开始时以一半尺寸画一次。
- 图片已经完全落下、也不在淡出，就画完最后一帧停掉。这一帧之后不做 GPU 的事。
- 画布只盖住图片框，像素密度最高 2×。画框不在视口里，什么都不画。
- 系统打开减弱动效：不呼吸（停在一幅静止的点阵上），不做放大、折射、色散和冲击波，图片用 200ms 淡入。

### 调参面板

一行一个数：名字、滑块、带单位的当前值。拖动马上反映到画面上。每种点阵自己记一套值，当前标签决定面板上显示哪一套。双击某一行，这一行回到该点阵的默认。Reset 只在有人动过之后出现，按下后整个点阵回到默认。产品有滑块就用产品的，并沿用它的键盘操作；Prompt 只要求面板能用键盘核对，没有另给按键。没有滑块时，用一根带填充的 range：直立的手柄，旁边一个数字框。

全局默认（括号里是范围）：

| 项 | 默认 | 范围 |
|---|---|---|
| Breathing 呼吸 | 100% | 0–300 |
| Reveal time 出图时长 | 1400ms | 400–3000 |
| Dot pitch 点距 | 20px | 12–36 |
| Dot size 点大小 | 15% | 5–30 |
| Noise scale 噪波尺度 | 180px | 80–360 |
| Zoom in 放大 | 10% | 0–30 |
| Blur in 模糊 | 50% | 0–100 |
| Dispersion 色散 | 30% | 0–100 |
| Edge bend 边缘折射 | 6px | 0–20 |
| Edge glow 边缘光 | 30% | 0–100，仅深色 |
| Dot glow 点光晕 | 40% | 0–100，仅深色 |
| Theme 主题 | Auto | Auto / Dark / Light |

和式子对得上的几项：呼吸 100% 就是式子里的 t（0 为停住，300 为三倍快，改值时相位不跳）；点大小 15% 就是半径上限占点距的比例；放大 10%、色散 30%、折射 6px、出图 1400ms、噪波尺度 180px、点距 20px，就是上面的那些默认。模糊滑块默认 50%，而出图公式写的是 3 个 mip；百分比怎样乘到 mip 上，Prompt 没有写，不要自己补一条曲线。Halftone 的 4.5 mip 和 Denoise 的 6 mip 是它们自己的对焦，不是这个滑块。边缘光 30% 和点光晕 40% 是强度；形状仍按上面的高斯和两层边缘光，百分比的乘法 Prompt 也没有写。

每种点阵在全局默认之上再改这些，没写到的保持全局：

| 点阵 | 改掉的默认 |
|---|---|
| fBm breath | 出图 1600ms，放大 8%，模糊 30%，折射 8px |
| Domain warp | 噪波尺度 220px，呼吸 120%，出图 1600ms，模糊 70% |
| Snowflakes | 点距 15px，点大小 14%，出图 1800ms，色散 35%，折射 8px |
| Ripple | 呼吸 90%，出图 1600ms，放大 6%，色散 35% |
| Light band | 呼吸 110%，出图 1300ms，放大 8%，折射 8px |
| Sine plasma | 噪波尺度 200px，出图 1600ms |
| Double helix | 点距 18px，呼吸 110%，色散 25% |
| Static grain | 点距 16px，点大小 13%，出图 1800ms，放大 12%，模糊 30%，色散 25%，折射 8px |
| Spiral arms | 呼吸 120%，出图 1500ms，色散 35% |

### 按平台落地

下面是官方 Prompt 的建议，先对一下产品的技术栈和版本。

- **Web（演示就是这样）。** 一张 WebGL 2 画布盖在图片框上。第一遍把点阵画进「一点一个 texel」的纹理；第二遍把视口设到每个画框，读取这张纹理、带 mipmap 的图片（模糊和网点平均色都靠它）和遮罩。整页共用一个 `requestAnimationFrame`。状态行留在 DOM 里。
- **iOS。** Metal（`MTKView`）同样两遍；或 iOS 17+ 的 SwiftUI：点用 `colorEffect`，出图、折射和色散用 `layerEffect`。
- **Android。** API 33+ 用 Compose 里的 AGSL `RuntimeShader`（`graphicsLayer` 加 `RenderEffect`）；更低版本用 OpenGL ES 3。
- **Flutter。** `FragmentProgram`，点纹理用 `CustomPainter` 画。React Native 用 `react-native-skia` 的 `RuntimeEffect`。
- **没有 GPU。** 2D canvas 上每个点画一段弧。出图用 CSS：形状用 `mask-image`，放大用 `transform: scale`，变软用 `filter: blur`。折射、色散和冲击波省掉。

### 做完对照

这些核对项来自官方 Prompt。

- 小画框和大画框上，每种点阵都还认得出来：雪花是六臂，螺旋是两股。
- 出图一定等解码；很快返回的图不闪点阵；失败时点停住，并说明原因。
- 进来时图是放大且偏软的，然后变清晰；边缘只在画面移动的地方弯曲和分色；前缘白线不带颜色。
- 冲击波的光很淡，而且不照到点上。
- 每个滑块马上生效；每种点阵记住自己的值；Reset 能把它们还原。
- 中端手机上同时几个画框仍能到 60fps；图落定之后没有 GPU 开销；浅色、深色、减弱动效、面板的键盘操作都走过。

## 实现要点（本仓库整理 / 示例实现）

下面只把门槛、点的半径和「一点一个 texel」的结构收成可抄的起点。九种点阵和九种遮罩的式子在上一节，这里不重写成一份完整 shader。`smoothstep` 用常见的三次 Hermite，Prompt 没有展开这个函数。呼吸时钟用累加，这样改速度不会把相位跳开。变量名是本仓库的。

```ts
const DOT = {
  fadeInMs: 500,
  skipIfReadyMs: 300,
  revealMs: 1400,
  ease: "cubic-bezier(0.16, 1, 0.3, 1)",
  pitch: 20,
  radiusMin: 0.55,
  radiusMaxOfPitch: 0.15,
  opacityMin: 0.1,
  noiseScale: 180,
  reducedFadeMs: 200,
};

function smoothstep(edge0: number, edge1: number, x: number) {
  const t = Math.min(1, Math.max(0, (x - edge0) / (edge1 - edge0)));
  return t * t * (3 - 2 * t);
}

function vignette(distanceToEdge: number) {
  return 0.45 + 0.55 * smoothstep(8, 72, distanceToEdge);
}

function dotRadius(v: number, pitch = DOT.pitch) {
  const maxR = pitch * DOT.radiusMaxOfPitch;
  return DOT.radiusMin + v * (maxR - DOT.radiusMin);
}

/** breathing 是 0–3，对应滑块 0–300%。时钟只往前累加。 */
function stepBreath(clock: number, dtSeconds: number, breathing = 1) {
  return clock + dtSeconds * breathing;
}

type Arrival = "skip-fade" | "reveal";

async function chooseArrival(img: HTMLImageElement, requestedAt: number): Promise<Arrival> {
  await img.decode();
  return performance.now() - requestedAt < DOT.skipIfReadyMs ? "skip-fade" : "reveal";
}

// 第一遍：每个点写一个 texel。纹理尺寸 = 列数 × 行数，不是画框像素。
// 第二遍：只在画框视口里采样这张纹理、图片的 mip，以及遮罩。
// 画框离开视口，或这张图已经落定且不在淡出：这一帧不提交。
```

状态行的扫光用两层 transform，正文那一层不参与动画，避免每帧重绘字形。窗口宽 96px、周期 1.7s，来自 Prompt。行程是这一行文字自己的宽度，由布局量出来，Prompt 没有另给像素；窗口走到 `var(--sheen-x)`，里面的副本走到相反数，字形才和底下的正文对齐。两端要柔和，柔边有多长 Prompt 没有写，示例里不补。淡墨和满墨用产品的文字色，Prompt 只说了「偏淡」和「满墨」，没有给色值。

```css
.status {
  position: relative;
  font-size: 20px;
  color: var(--dot-ink-dim); /* 产品的次级文字色 */
}
.status__window {
  position: absolute;
  top: 0;
  left: 0;
  width: 96px;
  overflow: hidden;
  color: var(--dot-ink); /* 产品的正文色 */
  transform: translateX(var(--sheen-x));
}
.status__window-text {
  transform: translateX(calc(var(--sheen-x) * -1));
}
```

`--sheen-x` 在 1.7s 里从 0 走到这一行的宽度，再回到 0。减弱动效时停掉扫光，点阵停在当前图案上，出图改成 200ms 淡入，并去掉放大、折射、色散和冲击波。

## 可直接用的 Prompt（精简版）

> Wire a procedural dot-matrix wait and reveal into image generation this product already has. While a picture is being made, its box fills with a square grid of dots that breathe in one of nine patterns, under a status line with a sheen. When the picture is ready it comes in through one of nine reveals, a little large and soft at first, then sharp, with a glass-like bend at the moving edges and a colour split only where the picture actually moves. Three reveals also push a shockwave. Any pattern can pair with any reveal. Put the demo numbers on sliders so they can be tuned live. Every number is a default from a working demo, not a hard rule. Keep this product's layout, palette, slider and wording; keep the dots a neutral ink.
>
> Look first. Find where a request starts, where the result arrives (URL, blob, base64 or stream), whether progress or a partial image is reported, and which component draws the box. Also find how the project draws, its framework and versions, its sliders, dark mode and reduced-motion setting. Tell me that in a few lines before you ask anything.
>
> Then ask one question at a time, with the recommended answer marked: (1) Where it goes — every place a generated image appears (recommended; list them), or only the places I name. (2) Which pair — one pair everywhere (recommended: fBm breath with Halftone grow), a pair per place, or a different pair for each new image. (3) The settings panel — development only (recommended), behind a toggle, or for everyone. Repeat the answers, say which platform path you will use, and wait for a go-ahead.
>
> Timing. Dots fade in over 500ms. Status copy is the product's own; if it has none, start from "Making the first draft…", then "Refining the details…" after 1.3s, then "Done" once decode finishes and the reveal starts. Open the reveal only after decode (`image.decode()` or the equivalent), never on the response alone. Run it 1400ms with cubic-bezier(0.16, 1, 0.3, 1); call the eased progress R. If decode finishes within 300ms of the request, skip the dots and fade the picture in — the author's tip 017 uses the same waiting-state rule, and a flash of loading is worse than none. The prompt does not give that fade a duration. On failure, freeze the dots, say what happened, and offer a retry. Several images at once use uneven delays, for example 0, 560, 220, 820, 380, 1040, 120, 700, 300ms across a 3×3, not one shared beat and not a sweep.
>
> The box keeps the image's own aspect. In the demo card (600×772) the card radius is 28px and the box radius is 14px, concentric. Dots sit on a 20px grid, centred. Each dot's v (0 to 1) is computed once per dot: radius from 0.55px to 15% of the pitch, opacity from 0.1 to 1, vignette `0.45 + 0.55 * smoothstep(8, 72, distance to the edge)`. Dark theme only: a halo of sigma 0.3 × pitch and strength v², summed with the eight neighbours, plus two rim glows that do not swell together (outer haze `exp(-d/12)` and a line `exp(-d/2.5)`, breath 2.4s, two arcs clockwise over 8s; inner band about 6px in, Gaussian 3.5px wide, breath 3.7s from the opposite phase, three arcs anticlockwise over 11s). Both rims fade over the first third of the reveal. Light theme: dark dots, no glows, a softer rim. The status line is 20px in a dim ink; a 96px window of full ink, soft at both ends, crosses it every 1.7s. Move the window and a copy of the text inside it with opposite transforms so the glyphs never repaint.
>
> Patterns take the dot centre p and a breathing clock t. Scale t by the Breathing slider without jumping the phase. n is four-octave value-noise fBm at p / scale (180px) and t × 0.32. fBm breath is `smoothstep(0.28, 0.74, n)`. Keep the other eight in their published shapes: Ripple rings, three Spiral arms, Domain warp drifting one way like smoke (`drift = (0.22t, -0.1t)`), four-sine Sine plasma, a Light band 0.15 wide sweeping diagonally every 3.3s, a Double helix of two strands in a space where the box height is 306 with rungs on every second column, Static grain that eases about 2.5 times a second and mixes 42% over n, and six-armed Snowflakes (arm at least four dots, fall 18px/s, finer snow behind at twice the speed). Do not invent another pattern.
>
> Every reveal shares this arrival. Sample about the centre at `1 + 0.10 * (1 - R)`; the box itself does not scale. Blur from 3 mip levels, `bias = 3 * (1 - R)^0.8`, except Halftone and Denoise, which sharpen on their own. In the last 26px, read further in by `bend * 2 * (1 - R)^0.7 * (1 - d / 26)^2` with bend 6 (12px at the rim when R is 0). The 20px inside a front bends by `bend * exp(-(d / 14)^2)` with a slow ±67% wobble, gone after R passes 0.85. Where the sample moved, read red 30% further along that move and blue 30% less. A plain white front (0.55 on dark, 0.22 on light) fades on the same 0.85. No coloured stroke.
>
> Default pairs, still freely mixable: Ripple with Fisheye, fBm breath with Halftone grow, Double helix with Unzip, Static grain with Denoise, Spiral arms with Spiral wipe, Domain warp with Noise dissolve, Sine plasma with Interference wipe, Light band with Top-down stream, Snowflakes with Cell bloom. Halftone, Dissolve, Denoise and Cell bloom travel best. Shockwave only on Halftone grow, Denoise and Cell bloom: a ring out to 1.25× the half-diagonal, 0.12× wide, stretch ahead and squeeze behind, faint glow inside-to-out `#508CFF #A06EFF #FF6EB4 #FFBE8C`, only on pixels already open, never on the dots.
>
> Stay cheap. One texel per dot, then a full-size pass that only reads it. Draw the dissolve map once at half size. Stop once the picture has settled. The canvas covers the boxes only, at most 2×, and draws nothing off screen. Reduced motion: a still pattern, no zoom, bend, dispersion or shockwave, a 200ms fade. One slider row per number, live; each pattern keeps its own values; double-click a row to restore it; Reset shows up only after a change. Build from Breathing 100% (0–300), reveal 1400ms (400–3000), pitch 20px (12–36), dot size 15% (5–30), noise scale 180px (80–360), zoom 10% (0–30), blur 50% (0–100), dispersion 30% (0–100), edge bend 6px (0–20), edge glow 30% and dot glow 40% (dark only), theme Auto / Dark / Light. Per pattern, on top of those: fBm breath reveal 1600, zoom 8, blur 30, bend 8; Domain warp scale 220, breathing 120, reveal 1600, blur 70; Snowflakes pitch 15, dot 14, reveal 1800, dispersion 35, bend 8; Ripple breathing 90, reveal 1600, zoom 6, dispersion 35; Light band breathing 110, reveal 1300, zoom 8, bend 8; Sine plasma scale 200, reveal 1600; Double helix pitch 18, breathing 110, dispersion 25; Static grain pitch 16, dot 13, reveal 1800, zoom 12, blur 30, dispersion 25, bend 8; Spiral arms breathing 120, reveal 1500, dispersion 35. The blur slider is not given a formula that turns 50% into the 3 mip levels — do not invent one. Web: one WebGL 2 canvas, two passes, status line in the DOM. iOS: Metal, or SwiftUI `colorEffect` / `layerEffect` on iOS 17+. Android: AGSL on API 33+, otherwise OpenGL ES 3. Flutter `FragmentProgram`, or react-native-skia. Without a GPU: one arc per dot, CSS mask, scale and blur, and leave out bend, dispersion and shockwave.
>
> Check that each pattern still reads on a small box, the reveal waits for decode, a fast image skips the dots, a failure freezes them and explains, colour splits only where the picture moves, the front line stays white, the shockwave never lights the dots, sliders are live and per pattern, Reset restores them, a mid-range phone holds about 60fps with several boxes and does no GPU work once they settle, and light, dark, reduced motion and keyboard use of the panel all work.

上面是改写，不是原文。完整 Prompt 在[作者回复](https://x.com/Kubai087/status/2125220175591145682)，作者站点同一条也放了全文：https://vibedesign.tips/zh/024-image-loading/

## 常见错误

- 还没看现有的生图流程、也没等那三个问题得到回答，就开始写。
- 网络一返回就出图，解码还没完成。
- 300ms 内已经好的图仍先闪一下点阵。把减弱动效的 200ms 拿去当成这次跳过的淡入时长。
- 失败时把点清掉，或不说明原因、不给重试。
- 一批图同时出现，或排成一次整齐的扫过。
- 在每个屏幕像素上算噪声，或溶解遮罩每帧重画。
- 画布比图片框大、密度超过 2×，或画框离开视口仍在画。图片落定之后循环还在跑。
- 缩放的是画框，不是画面采样。Halftone 或 Denoise 上又叠了共用的那层 3 mip 模糊。
- 给模糊滑块和 3 个 mip、或给光晕百分比，补一条 Prompt 里没有的换算。
- 静止的区域也分色，或沿边缘描一圈彩色线。前缘白线被染了色。
- 冲击波的光打在还没显影的点上，或加到了另外六种出图上。
- 浅色模式仍画点光晕和边缘光。
- 九种点阵共用一套滑块值。Reset 一开始就显示，或不能回到该点阵自己的默认。
- 双螺旋、拉开的浪，把「画框高度 = 306」那套单位当成像素。雪花的臂短于四个点，小画框里变成一块斑。
- 减弱动效时点还在呼吸，或仍做放大、折射、色散、冲击波。
- 没有 GPU 仍硬做折射、色散和冲击波。
- 产品已有颜色、圆角、滑块和文案，却改成演示里的 `#0b0b0c` 一套，或把 “Making the first draft…” 写成必须出现的句子。
- 产品并没有在陈列这些选项，却加上 S 形连线和跑动的亮点。
- 把只在约 51 秒演示视频里出现的细节写成规则。
- 把 300ms 这条写成对仓库里 017 文件的依赖。它只是同一条等待原则，本分支没有那篇文件。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2125220063301271808 （2026-11-24 13:28:42 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-24）
- Prompt 回复：https://x.com/Kubai087/status/2125220175591145682 （2026-11-24 13:29:09 UTC）
- 演示回复：https://x.com/Kubai087/status/2125234339374137635
- 作者站点同一条（中文页，含完整 Prompt）：https://vibedesign.tips/zh/024-image-loading/
- 英文页同一条：https://vibedesign.tips/024-image-loading/ （标题 Dot-matrix loading and reveal animations for AI images）
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 没有英文原帖。介绍帖附约 51 秒演示视频。只在视频里的细节没有写成新规则。
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例里的变量名、`smoothstep` 的展开、呼吸时钟的累加，以及状态行用 `--sheen-x` 对向平移的写法，是本仓库整理 / 示例实现。
