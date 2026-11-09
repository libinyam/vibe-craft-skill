# 014 · 让元素离场更生动：卡片删除效果
**Four card-delete exits: Burn, Plume, Blocks, and Peel**

> 分类：质感效果 Texture & Effects · 原帖：[介绍 2026-11-09](https://x.com/Kubai087/status/2119831380276523207) · [Prompt](https://x.com/Kubai087/status/2119831540301692975) · 作者 [@Kubai087](https://x.com/Kubai087) / 枯白
>
> 本文是对原帖说明与 Prompt 的改写整理，不是原文。数值与公式取自原帖；「实现要点」代码为本仓库示例。介绍帖附约 13 秒演示视频，观感以视频为准。介绍帖下还有另外两条回复，这次没有打开，若其中另有约束，以原帖为准。

## 原理

删掉一张卡片不该只是淡出。原帖做了四种离场，并且旁边的卡片会接住这件事：火会照亮、热浪会把它们扭一下，撕贴纸时它们会被拽过去，碎块砸出去时它们会晃。空出来的位置用弹簧补上，不要去动画高度。

四种效果对应介绍帖里的四个名字：

| 效果 | 介绍帖 | 适合 |
|---|---|---|
| Burn | 燃烧 | 一次终结感很强的删除 |
| Plume | 火焰 | 轻、或带正面意味的离开（完成、归档） |
| Blocks | 碎块 | 俏皮 |
| Peel | 撕贴纸 | 本来就像贴纸的东西 |

一个产品通常只留一种。数据在按下时就按现在的逻辑删掉；画面只是「它正在离开」的图像，不能挡住输入，也不能等动画结束才提交。这是改现有列表，不是做一条演示：不要手机外框、背景板、示例卡片或示例字体，静止时列表的长相不变。

## 规则 / 数值

下面的像素按大约 432×130 的条目来。先看代码（第 0 节），再用几行字说明看到了什么，然后一次问一件事，每次都带上默认，等回答：

1. **哪里删。** 默认是条目自己的删除按钮。滑动删除和批量删除默认保持现状，或走第 8 节的淡出。有没有撤销。
2. **哪一种。** 按上表挑一种，并说明它配不配这个条目的形状和大小。
3. **邻居。** 周围的条目要不要反应（默认要）。热浪要不要弯它们，以及在这个平台上要花多少。
4. **颜色和模式。** 贴纸背面用条目自己的分类色或品牌色。Plume 的火焰色用原帖给的，或从现有色板来。应用有哪些外观模式。

然后提出接法：改哪些文件、挂在列表的哪一步、怎么截条目、用什么画、成本和回退。等同意再写。原帖第 9 节是起点，不是处方。

### 0. 从现有界面出发

找到列表、条目组件、移除发生的位置、现在怎么补位、有没有现成的离场或列表动画库。补位的弹簧若那个库做得到，就用它。

读条目真实的尺寸、圆角和圆角曲线、背景、边框、阴影，以及有没有图片、视频或内部滚动。找到滚动容器、会裁切或变换列表的祖先、列表上方的东西（标题、标签栏、浮动按钮），以及列表是不是虚拟化或回收的。

记下条目用什么画、产品已经带了哪些图形栈（WebGL、three.js、Pixi、Skia、Metal、Vulkan、wgpu）和平台版本范围。这决定快照和渲染器。保留现有的动效 token、颜色 token、深色模式、减少动态效果，以及测试和截图。

### 1. 套到自己的尺寸上

`k = clamp(sqrt(w * h / (432 * 130)), 0.6, 1.4)`。像素距离和火焰高度乘 k。Burn 的场本身已经相对条目，不再乘。网格格子仍是 5px 或 6px。

比大约 200×60 小很多的条目：用 Blocks，块改成 9px 和 18px，或者直接淡出。通栏表格行用 Burn、Blocks 或 Peel。Plume 需要高度才看得清，行很矮时把火焰缩短，并允许它伸进上面那一行。

凡是要算边的地方（快照、模切边、火焰裁切）都用产品自己的圆角和曲线。连续曲率用 `|x|^n + |y|^n = 1`，和 CSS 或平台 API 同一条曲线。方行圆角为 0。

效果读的是条目自己的画面，所以带色的条目在烧焦之前保持自己的色。背景是半透明或模糊的，快照里先把它压到真实的底上。

深色的起点（要肉眼再看，不是最终 token）：Burn 的焦黑在深底上会消失，抬到 `rgb(58 50 46)`，灰线保留。照在邻居上的火光大约一半强度，并且用普通混合——深色上 multiply 没有作用。Plume 的光晕 20%。Peel 的影子 50% 且为黑，背胶 12%。

Burn 和 Blocks 从真正的删除控件中心点起，控件在哪就算哪。Peel 从离控件最近的那只角掀起，走向条目的远边，方向要让翻起的那片留在屏幕里；控件在左边就把方向镜像。

画布挂在滚动容器这一层、列表外面。条目被卸掉时画布还在，并且和列表同样被裁切。列表在效果期间滚动，画布跟着动。

### 2. 四种共用

在删除处理函数里、状态改变之前：读矩形，截一张图，再藏起真条目。虚拟列表下一帧就会把格子回收，所以必须先截。

画在列表之上的一块不接输入的表面上。标题和导航留在这层表面之上，火焰可以从它们后面过去；可以短暂盖住邻居，只受滚动区域或屏幕裁切。

第一帧必须和真条目重合，包括圆角曲线，对不上就是快照错了。按 2x 或设备像素上传，带预乘 alpha；重新上色之前先解除预乘。

补位只用 transform 上的弹簧，`progress = 1 - exp(-7t) * cos(6t)`，520ms，`t` 为秒。不要动画 height 或 margin。弹簧在效果还没结束时就开始：Burn 第 900ms，Plume 第 720ms，Blocks 第 240ms，Peel 第 640ms。若列表会补进一条新的，它从低 60px 的地方来，前 40% 的时间里淡入。不要把刚删掉的文字再带回来。

只在有效果时画，结束就释放缓冲和纹理。每次删除各管各的，同时最多 3 个，再多就改用淡出。撤销时立刻停掉那一次，条目按产品原来的方式回来。

Burn 用的场（`click` 是删除按钮中心，`maxD` 是它到最远角的距离）：

```
f(p) = length((p - click) * (1, 1.15)) / maxD
     + (fbm(p * 0.018) - 0.5) * 0.42
     + (fbm(p * 0.085) - 0.5) * 0.05
     - pocket
pocket = smoothstep(0.6, 0.78, fbm(p * 0.03 + 40)) * 0.12
```

`fbm` 是五层平滑值噪声，每一层频率 ×2.03 再加偏移 17.1，权重减半。粗场去掉最后两项（第二层 fbm 和 pocket）。`d = f(p) - level` 为正表示火还没烧到。

### 3. Burn（全程 2s，火线用 1.15s）

`level = t / 1.15 * 1.4 - 0.1`。绘制顺序：背后的光、纸、火焰、粒子。

- **背后的光。** 火线周围的页面上，`rgb(255 158 56)`，强度 `exp(-abs(d) / 0.07) * near * 30%`。`near` 在条目上为 1，外面是 `exp(-distance / 26)`。level 从 1.1 走到 1.3 时这段光 fade 掉。
- **纸，由远到近。** `d` 从 0.17 到 0.05，以 55% 混向 `rgb(247 194 102)`；0.06 到 0.008 混向 `rgb(107 51 18)`；0.012 到 -0.006 混向 `rgb(13 9 8)`。焦黑要干净：一条约占场宽 5%（大约 20px）的带子，从暖黑到 `rgb(43 31 26)`，边缘只在慢噪声上走，`charD = d + (noise(p * 0.05) - 0.5) * 0.014`。`abs(charD + 0.05) < 0.0035` 处是一条 `rgb(189 179 168)`、85% 的灰。过了这条线纸就没了，边是硬的。不要灰色斑驳，也不要冒烟的软边。
- **热在还没烧到的一侧**，焦黑保持暗。每条带子是 `exp(-(d / w)^2)`，`w` 分火线前、火线后：橙 `rgb(255 112 13)` 为 0.05 与 0.012、92%；金 `rgb(255 204 66)` 为 0.024 与 0.006、95%；线 `rgb(255 250 214)` 的 `w` 为 0.008。全部再乘 `0.72 + 0.28 * noise(p * 0.02)`，噪声以 1.5/s 移动。
- **提前点着的点。** `smoothstep(0.78, 0.84, noise(p * 0.17))`，只在 `d` 约 0.02–0.15，色 `rgb(255 184 64)`，闪烁 `0.6 + 0.4 * sin(18t + noise(p * 0.05) * 30)`。
- **火舌。** `n1 = noise(x * 0.045 + 0.6t, y * 0.02 + 6t)`，`n2 = noise(x * 0.11 + 5, y * 0.05 + 9t)`，高度 `16 + 72 * smoothstep(0.32, 0.8, n1)`。每个点向下取 5 个样本，累加 `exp(-(粗场火线 / 0.042)^2) * (1 - smoothstep(0, tall, s)) * 0.5`，只算落在条目上的样本（到边的距离用 `smoothstep(2, -6, …)`）。封顶 1，再乘 `smoothstep(0.2, 0.62, n1 * 0.7 + n2 * 0.3)`，让火分成一舌一舌。颜色从 `rgb(255 107 18)` 走到 `rgb(255 194 51)`（0.2–0.55），再到 `rgb(255 250 214)`（0.55–0.9）。alpha 为 `smoothstep(0.03, 0.62, flame) * 95%`。低端设备可以只采样 3 次，或把这一层画在半分辨率。
- **纸是 6px 的网格。** `lift = clamp(1 - (f - level) / 0.1, 0, 1)^2`。沿离开点击点的方向缩回 `9px * lift`，并升起 `12px * lift^2`，颜色收到 `rgb(209 194 173)`。
- **64 颗粒子**，火线烧到才出发，活 0.5–1.2s。80% 是 2.5–4.5px 的火花，`rgb(255 158 41)` 收到一个浅核。其余是 3–8px、60% 的浅灰屑，轮廓不规则。上升 60–240px（火花 ×1.2，灰屑 ×0.8），横向 `sin(6 * age) * 12 * age` 再加至多 20px，从寿命的 45% 起淡出。

### 4. Plume（1.4s）：一团软火从下往上收走

纸从下往上走。线在 0.84s 到达顶部，从 0.04s 起 ease-in-out。边缘大约游荡 17px，半程时中线比两侧最多超前 80px。线附近 90px 内，纸暖到 `rgb(255 212 173)`，最多上移 20px，向中间收 20% 并晃，像被吸进火根。最后 10px 淡掉。

只有一团火，一条着色器盖住条目，从底边中点起。根要窄：半宽 36–62px，再加 `0.75 * 高度`，所以越往上越张开。最宽处不要在底部，否则像一块板。高度 90–300px，0.16s 内长出，并大约脉动 10%。越高摆得越多：`sin(3.6t + up * 0.012) * up * 0.08 + sin(6.1t + up * 0.03) * up * 0.03`。`fan = (x - 中线 - sway) / (半宽 + up * 0.75)`。

形状：`m = 1 - length(fan, rel) - (v.x - 0.35) * 0.3`，`m > 0` 才画。冠上有尖舌，`rel = (up / height) / (0.86 + 0.5 * c1^2 + 0.18 * c2^2)`，`c1 = 1 - abs(sin(fan * 3.4 + 2.6t + sin(1.7t)))`，`c2 = 1 - abs(sin(fan * 7.1 - 4t + 1.3))`。

瓣用 Voronoi：`q = (fan * 2.4, (up + 20)^0.8 * 0.02 - 2.6t)`，再被至多 0.58 的噪声扭一下，点以 3.2 rad/s 打转。`v.x` 是到最近点的距离，`v.y` 是到第二近的。这样边是从根部升起的长曲线。底边做成扇贝：`smoothstep(0, 1, (rise + 14 - v.x * 46) / 12)`。

颜色：粉 `rgb(250 153 168)` 和橙 `rgb(255 168 87)` 落在软噪声块上，每一瓣内部更亮，朝 `rgb(255 204 163)`。`v.y - v.x` 小于 0.04–0.075 的地方是筋，`rgb(255 240 158)`。下半有一颗近白的心 `rgb(255 247 214)`。周围一圈 30% 的 `rgb(255 184 143)` 光晕。

从 0.25s 起脱落 6 片，每 0.13s 一片，椭圆从 26px 到 6px，0.55–0.85s 内上浮 120–240px。44 颗 4–12px 的浅点 `rgb(255 237 189)` 在 0.15–0.85s 之间从火里升起。

0.72s 之后快速烧尽，不要整块飞走。`gone = smoothstep(0.72, 1.16, t)`：高度 ×`(1 + 1.3 * gone^2)`，半宽 ×`(1 - 0.6 * gone)`，上方张开从 0.75 收到 0.32，摆动最多到 2.2 倍，并从根部碎开——从 `m` 里减去 `gone * (1.25 - 0.55 * rel) * (0.55 + 0.45 * noise)`，所以尖端最后才没。根只升起 `150px * gone^2`。

### 5. Blocks（1.5s）

先是 70ms 的一击：条目胀 4%，并闪到最多 55% 白，`sin(π * t / 0.07)`。

然后在 36px 网格上切三档：35% 保持整格，45% 切到 18px，20% 切到 9px。每块只画条目上自己的那一片，1px 描边 20%，底边和右边在 5px 内最多暗 36%，自身明度 92–100%，看起来是实心的。

同时从中心炸开，彼此最多错 30ms。方向：`normalize((from - center - (0, 14)) * (1, 2.2) + 至多 20px 随机)`。速度 `(200 + rand * 400 + 220 * nearness) * (1.25 - size / 72)`，再加 430 向上。重力 2300 px/s²。自旋最多 6 rad/s。

落在可见列表的底边，误差 26px 内。弹一次，速度留下 42%，横向留下 60%，自旋留下 40%。0.95–1.25s 后缩到 20% 并淡出。

补位：如果列表能从上方补一条进来，上面那条带着弹跳落下，90ms 后再从顶上掉进一条新的，全程 700ms：`(t / 0.5)^2` 落到一半，然后 10% 的一跳（t 0.5–0.76）和 3% 的一跳（t 0.76–1）。否则用第 2 节的弹簧。

### 6. Peel（1.2s，拉的过程 0.62s）

条目是 5px 的网格，绕着半径 20 的圆柱弯。圆柱的线从角上扫过。删除控件在右上时，方向是 `(-0.55, 1)` 归一化，0.62s 到达远角。线的位置是 `reach * p^2 * (1.6 - 0.6p)`。`p` 从 0 到 1，0.62s 时到达远角，此时位置正好是 `reach`（到远角的距离）。开始慢，最后背胶松开时加快，纸在脱开之前不要停住。

过了弯折、沿该方向再走 `e` 的点：弯折角从 1.3 rad 增到 1.8 rad 之前，沿圆柱走，`along = line - r * sin(e / r)`，`z = r * (1 - cos(e / r))`。再过去走一条半径 420 的大弧，从圆柱末端以同一条切线接上。两段都保持弧长，所以不拉伸；被拉起的那只角的速度是线速度的 `(1 - cos(弯折角))` 倍。透视焦距 1400，对准屏幕中心，不是条目中心。

绘制顺序：留下的背胶、还粘着的部分、影子、翻起来的那片。翻片从粘着的一端画到自由的一端，这样折痕上层在上，不需要深度缓冲。

背胶是 34% 的白，2px 网格上带斑点，脱开后再留 0.3s。影子落在屏幕上：偏移 `(0.14, 0.22) * z`，不做透视；边缘软度 `8 + 0.24 * z`；不透明度 `34% / (1 + z / 170)`，高度的前 14px 里从 0 升起。影子也落在还粘着的部分上。折痕里那一部分再暗 16%，从弯折处 16px 内淡掉。

一盏光 `(-0.35, -0.55, 0.76)`。法线 `(dir * sin(弯折角), cos(弯折角))`，背面翻转。漫反射 `0.74 + 0.26 * max(dot(n, L), 0) / L.z`。Blinn-Phong 高光指数 24、14%，前 0.5 rad 内淡入。脊上的边缘光：`(1 - abs(n.z))^5 * 50%`，也就是几乎侧对视线的地方。

背面印的是条目自己的颜色，再提艳：饱和度 ×1.5，朝白走 12%，亮度 ×1.06。外面一圈 6px 的白模切边，跟着条目自己的圆角曲线。这一圈是它读成贴纸的原因。

点按或点击时角自己掀起。只有产品里的删除本来就是拖或滑，才把线拴在手指上，角留在手指下面。

0.62s 贴纸脱开。取质心在最后几毫秒的平面速度和高度速度并保留：位移 `v * g`，`g = (1 - exp(-5t)) / 5`（空气阻力）；下落 `(1500 / 5) * (t - g)`；绕质心转 `2.4 * g` rad；0.3–0.55s 内淡出。

### 7. 邻居和热浪

每条用它在列表里的距离 `d`，影响按 `1 / |d|` 衰减。只改真条目的 transform 和绘制，不改布局，不挡输入。

- **Burn 和 Plume 的光照。** 朝火的那条边上，230px 宽的径向渐变，72% 处透明。画进条目自己的填充里——单独一层相同圆角会在角上留一圈淡边。最近的一条 42%，再下一条 10%，闪烁 `0.8 + 0.2 * sin(31t) * sin(13t)`。Burn 的光跟着火线走，Plume 的留在中部。Burn 用 `rgb(255 176 96)`，Plume 用 `rgb(255 178 150)`。同时背离火源 3px（Plume 7px），并缩小 0.6%。150ms 内进入，在 1.4s 这段的最后 350ms 退出（Plume 这段是 1.25s）。
- **热浪。** 以 170px/s 向上漂的噪声挪动邻居和标题的像素。横向最多 Burn 4px、Plume 5px，纵向大约是横向的三分之一，轮廓也一起动。下方的条目 60%；标题 Burn 50%、Plume 70%。纵向噪声周期大约 80px，只要一层，整行平滑地弯；周期再短会把单个字母撕开。只在有火的时候出现。要看得出，但不能妨碍阅读。
- **Blocks。** 冲击把它们推开再荡回来：`18px * exp(-6.5t) * sin(24t)`，倾斜 `1.1° * exp(-6.5t) * sin(20t)`，挤压 `1.8% * exp(-9t)`，从 70ms 开始。
- **Peel。** 拉的过程中朝贴纸收 6px，松开后 `exp(-7t) * cos(26t)` 荡一下。

热浪的画法按平台选，先用产品已经有的：

- 有 HTML-in-Canvas 时，把列表画进 WebGL，在火周围一个场里逐像素扭：火的盒子里最强，按 170px 的高斯衰减，向上大约是向下的两倍。这是渐进增强，DOM 布局留着。WebGL 的上传接口在 Chrome 版本之间变过，更稳的是 `drawElementImage` 到 2D canvas 再上传。
- 否则每个邻居一个 SVG 滤镜：`feTurbulence` 的 `baseFrequency` 为 `0.008 0.013`、一层；红通道对比加倍，让位移均匀；`feOffset` 把它往上送；`feDisplacementMap` 的 scale 大约是想要的像素的 2.5 倍。它不会把挪动的边抹平，所以要自己补轮廓：结果膨胀 1px 垫在下面，再裁到自身 alpha 上（模糊 0.8px，斜率加倍）。只在效果期间挂上，并在 Safari 里看过。两条路都不通，就把热浪留空。

### 8. 核对与减少动态

用真实列表、真实数据看：长文、短文、有图、第一条、最后一条、只剩一条、列表已经滚过。第一帧和真条目重合，换成图片时不跳。周围的字仍可读。效果期间列表还能用，连着删不会泄漏或叠成一堆。效果中途撤销有效。

`prefers-reduced-motion`、渲染器不可用或上下文丢失、以及同时超过 3 个：都不要效果、不要邻居反应，改为 220ms 的淡出加 4% 缩小，然后补位。

焦点移到下一条。读屏要听到这一条被删了。静止时列表和改之前逐像素一致，没有删除在进行时什么都不跑。浅色和深色都看过。已有测试仍然通过。

### 9. 按平台接

四块是一样的：一张条目的图、逐像素着色器（Burn、Plume）、真网格（Peel——折叠在一个点上有两层，逐像素扭曲做不到）、许多小四边形（Blocks 和粒子）。优先用产品已经在用的。版本和细节以原帖第 9 节为准，这里只留分叉：

- **Web。** 列表上一块 WebGL2 canvas，Blocks 和粒子用实例化四边形。已有 three.js、Pixi 或同类层就画在那里。React / Vue 在点击处理里、状态更新前读矩形并截图，补位用 Motion layout、`useLayoutEffect` 里的 FLIP，或 TransitionGroup。Svelte 用 `animate:flip`。Virtuoso、TanStack Virtual 这类要把表面放在虚拟列表外面。没有 WebGL2 时，Blocks 可以用 Canvas 2D 的 `drawImage` 切片，几百块以内；其余退回淡出。
- **快照。** 条目简单时，同步画到 2D canvas（用 `Range` 矩形读字的位置，字体、颜色、字距读 computed style）。HTML-in-Canvas 能拿到活页面，目前是较新 Chromium 的旗标，Firefox 和 Safari 还没有。html-to-image 或 modern-screenshot 在删除时截一次，要几十到几百毫秒，字体和图片要同源。html2canvas 留到最后。
- **iOS。** `ImageRenderer` 或 `drawHierarchy`。UIKit 从 data source 的删除里拿 cell 的矩形；SwiftUI 从删除动作里，在 List 上方盖一层。Burn、Plume 用 SwiftUI `.layerEffect`（iOS 17+），热浪用每个邻居的 `.distortionEffect`。Peel 要网格：Metal 或 SceneKit 平面。Blocks 用 `TimelineView` 里的 SwiftUI Canvas。iOS 17 以下：Blocks 和淡出。
- **Android。** `View.drawToBitmap` 或 Compose `GraphicsLayer.toImageBitmap()`。挂在 RecyclerView 的 ItemAnimator，或 Compose 的 `Modifier.animateItem()`，效果在 overlay 里。Burn、Plume 和热浪用 AGSL `RuntimeShader` 的 `RenderEffect`（API 33+）。Peel 用 `Canvas.drawBitmapMesh` 按层画，或 OpenGL ES 做真正的透视。Blocks 用 `drawVertices` 或按矩形 `drawBitmap`。
- **Flutter。** `RenderRepaintBoundary.toImage` 是异步的，在按下时截，或预先留一张。空隙用 `AnimatedList.removeItem`，效果放 Overlay。Burn 和 Plume 用 Impeller 上的 `FragmentProgram`。Blocks 用 `Canvas.drawAtlas`。Peel 用带纹理的 `Canvas.drawVertices`。
- **Rust。** 条目先画成纹理，再用 wgpu：实例化四边形，Peel 单独一条网格管线，着色器移植到 WGSL。Bevy 用 `Mesh2d` 加自定义 `Material2d`。egui 或 iced 走渲染器的 wgpu 回调，控件自己做不了。GPUI 画进自定义元素，否则淡出。
- **Electron / Tauri。** 走 Web。Electron 可以自带需要的 Chromium 能力。Tauri 用系统 WebView，macOS 和 Linux 上要在 WebKit 里看过 WebGL2 和热浪滤镜。
- **其他（Qt、游戏引擎、自绘）。** 同样四块，用它自己的着色器和网格。

## 实现要点（示例实现，非原帖代码）

把补位弹簧、四种效果开始补位的时刻，以及 Burn 的场收成常量。火焰、碎块和撕起的网格按上一节画，不要在这里另写一套运动。

```ts
/** 本仓库示例。t 为秒。补位只写在 transform 上。 */
export function gapSpring(t: number) {
  return 1 - Math.exp(-7 * t) * Math.cos(6 * t); // 用到 0.52s
}

export const GAP_START_S = { burn: 0.9, plume: 0.72, blocks: 0.24, peel: 0.64 } as const;

export function fitScale(w: number, h: number) {
  const k = Math.sqrt((w * h) / (432 * 130));
  return Math.min(1.4, Math.max(0.6, k));
}
```

`fbm`、火线和 Peel 的圆柱不要在示例里截断后冒充完整实现；系数都在上面，按原帖抄。

## 可直接用的 Prompt（精简版）

> Add one delete exit to the cards or list items already in my product — Burn, Plume, Blocks, or Peel — not a demo scene. Do not change what deleting does or how the list looks at rest. Look at the list first, then ask one question at a time: which delete actions, which single effect, whether neighbors react and whether heat may bend them, and which colors and modes. Snapshot the item before the state update, hide the real item, and draw on a surface mounted at the scroll container, outside the list, that ignores input. Scale distances by k = clamp(sqrt(w*h/(432*130)), 0.6, 1.4); keep mesh cells at 5 or 6px. Close the gap with the spring 1 - exp(-7t)*cos(6t) over 520ms on transforms only, starting at 900 / 720 / 240 / 640ms for Burn / Plume / Blocks / Peel. Neighbors use transforms and paint only, falling off as 1/|d|; heat drifts upward at 170px/s and stays readable. At most 3 at once, otherwise a 220ms fade and 4% shrink — the same for reduced motion. Copy the burn field, plume, blocks, peel mesh, and per-platform snapshot notes from the original prompt.

完整 Prompt 见[回复](https://x.com/Kubai087/status/2119831540301692975)。介绍帖见[原文](https://x.com/Kubai087/status/2119831380276523207)。

## 常见错误

- 做成一条带手机框的演示，或改了静止时的列表、删除的数据逻辑。
- 等动画结束才删数据，或效果挡住了输入。
- 状态更新之后才截图，虚拟列表里第一帧已经对不上。
- 用 height 或 margin 补位，条目在动画里重新折行。
- 一个产品同时上四种，或给很小的行硬做完整的 Plume。
- 热浪的噪声周期太短，把字撕开；或把火光画在单独一层上，圆角处留一圈淡边。
- 用逐像素扭曲做 Peel，折起来的两层对不上。
- 超过三个还在叠完整效果；减少动态效果时仍在烧、炸、撕。
- 效果结束不释放纹理；没有删除时画布还在跑。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2119831380276523207
- Prompt：https://x.com/Kubai087/status/2119831540301692975
- 作者：[@Kubai087](https://x.com/Kubai087)
