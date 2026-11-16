# 018 · 会倾斜、能翻转的 3D 镭射卡片
**A holographic card that tilts in 3D and can flip over**

> 分类：质感效果 Texture & Effects · 原帖：[2026-11-15](https://x.com/Kubai087/status/2121892029403922717) · Prompt：[作者回复](https://x.com/Kubai087/status/2121892403212964060) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对原帖说明和作者回复里 Prompt 的改写，不是原文，也没有照抄原版 Prompt。色标、角度、弹簧这些数来自原帖和那条 Prompt，都是建议，不是硬性规定。原帖没写成文字的细节不补。帖子时间是 2026-11-15 09:04:17 UTC，也就是洛杉矶时间 2026-11-15 01:04，按本仓库约定记为 2026-11-15。Prompt 回复在 2026-11-15 09:05:46 UTC。原帖附演示视频；只在视频里出现的画面，这里不写。「实现要点」的代码是本仓库的示例。

## 原理

改产品里已经有的那一张卡，不要新做一页演示，也不要改它原来的文字和行为。

一屏只做一张。适合门票、会员卡、积分卡、分享卡片、购买凭据、勋章、档次这一类代表身份或所有权的卡。每张都在发光，这层质感就没有意义了。

卡片朝指针倾斜，最多 12°。指针离开时轻轻回弹。没人碰时自己缓慢摆一点。

箔铺在底色上，离光点越近越亮。眩光从光点向外散开。亮着的边缘光只是靠近光的那一段，会出现，也会淡掉，不要画成一整圈实线。

窗里的图形分成不同深度，卡片转动时彼此错开。这就是视差。

点一下可以翻到背面。翻转时卡片朝观看者抬起一点，并露出 10px 的侧边。背面要用户自己定，例如二维码或条款。用户没定背面，就不要翻。

下面第 1–7 节是一套能用的「标准」。第 9 节是围着它的几档。先问，等人选，再做选中的那一档。

## 规则 / 数值

### 0. 先看现有的卡片

问任何问题之前，先在现有代码里找到这张卡，用几行写清楚：

- 组件、尺寸、圆角、图和文字、放在哪、周围是什么。
- 有没有背面。
- 画面怎么画出来的：DOM、原生视图树，还是 GPU。
- 要支持的系统版本。
- 有没有暗色。
- 鼠标和触摸怎样到达这张卡。

然后才开始问。一次只问一题。选项写成短列表，标出推荐，并带上关键数值。等人回答。对方可以选一项、改一个数，或给自己的数。

### 一次一题

1. **哪一张卡。** 会员 / 积分、票 / 通行证、勋章 / 成就、个人资料 / 收藏卡、档次。推荐：屏幕上已经代表身份或所有权的那一个元素。一屏一张。
2. **质感。** 第 3 节六种里的一种，Prism 推荐。档次或稀有度可以几种一起用。也可以用品牌色，那时要先推出底、箔、眩光、边缘光、侧边，把数值列出来再等确认。原帖没有给从品牌色换算的公式，不要自己编一套算法当成规定。
3. **手感。** 安静、标准（推荐）、活泼，见第 9 节。也可以自己给倾斜角度、弹簧和摆动。
4. **交互。** 建议的起点：指针倾斜开，闲置摆动开，点击翻转开（必须先说背面放什么），手机倾斜关。手机倾斜在 iOS 上要权限。
5. **若要翻转。** 厚度（薄 / 标准推荐 / 厚）和翻转弹簧，见第 9 节。
6. **景深。** 窗里哪几块各自一层、多深。无、弱、标准（推荐）。
7. **背景。** 页面保持原样（大多数产品推荐），或在卡片后面加缓慢的流体。

七题都有了答案之后，交一张选择和数值的表，再交一份对着当前技术栈的做法（第 8 节只是起点），写上开销和退路，然后等对方说可以做。表没通过之前不写效果代码。

开销原帖没有给毫秒或电量。计划里写清这几件就够：整张卡只由一条 `requestAnimationFrame` 驱动；离开屏幕就停；流体若做，按三分之一大小、每秒 30 次画，再放大；减弱动效、以及这个平台没有对应着色器时，退到哪一档。

### 1. 倾斜和光（标准）

指针在卡片上的位置 `(fx, fy)`，范围 0 到 1。

- `nx = fx * 2 - 1`，`ny = fy * 2 - 1`
- 目标 `rotateY = nx * 12°`，`rotateX = -ny * 12°`
- 透视 1600px，加在卡片的父级上，不要加在卡片自己身上。

倾斜用弹簧跟上目标。松手时会轻轻冲过一点再回来。加速度按秒算：

`加速度 = 140 * (目标角度 − 当前角度) − 15 * 速度`

光点就在指针上：`lx = fx * 100%`，`ly = fy * 100%`。它也用弹簧跟上，不过冲。刚度 160，阻尼 `2 * √160 ≈ 25.3`。没有指针时，光停在左上 `(34%, 22%)`，像房间里的一盏灯。

悬停强度 `on` 在指针位于卡片上时从 0 走向 1，每帧用 `1 - exp(-7 * dt)`。它把箔和眩光调亮。指针离开之后 `on` 怎么回到 0，原帖没有单独的系数。

没人碰时的摆动，`t` 是秒：

- `rotateX = 2.2° * sin(0.9t + 0.6) + 0.8° * sin(2.3t)`
- `rotateY = 3.2° * sin(0.7t) + 0.9° * sin(1.9t + 1.2)`
- 光跟着走：`lx = 34% + rotateY * 3`，`ly = 22% − rotateX * 3`

有指针时，倾斜的目标是指针那一组角度，光的目标是指针位置。没人碰时，目标改成上面的摆动，光按摆动公式走。原帖没有再给一条把两者混在一起的权重。

触摸：在卡片上拖，倾斜方式和指针相同。只在这张卡上设 `touch-action: none`。若选了手机倾斜，把设备倾角映射到同一组目标角，并夹在同一个最大角度里。原帖没有给设备角度到卡片角度的增益，不要自己写一个比例。

阴影是一块模糊的椭圆：黑色 60%，模糊 24px，宽是卡片的 84%，高 70px，在卡片下方 30px。它从抬起的那一侧挪开：`x = -rotateY * 1.8px`，`y = rotateX * 1.2px`。角度用度。

### 2. 一面的层次

从底到顶：

1. **底图。** 底色或图片。
2. **箔。** 选定的质感。离光越近越亮。
3. **印刷。** 文字、标志、窗。放在箔上面，字才读得清。
4. **眩光。** 以光点为中心的径向渐变：中心白 60%，28% 处不透明度 14%（这一档原帖没写颜色），95% 也就是最远角处黑 30%。混合模式 `overlay`。不透明度 `0.35 + on * 0.65`。
5. **边缘光。** 1.5px 的环，用 `content-box` 排除的遮罩，只留下环。整圈是很淡的底色，再在光点处叠一层边缘光的亮色，到 42% 淡掉。不透明度 `0.6 + on * 0.4`。它会来也会走，不要变成一条实线。
6. **可选的全息封印。** 44px 的圆盘。`conic-gradient` 从下面的角度起，颜色依次是 `rgb(255 154 200)`、`rgb(255 224 138)`、`rgb(164 255 207)`、`rgb(143 220 255)`、`rgb(183 155 255)`，这一组排两遍。上面再铺细环：白 35%，1px 一条、每隔 3.5px。最外 1px 白边，不透明度 60%。角度 = `atan2(ly − 50%, lx − 50%) * 1.5 + rotateY * 6`。

没在第 3 节另写的边缘光：底色是白 5%，亮色是白 50%。

圆角：440×616 的演示卡是 32px，卡片变大变小就按比例变。平台有连续曲率圆角就用，没有就用普通圆角。

每一面都要在箔最亮和最暗的时候把字读清楚。

### 3. 六种质感

第 2 节的眩光和边缘光，这里没改的就沿用。下表是每种相对第 2 节的差别。渐变全文在表后面，不缩写。

| 质感 | 油墨 | 侧边 | 光晕色相 | 箔不透明度 | 眩光 | 边缘光 |
|---|---|---|---|---|---|---|
| Prism（推荐） | `rgb(244 244 246)` | `rgb(26 42 138)` | 228 | `0.15 + on * 0.45` | 色标沿用第 2 节，不透明度改成 `0.25 + on * 0.45` | 沿用第 2 节 |
| Rainbow | `rgb(245 245 247)` | `rgb(42 44 53)` | 285 | `0.3 + on * 0.6` | 沿用第 2 节 | 沿用第 2 节 |
| Pearl | `rgb(23 21 46)` | `rgb(230 226 242)` | 300 | `0.7 + on * 0.3` | 见下文，不透明度 `0.5 + on * 0.5` | 底 `rgb(60 50 120 / 0.06)`，亮色白 95% |
| Sparkle | `rgb(238 240 255)` | `rgb(26 32 72)` | 235 | 原帖没另写整层不透明度 | 沿用第 2 节 | 沿用第 2 节 |
| Chrome | `rgb(27 29 36)` | `rgb(180 184 194)` | 200 | `0.3 + on * 0.35` | 见下文。元素不透明度没另写，沿用 `0.35 + on * 0.65` | 底白 18%，亮色白 95% |
| Etched | `rgb(237 237 238)` | `rgb(27 27 30)` | 190 | `0.25 + on * 0.75` | 色标沿用第 2 节，不透明度改成 `0.2 + on * 0.4` | 沿用第 2 节 |

#### Prism

从内部点亮的深蓝玻璃。

| 层 | 值 |
|---|---|
| 底 | `radial-gradient(60% 40% at 50% 0%, rgb(90 150 255 / 0.55), transparent)`，再叠 `linear-gradient(170deg, rgb(13 35 168), rgb(8 18 102) 55%, rgb(22 15 92))` |
| 窗 | 三层径向压在一条线性上。色标多，读起来是光，不是几块色斑。第一层 `radial-gradient(46% 50% at 光点, rgb(255 175 235) / 0.55，18% 处 0.4，38% 处 0.22，60% 处 0.09，82% 处 0.02，然后透明)`。第二层 `radial-gradient(54% 56% at (100% − lx, 74%), rgb(125 225 255) / 0.45，18% 处 0.32，38% 处 0.17，60% 处 0.07，82% 处 0.015，然后透明)`。第三层 `radial-gradient(70% 55% at 30% 100%, rgb(150 255 210 / 0.3), transparent 75%)`。底下是 `linear-gradient(170deg, rgb(59 98 230), rgb(42 63 191) 55%, rgb(58 47 168))`。18% 之后原帖没有把颜色再写一遍 |
| 窗框 | `rgb(140 170 255 / 0.1)`，加 1px 线 `rgb(170 200 255 / 0.28)`。浅色这层的宽度原帖没写 |
| 箔 | 2px 的亮边。`radial-gradient(at 光点, rgb(170 220 255 / 0.9), transparent 60%)`，遮罩成环。不透明度见上表。原帖没写混合模式 |

#### Rainbow

色带随着倾斜滑动。

| 层 | 值 |
|---|---|
| 底 | `radial-gradient(130% 80% at 25% 0%, rgb(59 62 76), rgb(27 28 35) 62%)` |
| 箔 | `repeating-linear-gradient(115deg, rgb(255 95 126) 0%, rgb(255 184 107) 2.6%, rgb(244 255 107) 5.2%, rgb(107 255 181) 7.8%, rgb(95 210 255) 10.4%, rgb(139 123 255) 13%, rgb(255 107 214) 15.6%, rgb(255 95 126) 18%)`。`background-size: 200%`，位置在光点。上面再压光栅：`repeating-linear-gradient(125deg, transparent 0 2px, rgb(0 0 0 / 0.3) 2px 3px)`。`mix-blend-mode: color-dodge`。`brightness(0.72) saturate(1.25)`。遮罩 `radial-gradient(farthest-corner at 光点, rgb(0 0 0) 5%, rgb(0 0 0 / 0.16) 72%)` |

#### Pearl

白卡，边上的粉彩光最强，像一圈框。

| 层 | 值 |
|---|---|
| 底 | `linear-gradient(160deg, rgb(253 252 255), rgb(241 238 248))` |
| 箔 | `linear-gradient(115deg, rgb(255 196 225) 0%, rgb(201 184 255) 12%, rgb(180 233 255) 24%, rgb(194 255 216) 36%, rgb(255 243 184) 48%，然后同一组再排一次)`。`background-size: 260%`，位置在光点。`multiply`。遮罩 `radial-gradient(70% 60% at 50% 45%, rgb(0 0 0 / 0.35) 30%, rgb(0 0 0) 100%)`。遮罩按透明度读：中心 0.35、边缘 1，所以光靠边缘更强。后一半色标的百分比原帖没写 |
| 眩光 | `soft-light`。白 90%，到 55% 淡出 |

#### Sparkle

细闪光。

| 层 | 值 |
|---|---|
| 底 | `radial-gradient(120% 80% at 75% 5%, rgb(40 51 106), rgb(16 20 43) 65%)` |
| 闪点 | 240px 的一块里大约 950 个点，半径 0.45px 到 1.85px，多数偏小，透明度 0.35 到 1。当作透明度遮罩。随机分布原帖只说多数偏小，没有公式 |
| 箔 | `screen`，两层，各自先被闪点遮住，再被 `radial-gradient(at 光点, rgb(0 0 0), rgb(0 0 0 / 0.3) 45%, rgb(0 0 0 / 0.12))` 遮住。第二层的闪点偏移 `(117px, 71px)`。第一层 `repeating-linear-gradient(60deg, rgb(255 139 210) 0 3%, transparent 5% 8%, rgb(139 243 255) 10% 13%, transparent 15% 18%, rgb(255 243 139) 20% 23%, transparent 25% 28%)`，大小 260%，位置 `(lx * 1.6, ly)`。第二层 `repeating-linear-gradient(-35deg, rgb(169 139 255) 0 3%, transparent 5% 9%, rgb(139 255 196) 11% 14%, transparent 16% 20%, rgb(255 255 255) 22% 24%, transparent 26% 30%)`，大小 260%，位置 `(100% − lx, ly * 1.4)`。暗缝让每颗闪点在自己的角度亮一下，再灭掉。最后一档 `rgb(0 0 0 / 0.12)` 原帖没写百分比 |

#### Chrome

银色镜面，高光逆着倾斜走。

| 层 | 值 |
|---|---|
| 底 | `linear-gradient(125deg, rgb(228 230 234) 0%, rgb(141 147 158) 16%, rgb(246 247 249) 30%, rgb(124 130 141) 46%, rgb(217 220 225) 60%, rgb(148 154 165) 76%, rgb(241 242 244) 100%)`。`background-size: 220%`，位置 `(100% − lx, 100% − ly)` |
| 箔 | `linear-gradient(115deg, rgb(255 143 199), rgb(255 226 143) 14%, rgb(149 255 208) 28%, rgb(143 214 255) 42%, rgb(192 155 255) 56%，然后同一组再排一次)`。大小 240%，位置在光点。`overlay`。后一半色标的百分比原帖没写 |
| 眩光 | `soft-light`。白 85%，30% 处不透明度 10%（颜色没写），95% 处黑 18% |
| 印刷线 | 凹刻：`text-shadow: 0 -1px 0 rgb(60 64 76 / 0.55), 0 1px 0 rgb(255 255 255 / 0.95)` |

#### Etched

只有光扫过才看得见的纹。

| 层 | 值 |
|---|---|
| 底 | 哑光 `rgb(18 18 20)`，加细同心环：`repeating-radial-gradient(circle at 窗的中心, rgb(255 255 255 / 0.05) 0 1.2px, transparent 1.6px 7px)` |
| 箔 | `linear-gradient(115deg, rgb(255 95 126), rgb(255 184 107) 10%, rgb(244 255 107) 20%, rgb(107 255 181) 30%, rgb(95 210 255) 40%, rgb(139 123 255) 50%，然后同一组再排一次)`。大小 300%，位置在光点。被同一组环遮住，再被 `radial-gradient(at 光点, rgb(0 0 0), rgb(0 0 0 / 0.5) 18%, transparent 42%)` 遮住。后一半色标的百分比原帖没写 |
| 窗 | 保持一块哑光板，环到窗为止：`radial-gradient(70% 60% at 50% 35%, rgb(29 29 33), rgb(15 15 17))`。不要套 Prism 的窗内光 |

### 4. 翻面（只有选了才做，标准）

点击、触摸、聚焦时的 Enter 或 Space，绕竖直轴转到 180°。弹簧刚度 70，阻尼 11，大约 0.8 秒停下，并有一点过冲。再操作一次转回来。

转向观看者抬起：`translateZ(|sin(翻转角)| * 40px)`。侧向时阴影最多收窄 45%，并再下落 14px。中间帧怎么插，原帖没写。

厚度 10px。正面在中面之前 5px，背面在中面之后 5px，背面转 180°。两面都 `backface-visibility: hidden`。两者之间用这种质感的侧边色均匀排 10 份轮廓。只在翻转过程中画侧边。0° 和 180° 不画。原帖没写每一份的线宽。验收时不能两面同时看见，也不能在侧边露出缝。

背面用同一套质感、眩光和边缘光，光左右对调（`lx` 变成 `100% − lx`）。边距和正面相同。

正面要换质感或换内容时，等转到侧面再换，换的过程不能被看见。

用户没选定背面放什么，就不要做翻转，也不要编一个背面。

### 5. 窗里的景深（标准）

每一块有自己的深度 `d`。正数在面之后，负数在面之前。位移是真实 3D 会有的量：

- `x = -d * sin(rotateY + 翻转角)`
- `y = d * sin(rotateX)`

下面这组是演示图的深度，不是每张产品卡的要求：

| 块 | 深度 |
|---|---|
| 图片 | 身后 90px |
| 罗盘刻度盘 | 50px |
| 中环 | 34px |
| 虚线环 | 20px |
| 指针 | 身前 12px（`d = -12`） |
| 角上的印刷 | 贴在面上，不动 |

最后面的那一层要比窗各大 100px，四边都是，这样转的时候不会露出层的边缘。

图形自己会动时才加下面这些，并且只留一个缓慢的主意。指针用弹簧朝向鼠标，走短边：`加速度 = 60 * 角度差 − 5.5 * 速度`。没人碰时指针漂移 `12° * sin(0.77t) + 6° * sin(2.16t)`。刻度盘每秒 9°，虚线环每秒 18°，方向相反。

弱档把上面每个深度减半。无档就不要视差。

### 6. 节奏和无障碍

悬停进出、倾斜、光都是连续的。翻转大约 0.8 秒停下。

换质感：两面交叉淡化 420ms，或在翻转的侧面换掉。

减弱动效：不要倾斜，不要摆动，不要翻转的动画。立刻交换，或 200ms 交叉淡化，两样原帖都允许。光仍然可以跟指针。背景停住。

卡片原来是什么就还是什么。链接还是一个链接，按钮还是一个按钮。背面要用键盘到达，并被读出来。重要的内容不能只在某一个角度才看得见。

### 7. 背景（只有选了才做）

默认不加。访谈里和「页面保持原样」相对的是缓慢的流体。光晕写在同一节，跟着选中的背景一起做。

光晕在卡片后面：`radial-gradient(closest-side, hsl(<质感色相> 85% 64% / 0.3), transparent)`，宽 860px。元素不透明度 `(0.04 + 0.12 * on) / 0.3`。

流体用值噪声做 fBm。4 个八度，每个八度的坐标乘下面这张矩阵，振幅减半：

```
|  1.6   1.2 |
| -1.2   1.6 |
```

扭曲两层。`flow.yx` 是把 `flow` 的两个分量对调。

```
p = 位置 / 高 * 1.25 + flow
q = (fbm(p), fbm(p + (5.2, 1.3) − flow.yx))
r = (fbm(p + 3.5 * q + (1.7, 9.2) + 1.3 * flow), fbm(p + 3.5 * q + (8.3, 2.8) − flow))
f = fbm(p + 3 * r)
颜色 = mix(a, b, clamp(2.4 * f²))
再按 clamp(0.95 * |q| − 0.35) 混向 c
最后乘 0.5 + 1.1 * f²
```

用 `smoothstep(0.82, 0.18, 距离)` 淡回页面底色。距离从 `(50%, 56%)` 量起，y 先乘 1.2。

`flow` 在半径 0.33 的圆上转，大约 40 秒一圈。视场再按 `(-rotateY, rotateX) * 0.008` 挪一点，让它回应倾斜。

| 质感 | a | b | c |
|---|---|---|---|
| Prism | `rgb(10 20 80)` | `rgb(58 42 156)` | `rgb(26 108 192)` |
| Rainbow | `rgb(44 12 64)` | `rgb(12 68 80)` | `rgb(90 32 52)` |
| Pearl | `rgb(51 40 94)` | `rgb(28 74 74)` | `rgb(92 44 80)` |
| Sparkle | `rgb(12 20 56)` | `rgb(38 51 122)` | `rgb(74 58 22)` |
| Chrome | `rgb(24 28 36)` | `rgb(44 54 70)` | `rgb(22 64 74)` |
| Etched | `rgb(13 15 18)` | `rgb(26 36 40)` | `rgb(38 42 50)` |

换色板大约 1 秒。画在三分之一大小的画布上，每秒 30 次，再放大。它本身是糊的，分辨率低没问题。

`clamp` 的上下界、`|q|` 对向量是长度还是分量绝对值，原帖都没写。见文末出处。

### 8. 各平台怎么落

先看项目里已经有的画法，能接上就接上。下面是起点。

- **Web（演示就是这样）。** CSS 3D。层是叠起来的元素，用 `mix-blend-mode`。倾斜写在卡片自己的 `transform` 上。光写成面自己的自定义属性，不要写成会继承的父级变量，否则改一个变量，下面整棵树都要重算样式。眩光用一块 2 倍大的层做 `transform` 来挪。箔和会动的图形各自一层，`will-change: transform`。只画朝外的那一面。数值没变就不要写回 DOM。整张卡一条 rAF。流体是一小块 WebGL canvas。
- **iOS（SwiftUI）。** `rotation3DEffect` 带透视。叠加层用 `.blendMode(.colorDodge / .overlay / .multiply)`。渐变从光点出发。倾斜和翻转用弹簧。触摸用 `DragGesture`。手机倾斜用 CoreMotion 的 `CMMotionManager` `deviceMotion`。箔也可以用 Metal 的 `[[stitchable]]` `colorEffect` 一次画完。
- **Android（Compose）。** `Modifier.graphicsLayer { rotationX; rotationY; cameraDistance = 12 * density }`。`drawWithContent` 里用 `Brush` 和 `BlendMode`。弹簧用 Animatable。拖动用 `pointerInput`。手机倾斜用 `SensorManager` 的 `TYPE_GAME_ROTATION_VECTOR`。API 33 及以上可以用 AGSL `RuntimeShader` 画箔。
- **Flutter。** `Transform` 用 `Matrix4.identity()..setEntry(3, 2, 0.0006)..rotateX()..rotateY()`。层放在 `Stack` 里，`ShaderMask` 或 `CustomPainter` 做混合。箔和流体可以用 `FragmentProgram`。手机倾斜用 `sensors_plus`。
- **Rust（wgpu、Bevy、egui）。** 一块四边形，片元着色器用光的 uniform 画箔、眩光和边缘光，再做 3D 变换。翻转和侧边是第二块四边形加一个薄盒子。

### 9. 围绕标准的几档

问的时候给出这些。日常克制：Pearl 配安静。值得庆祝的时刻再用 Rainbow 或 Sparkle，配标准或活泼。

| 档 | 值 |
|---|---|
| 安静 | 倾斜 6°，弹簧 120 / 18，看不出过冲。摆动是标准的一半，大约 1° 到 1.5°。悬停强度封顶 0.7。原帖没写是截断还是按比例缩小，也没另给正弦系数 |
| 标准（推荐） | 倾斜 12°，弹簧 140 / 15，摆动大约 2° 到 3°。公式见第 1 节 |
| 活泼 | 倾斜 16°，弹簧 180 / 12，回弹看得出来。摆动大约 3° 到 4°。原帖没另给正弦系数 |
| 薄 | 6px，6 层，抬起 30px |
| 标准厚度（推荐） | 10px，10 层，抬起 40px |
| 厚 | 16px，16 层，抬起 56px |
| 翻转弹簧，软 | 55 / 9，更慢，过冲更多 |
| 翻转弹簧，标准（推荐） | 70 / 11 |
| 翻转弹簧，脆 | 110 / 18，快，不过冲 |
| 景深，无 | 各块不分开 |
| 景深，弱 | 第 5 节每个深度减半 |
| 景深，标准（推荐） | 第 5 节 |

安静和活泼的摆动，示例里不另写系数。真要做这两档，先把打算用的系数放进选择表。

### 10. 做完核对

- 每种质感都在四角和中间看过。字读得清，眩光没有把印刷洗白，边缘光不是一条实线。
- 快速连翻很多次。不会两面同时出现，侧边也没有缝。
- 写进实现的数和确认过的表一致。
- 在一部中端手机上跟得上屏幕刷新。离开屏幕之后不再耗。
- 减弱动效和键盘都走得通。背面能被聚焦，也能被读出来。
- 亮色页和暗色页都看过。质感用选定的那一套颜色，不要为暗色页另推一套原帖没有的色。

## 实现要点（示例实现，非原帖代码）

示例只做标准手感加 Prism。另外五种的色标在第 3 节，不在这里再写一遍。流体、手机倾斜、闪点的随机分布都不在示例里。接在现有卡片上，不新建一页。

```ts
/** 标准档。其它档见第 9 节，不要把没选中的数写进来。 */
export const HOLO = {
  tiltDeg: 12,
  perspectivePx: 1600,
  tiltStiffness: 140,
  tiltDamping: 15,
  lightStiffness: 160,
  lightDamping: 25.3, // 2 * sqrt(160)，原帖给的约数
  hoverRate: 7,
  restLx: 34,
  restLy: 22,
  quietHoverCap: 0.7,
  shadowAlpha: 0.6,
  shadowBlur: 24,
  shadowWidth: 0.84,
  shadowHeight: 70,
  shadowBelow: 30,
  shadowXPerDeg: 1.8,
  shadowYPerDeg: 1.2,
  demoCard: { w: 440, h: 616, radius: 32 },
  flipStiffness: 70,
  flipDamping: 11,
  liftPx: 40,
  thicknessPx: 10,
  edgeLayers: 10,
  faceHalf: 5,
  finishFadeMs: 420,
  reducedFadeMs: 200,
} as const;

const DEG = Math.PI / 180; // 把原帖的「度」送进 sin。不是新的设计值。

export function spring(
  angle: number,
  velocity: number,
  target: number,
  stiffness: number,
  damping: number,
  dt: number,
) {
  const acc = stiffness * (target - angle) - damping * velocity;
  const v = velocity + acc * dt;
  return { angle: angle + v * dt, velocity: v };
}

/** 没人碰时。有指针就不要再叠加这组。 */
export function idleSway(t: number) {
  const rotateX = 2.2 * Math.sin(0.9 * t + 0.6) + 0.8 * Math.sin(2.3 * t);
  const rotateY = 3.2 * Math.sin(0.7 * t) + 0.9 * Math.sin(1.9 * t + 1.2);
  return {
    rotateX,
    rotateY,
    lx: HOLO.restLx + rotateY * 3,
    ly: HOLO.restLy - rotateX * 3,
  };
}

/** 演示图的视差。d 正数在面后。产品卡的分层以确认过的表为准。 */
export function parallax(d: number, rotateXDeg: number, rotateYDeg: number, turnDeg: number) {
  return {
    x: -d * Math.sin((rotateYDeg + turnDeg) * DEG),
    y: d * Math.sin(rotateXDeg * DEG),
  };
}

/** 短边，走 −180° 到 180°。 */
export function shortAngle(deltaDeg: number) {
  return ((deltaDeg + 540) % 360) - 180;
}
```

`(delta + 540) % 360 - 180` 只是把角度差折进半圈，对应原帖的「走短边」。540 不是设计值。

```ts
type Motion = {
  rx: number;
  ry: number;
  vx: number;
  vy: number;
  lx: number;
  ly: number;
  lvx: number;
  lvy: number;
  on: number;
  turn: number;
  turnV: number;
};

function frame(m: Motion, dt: number, t: number, input: {
  onCard: boolean;
  fx: number;
  fy: number;
  reduce: boolean;
  quiet: boolean;
}) {
  if (input.reduce) {
    m.rx = 0;
    m.ry = 0;
    m.vx = 0;
    m.vy = 0;
    // 翻转不做动画：直接到 0° 或 180°，或另做 200ms 淡化。两样都可以。不要把 --turn 清掉。
    if (input.onCard) {
      ({ angle: m.lx, velocity: m.lvx } = spring(m.lx, m.lvx, input.fx * 100, HOLO.lightStiffness, HOLO.lightDamping, dt));
      ({ angle: m.ly, velocity: m.lvy } = spring(m.ly, m.lvy, input.fy * 100, HOLO.lightStiffness, HOLO.lightDamping, dt));
      m.on = 1 - (1 - m.on) * Math.exp(-HOLO.hoverRate * dt);
    } else {
      m.lx = HOLO.restLx;
      m.ly = HOLO.restLy;
      m.lvx = 0;
      m.lvy = 0;
      m.on = m.on * Math.exp(-HOLO.hoverRate * dt);
    }
  } else if (input.onCard) {
    const nx = input.fx * 2 - 1;
    const ny = input.fy * 2 - 1;
    const tx = -ny * HOLO.tiltDeg;
    const ty = nx * HOLO.tiltDeg;
    ({ angle: m.rx, velocity: m.vx } = spring(m.rx, m.vx, tx, HOLO.tiltStiffness, HOLO.tiltDamping, dt));
    ({ angle: m.ry, velocity: m.vy } = spring(m.ry, m.vy, ty, HOLO.tiltStiffness, HOLO.tiltDamping, dt));
    ({ angle: m.lx, velocity: m.lvx } = spring(m.lx, m.lvx, input.fx * 100, HOLO.lightStiffness, HOLO.lightDamping, dt));
    ({ angle: m.ly, velocity: m.lvy } = spring(m.ly, m.lvy, input.fy * 100, HOLO.lightStiffness, HOLO.lightDamping, dt));
    m.on = 1 - (1 - m.on) * Math.exp(-HOLO.hoverRate * dt);
  } else {
    const idle = idleSway(t);
    ({ angle: m.rx, velocity: m.vx } = spring(m.rx, m.vx, idle.rotateX, HOLO.tiltStiffness, HOLO.tiltDamping, dt));
    ({ angle: m.ry, velocity: m.vy } = spring(m.ry, m.vy, idle.rotateY, HOLO.tiltStiffness, HOLO.tiltDamping, dt));
    m.lx = idle.lx;
    m.ly = idle.ly;
    m.lvx = 0;
    m.lvy = 0;
    // 回落沿用走向 1 的那个 7。原帖只写了指针在卡上时走向 1。这是本仓库取值。
    m.on = m.on * Math.exp(-HOLO.hoverRate * dt);
  }
  if (input.quiet) m.on = Math.min(m.on, HOLO.quietHoverCap); // 截断是本仓库取值
  return m;
}
```

光和 `on` 写在面元素上，不写在父级：

```ts
function paintFace(face: HTMLElement, m: Motion) {
  face.style.setProperty("--lx", m.lx + "%");
  face.style.setProperty("--ly", m.ly + "%");
  face.style.setProperty("--px", String(m.lx));
  face.style.setProperty("--py", String(m.ly));
  face.style.setProperty("--on", String(m.on));
}
```

离开视口就取消这一条 rAF。原帖没指定用哪种观察方式。数值和上一帧相同就不要再 `setProperty`。

圆角示例：`32 * cardWidth / 440`。原帖只说随卡片缩放，没写按宽还是按短边。这是本仓库取值。平台有连续曲率就用它。

侧边示例：10 份、每份 1px，中心从 −4.5px 排到 +4.5px，把 10px 填满。线宽和这份间距原帖没写，是本仓库取值。只在 `turn` 不在 0° 或 180° 时挂上。

翻转阴影示例：在第 1 节的偏移之外，宽度乘 `1 - 0.45 * |sin(turn)|`，再向下 `14 * |sin(turn)|` px。用 `|sin|` 插中间帧是本仓库取值。

封印角度示例：`Math.atan2(ly - 50, lx - 50) * 1.5 + rotateY * 6`，结果直接当度数。不再乘 `180 / π`。单位原帖没写。

景深演示：图片 90、刻度盘 50、中环 34、虚线环 20、指针 −12。最底层四边各多 100px。角上的字 `d = 0`。

```css
.holo-stage { perspective: 1600px; }
.holo-card {
  position: relative;
  transform-style: preserve-3d;
  transform: translateZ(var(--lift, 0px)) rotateX(var(--rx)) rotateY(calc(var(--ry) + var(--turn)));
  touch-action: none;
  /* 子层的 translateZ 和背面隐藏要父级 preserve-3d 才成立。这是 CSS 的写法，不是新的设计值。 */
}
.holo-card .face {
  position: absolute;
  inset: 0;
  border-radius: var(--card-radius, 32px);
  backface-visibility: hidden;
  color: rgb(244 244 246);
  background:
    radial-gradient(60% 40% at 50% 0%, rgb(90 150 255 / 0.55), transparent),
    linear-gradient(170deg, rgb(13 35 168), rgb(8 18 102) 55%, rgb(22 15 92));
}
.holo-card .face-front { transform: translateZ(5px); }
.holo-card .face-back {
  transform: rotateY(180deg) translateZ(5px);
  /* 背面的 --lx 用 100% − 正面的 lx。 */
}

/* Prism 的窗。18% 之后沿用同一个颜色，只改透明度，见出处。 */
.holo-card .window {
  background:
    radial-gradient(46% 50% at var(--lx) var(--ly),
      rgb(255 175 235 / 0.55),
      rgb(255 175 235 / 0.4) 18%,
      rgb(255 175 235 / 0.22) 38%,
      rgb(255 175 235 / 0.09) 60%,
      rgb(255 175 235 / 0.02) 82%,
      transparent),
    radial-gradient(54% 56% at calc(100% - var(--lx)) 74%,
      rgb(125 225 255 / 0.45),
      rgb(125 225 255 / 0.32) 18%,
      rgb(125 225 255 / 0.17) 38%,
      rgb(125 225 255 / 0.07) 60%,
      rgb(125 225 255 / 0.015) 82%,
      transparent),
    radial-gradient(70% 55% at 30% 100%, rgb(150 255 210 / 0.3), transparent 75%),
    linear-gradient(170deg, rgb(59 98 230), rgb(42 63 191) 55%, rgb(58 47 168));
  border: 1px solid rgb(170 200 255 / 0.28);
  /* 窗框浅色 rgb(140 170 255 / 0.1) 的宽度原帖没写，这里不另造。 */
}

.holo-card .foil,
.holo-card .rim {
  position: absolute;
  inset: 0;
  pointer-events: none;
  border-radius: inherit;
}
.holo-card .foil {
  padding: 2px;
  background: radial-gradient(at var(--lx) var(--ly), rgb(170 220 255 / 0.9), transparent 60%);
  opacity: calc(0.15 + var(--on) * 0.45);
}
.holo-card .rim {
  padding: 1.5px;
  background:
    radial-gradient(at var(--lx) var(--ly), rgb(255 255 255 / 0.5), transparent 42%),
    rgb(255 255 255 / 0.05);
  opacity: calc(0.6 + var(--on) * 0.4);
}
.holo-card .foil,
.holo-card .rim {
  mask:
    linear-gradient(#000 0 0) content-box,
    linear-gradient(#000 0 0);
  mask-composite: exclude;
}

/* 印刷放在箔之后、眩光之前，靠文档顺序压在箔上面。原帖没给 z-index。 */

.holo-card .glare {
  position: absolute;
  left: 0;
  top: 0;
  width: 200%;
  height: 200%;
  pointer-events: none;
  /* 0.5 来自这层是卡片的 2 倍：用层自己的百分比，把中心移到光点。原帖没写这个式子。 */
  transform: translate(calc((var(--px) - 100) * 0.5%), calc((var(--py) - 100) * 0.5%));
  /* 28% 那一档原帖只写了不透明度 14%。这里用白色，是本仓库取值。 */
  background: radial-gradient(farthest-corner at 50% 50%,
    rgb(255 255 255 / 0.6),
    rgb(255 255 255 / 0.14) 28%,
    rgb(0 0 0 / 0.3) 95%);
  mix-blend-mode: overlay;
  opacity: calc(0.25 + var(--on) * 0.45); /* Prism。第 2 节默认是 0.35 + on * 0.65 */
}

.holo-shadow {
  position: absolute;
  width: 84%;
  height: 70px;
  left: 8%; /* 本仓库取值：按 84% 宽度居中。原帖没写水平位置。 */
  top: calc(100% + 30px); /* 本仓库读法：椭圆顶边在卡片底边下 30px。 */
  border-radius: 50%;
  background: rgb(0 0 0 / 0.6);
  filter: blur(24px);
  pointer-events: none;
}
```

减弱动效时，JS 把倾斜和摆动保持在 0，翻转用瞬间交换或 200ms 淡化。光仍可以跟指针。流体若有，停住。

## 可直接用的 Prompt（精简版）

> Turn one existing product card into a holographic card inside the current code and design system. Do not change what the card says or does. Do not build a demo page. One holographic card per screen. Every number below is a recommendation.
>
> Before asking, report in a few lines: the component, size, corner radius, artwork, content, placement, surroundings, whether a back already exists, how the screen is drawn, the platform version range, dark mode, and how pointer and touch reach the card.
>
> Then ask one question at a time. Give a short list, mark the recommendation, and wait. The person may pick, tweak a value, or supply their own.
>
> 1. Which card stands for status or ownership: member or loyalty, ticket or pass, badge or achievement, profile or collectible, plan tier. One per screen.
> 2. Finish: Prism (recommended), Rainbow, Pearl, Sparkle, Chrome, or Etched. Several may map to tier or rarity. Brand colors are allowed only after you derive base, foil, glare, rim, and edge and show those values. There is no conversion formula to invent.
> 3. Feel: Quiet (tilt 6deg, spring 120/18, sway about 1 to 1.5deg, hover strength capped at 0.7), Standard (recommended: tilt 12deg, spring 140/15, sway about 2 to 3deg), or Lively (tilt 16deg, spring 180/12, sway about 3 to 4deg).
> 4. Interactions, suggested start: pointer tilt on, idle sway on, flip on click only if they name the back (QR or terms, for example), phone-motion tilt off. Phone motion needs permission on iOS.
> 5. If it flips: thickness Thin 6px / 6 layers / lift 30px, Standard 10px / 10 layers / lift 40px (recommended), or Thick 16px / 16 layers / lift 56px. Flip spring Soft 55/9, Standard 70/11 (recommended), or Crisp 110/18.
> 6. Depth: None, Subtle (halve each depth), or Standard (recommended).
> 7. Background: leave the page as it is (recommended), or add the slow fluid.
>
> Then show one table of the chosen values, a build plan for this platform with cost and a fallback, and wait again. Do not start the effect before that table is approved.
>
> Standard motion: pointer (fx, fy) in 0..1 gives nx = fx*2-1, ny = fy*2-1, target rotateY = nx*12deg, rotateX = -ny*12deg. Perspective 1600px on the parent. Tilt acceleration per second = 140*(target-angle)-15*velocity. Light sits on the pointer and springs with stiffness 160 and damping 2*sqrt(160) ≈ 25.3, no overshoot. With no pointer it rests at (34%, 22%). Hover strength `on` eases toward 1 by 1-exp(-7*dt) while the pointer is on the card. Idle sway, t in seconds: rotateX = 2.2deg*sin(0.9t+0.6)+0.8deg*sin(2.3t), rotateY = 3.2deg*sin(0.7t)+0.9deg*sin(1.9t+1.2); lx = 34%+rotateY*3, ly = 22%-rotateX*3. Dragging tilts the same way; touch-action: none on the card only. Shadow: black at 60%, blur 24px, 84% of the card width, 70px tall, 30px below, x = -rotateY*1.8px, y = rotateX*1.2px.
>
> Layers back to front: art, foil, print (text, logo, window; above the foil), glare, rim, optional 44px seal. Default glare is a radial gradient on the light: white at 60% in the middle, 14% opacity at 28%, black at 30% at 95% / the farthest corner, overlay, opacity 0.35+on*0.65. Default rim is a 1.5px ring masked with content-box exclude, base white at 5% unless a finish says otherwise, light color white at 50% unless a finish says otherwise, fading out by 42%, opacity 0.6+on*0.4. Corners are 32px on a 440×616 card and scale with the card. Use a continuous curve where the platform has one.
>
> Use the finish tables for the chosen preset. Prism is deep blue glass: base radial rgb(90 150 255 / 0.55) at 50% 0% over linear-gradient(170deg, rgb(13 35 168), rgb(8 18 102) 55%, rgb(22 15 92)); ink rgb(244 244 246); foil is a 2px ring, radial rgb(170 220 255 / 0.9) to transparent at 60%, opacity 0.15+on*0.45; glare opacity 0.25+on*0.45; edge rgb(26 42 138); glow hue 228. Do not approximate the other finishes. Copy their stops. If a stop is written as "the same sequence again", do not invent the second set of percentages.
>
> Flip only when a back was chosen. Spring 70/11, about 0.8s, small overshoot, 180deg around Y. Lift translateZ(abs(sin(angle))*40px). Front 5px in front of the middle, back 5px behind and rotated 180deg, both with backface hidden. Ten edge-colored outlines fill the 10px thickness, drawn only while turning. The back repeats the finish with lx mirrored. Change content or finish only when the card is edge-on.
>
> Standard depth, only for artwork that needs it: x = -d*sin(rotateY+turn), y = d*sin(rotateX). The compass demo depths are not a requirement: picture 90px behind, dial 50, middle ring 34, dashed ring 20, needle -12, corner print fixed. Draw the backmost layer 100px larger on each side. One slow moving idea if the artwork can respond.
>
> Reduced motion: no tilt, no sway, no flip animation (instant swap or a 200ms cross-fade). The light may still follow the pointer. The background stays still. Keep the card's role. The back is reachable and announced. Nothing important exists at only one angle.
>
> Put the tilt on the card and the light variables on the face only. Move glare as a 2x layer. One animation loop. Stop it off screen. Skip writes that did not change. Draw only the face that shows. Check all four corners and the center, rapid flips, reduced motion, keyboard, and both light and dark pages.

上面是改写，不是原文。完整 Prompt 见[作者回复](https://x.com/Kubai087/status/2121892403212964060)。

## 常见错误

- 一屏上每张卡都做成镭射。只留那一张代表身份或所有权的。
- 选择表还没通过就开始写代码。
- 改了卡片上的文字、数据，或它原来的点击、跳转、提交。
- 箔画在印刷层上面，字被光吃掉。眩光把字洗白也算。
- 边缘光画成一圈始终实心的描边。它只该在靠光的一段出现，然后淡掉。
- 用户没选定背面，就自己编一个背面并让卡片翻转。
- 翻转时两面同时看见，或侧边露出一条缝。快速连翻也会出现。
- 换布局、换文字或换质感发生在翻转中途，而不是正侧面。
- 开了减弱动效，卡片仍在倾斜、摆动，或翻转还在做动画。
- 光的 CSS 变量写在父级上，整棵子树跟着重算样式。
- 卡片离开屏幕，循环还在跑。
- 另做一页演示，而不是改产品里那张卡。
- 原帖没给的数，被写成了规则。包括品牌色的换算、闪点的随机公式、安静档和活泼档没写出的正弦系数、设备倾斜的增益、窗框浅色的宽度。
- 做出来的数值和确认过的表不一致。
- 只在卡片中心看过字，没在四角看。亮色页和暗色页只看了一边。
- 背面用键盘到不了，或读屏不知道背面的内容。
- 没人选手机倾斜，却去申请设备方向权限。

## 出处

- 原帖：https://x.com/Kubai087/status/2121892029403922717
- Prompt 回复：https://x.com/Kubai087/status/2121892403212964060
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例代码是本仓库的。

下面这些是本仓库取值或本仓库的读法，原帖没有写成规则：

- 眩光 28% 那一档只写了不透明度 14%，示例用白色。Chrome 眩光 30% 处的 10% 同样没写颜色，示例没做 Chrome。
- 流体的 `clamp` 按 0 到 1。`|q|` 按向量长度。
- 圆角按 `卡片宽度 / 440 * 32`。原帖只说随卡片缩放。
- 阴影椭圆按 84% 宽度水平居中，所以左右各 8%。顶边放在卡片底边下 30px。
- 翻转过程中，阴影收窄 45% 和再下落 14px 用 `|sin(翻转角)|` 插到中间帧。
- 侧边 10 份按 1px 厚，中心从 −4.5px 到 +4.5px，把 10px 填满。
- 安静档的悬停封顶用 `min(on, 0.7)`。原帖没写截断还是按比例缩小。
- 封印角度把公式的结果直接当度数，`atan2` 的弧度不再换算。
- `sin` 的自变量先从度换成弧度。`shortAngle` 里的 540 只用于折半圈。
- 2 倍眩光层用 `translate(calc((var(--px) - 100) * 0.5%), …)` 把层中心移到光点。0.5 来自层是卡片的 2 倍。
- 窗内渐变在 18% 之后沿用起笔的那个颜色，只改透明度。
- 指针离开后，`on` 按 `exp(-7 * dt)` 回到 0。原帖只写了指针在卡上时走向 1，回落沿用同一个 7。
