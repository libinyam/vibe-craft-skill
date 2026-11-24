# 023 · 做好移动端网页适配的 4 个 Tips
**Four details that make web apps feel native on mobile**

> 分类：布局 Layout · 原帖：[2026-11-23](https://x.com/Kubai087/status/2125029129619644704) · Prompt：[作者回复](https://x.com/Kubai087/status/2125032522061099249) · 站点链接：[作者回复](https://x.com/Kubai087/status/2125036623922704401) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对介绍帖、作者站点同一条技巧和附带 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。关掉 iOS 点击高亮、悬停只留给鼠标、把顶部 Tab 移到底部并随滚动收成小胶囊、状态栏接上页面顶部的颜色、弹窗遮罩和底栏不要固定在窗口上（否则工具栏后面多出一条灰带）、有条件就支持添加到桌面，来自介绍帖。iPad 横屏仍按触屏处理、悬停放进 `@media (hover: hover)`、向下超过 8px 收成屏幕正中的胶囊、向上超过 8px 或点一下再展开、从 iOS 26 起 Safari 读取贴着顶边和底边的最上层、不用时必须 `visibility: hidden`、遮罩放回页面文档流、底栏毛玻璃画在内层、图标至少 512 且 `apple-touch-icon` 不带透明、短名称大约十来个字符、用 service worker 让访问过的页面离线能开，来自作者站点。点击热区为什么会大于按钮、`:active` 需要一次触摸监听、Tailwind 的 hover 开关、底栏尺寸和收放时序、Safari 取色的完整行为、manifest 字段与图标尺寸、先列表再改，来自官方 Prompt，下面会标明。只在演示视频里的画面不当成新规则。介绍帖时间为 2026-11-24 00:50:00 UTC，Prompt 回复为 2026-11-24 01:03:29 UTC，按本仓库约定把介绍帖记为 America/Los_Angeles 的 2026-11-23。没有英文原帖。原帖附约 35 秒演示视频。完整 Prompt 在作者回复，作者站点同一条也收录了全文：[做好移动端网页适配的 4 个细节](https://vibedesign.tips/zh/023-mobile-web/)（英文页题为 Four details that make web apps feel native on mobile）。本文中文标题用介绍帖，英文标题用英文页标题。「实现要点」代码为本仓库整理的示例实现。

## 原理

桌面上完整的网页，放到手机上常常少了原生应用的完成度：添加到桌面后图标发糊、名称被截断，一点就闪出一块深色，导航停在拇指够不着的顶部，Safari 的状态栏和工具栏也跟页面颜色脱节。介绍帖把要补的地方收成四条：关掉点击高亮、把 Tab 移到底部并随滚动收放、让 Safari 顶底栏取到页面自己的颜色、有条件就做成可以添加到桌面的 PWA。作者站点用 `vibedesign.tips` 的做法把这四条展开。

官方 Prompt 要求先诊断，按这四处列出问题，等确认再改。这是改产品里已经有的页面，不是做一页演示。在现有代码、技术栈和样式里改。没有问过，不加依赖。颜色、材质、圆角、字体、间距、图标和动效都用产品自己的；下面的尺寸是还没有对应 token 时的默认值。产品里已经有浮动底栏、抽屉、毛玻璃或应用图标，就在那上面改。

## 规则 / 数值

先判断，再建议。点头的那些才改。桌面端的悬停和键盘焦点保持原样。

### 1. 去掉点击高亮，悬停只留给指针

介绍帖：WebKit 会在点过的地方盖一块深色，悬停样式也会停住，元素看起来在闪、在跳。关掉高亮，悬停只留给鼠标。

作者站点和官方 Prompt 把原因写细了。Safari 在 iOS 上给被点的元素盖一块半透明深色，范围是整块点击热区。热区如果用 padding 或伪元素加大，这块深色就比控件本身大。每一页都关掉它：

```css
html { -webkit-tap-highlight-color: transparent; }
```

按下反馈留下。每个控件保留 `:active`。iOS 上的 Safari 只有在页面监听了触摸时，才会把触摸算成 `:active`。手机上按下样式不出现时，在 `document` 上加一个空的、`passive` 的 `touchstart` 监听。

触屏没有悬停，但 Safari 会把点过的元素放进 `:hover`，直到再点别处才放开。`:hover` 改了尺寸、位置或颜色，就会在点击时出现并停住，控件像是跳了一下。

- 每条 `:hover` 放进 `@media (hover: hover)`，只有鼠标或触控板才用。
- 选择器把 `:hover` 和 `:focus-visible` 或 `:focus-within` 写在一起时拆开：焦点那一半留在媒体查询外面，键盘用法不变。`:active` 也留在外面。
- Tailwind：v4 的 `hover:` 已经只在 `(hover: hover)` 下生效；v3 打开 `future.hoverOnlyWhenSupported`。任意变体和绕过这套机制的普通 CSS 要单独看。
- 宽度不是触屏。iPad 横屏宽过 1024px，仍然是触屏。写在宽度查询里的 `:hover`，里面再套一层悬停查询：

```css
@media (min-width: 1024px) {
  @media (hover: hover) {
    .chip:hover { /* … */ }
  }
}
```

- 只给触屏的样式放进 `@media (hover: none)`：更大的圆点或点击目标，以及原本只在悬停时出现的箭头。
- 只在悬停时出现的东西（箭头、菜单、提示、删除按钮）要有一条能点到的路径。
- 脚本里 `element.matches(':hover')` 在点击之后也会一直为真。需要区分鼠标和手指时，用指针事件的 `pointerType`。
- 两类悬停不要动：只在 `prefers-reduced-motion` 下关掉过渡的规则，以及内容本身就是悬停演示的预览。

### 2. 底部 Tab，以及按顺序阅读时的上一篇 / 下一篇

介绍帖：拇指常用的区域在屏幕下方。底部 Tab 随滚动收起和展开——往下读时收成小胶囊，往上滚或点一下再展开。

作者站点补充了本站的做法：首页顶部的 Tab 移到底部，教程页的上一期、下一期也放在同一位置，避免底栏一直挡住正文。

官方 Prompt 把两条底栏当成同一套设计。形状和尺寸是默认值；颜色、材质、圆角和图标用产品自己的。长文（文章、课程、章节）才放上一篇 / 下一篇。必须保持完整的底栏（结账合计、输入条、播放器）留在原地，并在表里写明是哪几条。

**共用外形。** 贴在底部：左右和底边各 12px，再加上 `env(safe-area-inset-bottom)`。高 56px，内边距 4px，胶囊形。以安装后的应用打开时（`display-mode: standalone`）底下没有浏览器工具栏，改用 iOS 自己的标签栏尺度：左右和底边各 21px，高 62px。

底板是 `::before`。产品已有浮动材质就用那一套。还没有时，Prompt 的默认是：表面色大约 82%，`backdrop-filter: blur(18px) saturate(1.4)`，一条发丝线（`0 0 0 0.5px`、6% 黑）和一层软阴影（`0 2px 8px -2px`、16% 黑），阴影落在底栏下方的空隙里。底栏自己不带背景，原因见下一节。按下时按键有按下的样子，但不加阴影。页面底部留出底栏的位置，最后一行不要被挡住。

**Tab 栏。** 二到五个视图均分。每一项是图标在上、文字在下，间距 2px，居中。图标 20px（作为应用打开时 24px）；文字 11px、行高 14px（作为应用时 10px / 13px），一行，超出用省略号。未选中用次要文字色。当前项坐在一块浅色上，这块浅色是栏内边距里的一颗胶囊，切换时滑过去（320ms，`ease-out`）。再点当前项，页面平滑回到顶部；开启减弱动效时立即回到顶部。

图标用产品图标库里的双色。浅的那一层从底板的颜色混出来，Prompt 举的例子是 `color-mix(in srgb, currentColor 30%, var(--surface))`，亮色和暗色都能读。图标库没有双色时，可以建议 [Kubai Icons](https://icons.kubai.design)（MIT）：16 网格的双色图标，颜色跟 `currentColor`，有 React、Vue、原生 JavaScript 和静态 SVG（`kubai-icons` 的 duotone，浅层读 `--kd-b`）。问过再加依赖。它的图标会做出含义相关的动作；只在某一项变成当前项时、或指针悬停时播放，不要自己循环。

**上一篇 / 下一篇。** 外形和位置与 Tab 栏相同，用在按顺序读的页面上。两个键平分：箭头加上一篇标题，以及下一篇标题加箭头。标题各占一行并省略，栏高不会变。读屏听到 `Previous: <标题>` 和 `Next: <标题>`。两端循环：最后一篇的下一篇是第一篇，第一篇的上一篇是最后一篇，两个键始终可点。前面可以再放一颗圆形键（分享或来源），高度与栏相同，距栏 8px；底板从这颗键之后开始。

**随滚动让路。** 两条底栏同一套规则。

| 时机 | 条件 |
|---|---|
| 收起 | 所属内容连续向下移动超过 8px |
| 展开 | 连续向上超过 8px；或者距顶部 48px 以内；或者点了已经收起的胶囊（立即展开） |
| 不计入 | 超过一屏的跳动（恢复滚动位置、跳到某一节），以及越过两端的回弹 |

只计算这条栏所属的滚动：Tab 栏跟它切换的那一页，阅读栏跟正文。每次收起或展开先停住，时间比动画略长（280ms 的收起对应停 320ms），之后滚动才可以把它转回去，而且只执行最新的一次决定。转到一半时，用布局移动的底板和用 transform 移动的内容会错开。

收起后栏的位置不动，底板从两侧同时收向屏幕正中的一颗胶囊：高 40px，作为应用打开时 44px。

| 栏 | 收起后留下什么 |
|---|---|
| Tab | 当前项：图标在文字左边，文字放大到 1.1 倍，间距 6px，两侧各 14px |
| 上一篇 / 下一篇 | 两支箭头，距中线两侧各 14px，胶囊宽 80px；前面的圆形键淡出并缩到 0.8 倍 |

其余内容先淡出，大约 100ms，底板再从它们旁边收过去。展开沿同一条路径回来；底板大体打开之后，其他视图和标题再淡入。单程 280ms，`ease-out`，以中线对称。收起后只有胶囊接收点击，其余区域把点击交给页面。正要移进栏里的按键会把栏打开。栏里还有键盘焦点时不要收起。开启减弱动效时，收起和展开不做动画。

### 3. Safari 状态栏和工具栏的取色

介绍帖：Safari 会读贴着屏幕顶边和底边的那一层来上色。状态栏想接上页面顶部的颜色。弹窗遮罩和底栏不要固定在窗口上，否则工具栏后面会多出一条灰带。

作者站点写的是 iOS 26 起的做法，并记下本站的两处处理：打开教程时，在顶边放一条与演示同色的细条，让状态栏接上演示背景，页面一滚动就把细条撤掉；菜单遮罩放回文档流，底栏毛玻璃画在内层，外框本身不带底色。调试用 Xcode 自带的 iPhone 模拟器，取色和真机一致；同时测正式构建，构建时注入的脚本在开发服务器里未必存在。

官方 Prompt 把取色写成测试里观察到的行为，并写明 Apple 没有为它提供文档。iOS 26 及之后的 Safari：

- 大体上不看 `<meta name="theme-color">`。
- 读取贴着屏幕顶边或底边、最上层的 `fixed` 或 `sticky` 元素的背景，用来填状态栏，或填工具栏下面那一条。
- 跳过 `visibility: hidden`。不跳过 `opacity: 0`，也不跳过透明背景。
- 最上层若是一块透明的全屏层，Safari 不会往下看，上次读到的颜色会一直留到刷新。颜色「卡住」就是这个原因。
- 页面自己在滚动、内容从两条系统栏下面经过时，Safari 会在状态栏下加自己的模糊，工具栏也可以随滚动收起。
- 固定在窗口上、并且碰到窗口底边的一层（压暗遮罩、对话框的背景）会把工具栏后面涂成一块纯色；这层关掉之后，颜色仍可能留到刷新。

逐项检查：

- **透明的全屏层。** 盖住屏幕、平时是透明的固定层（滚动条层、用来接点击的层、还没打开的遮罩）会把两种颜色都冻住。不用时设 `visibility: hidden`，只有真正要看见的部分设 `visibility: visible`（子元素可以盖过隐藏的父元素）。不要只靠 `opacity: 0` 或透明背景。
- **遮罩。** Safari 的工具栏浮在页面上，页面应当从它下面透出来。固定层只要伸到窗口底边，Safari 就改涂纯色，层的阴影和 `pointer-events` 也算。菜单、抽屉、对话框后面的压暗遮罩改成页面的一部分，不要固定在窗口上：用页面坐标绝对定位（从面板下沿起、高度为一屏），显示时把页面按住不动。浏览器就会把压暗后的页面画到工具栏下面。
- **靠近底边或顶边的毛玻璃 / 半透明栏。** 背景、模糊和阴影挪到 `::before`，栏本身不设背景，Safari 读到的才是页面自己的颜色。底栏的底板用这个办法，也是为了这一条。
- **自己内部滚动的全屏盒子。** 固定的抽屉、文章视图或覆盖手机屏幕的面板，打开时被读一次；里面再滚动，顶底栏的颜色不变。手机宽度上，一旦它盖满屏幕，改由页面来滚：盒子回到普通文档流，后面的页面固定并隐藏。切换时把盒子的滚动位置交给页面，避免跳动；关闭时交还，并恢复页面自己的滚动位置。平板和桌面仍用内部滚动。这一改动比较大：先描述，问过再做。
- **接上内容的顶边。** 屏幕顶部是一块有颜色的区域（主视觉、封面、演示），状态栏应当延续它时，在顶边放一条几像素高的固定细条，颜色取这块区域顶边的颜色，并沿宽度取平均。边缘不均匀，或已经接近页面背景时，不要设。页面一开始滚动就藏起细条，之后交给 Safari 自己的模糊。这条是提议，没有点头不要加。
- **其他浏览器和较旧的 Safari。** `theme-color` 以及 `html` 的背景，亮色和暗色都保持正确。
- **颜色不对、原因不清楚时。** 在顶边放一条几像素高的亮绿色固定细条，并给高的 `z-index`。状态栏变绿，说明取样是通的，问题在被设进去的颜色。不变绿，说明上面还有另一层固定元素：把 `z-index` 抬到变绿为止，那一层就是挡住取样的层。这是调试手段，不要留在产品里。

### 4. 有条件就做成 PWA

介绍帖：让用户把网页添加到桌面，像应用一样打开。

作者站点：准备一份写明名称、图标和打开方式的 manifest，并在每页页头引入。图标至少 512×512，按自己的标志重画，不要把小 favicon 放大。iOS 的 `apple-touch-icon` 不能有透明通道，透明像素会被填成黑。桌面图标下方大约只容得下十来个字符。用 service worker 缓存访问过的页面，弱网或离线时仍能打开。

官方 Prompt 把清单写成这些默认项。产品里已经有 manifest、图标或 service worker，就扩展现有的。

**manifest。** `id` 和 `start_url`（`"/"`）、`scope`、`name`、不超过 12 个字符的 `short_name`、`description`、`lang`、`display: "standalone"`。`background_color` 和 `theme_color` 用产品 token，也就是页面自己的背景。加上 `launch_handler`，`client_mode` 为 `["navigate-existing", "auto"]`；二到四个主要位置的 `shortcuts`；一张竖的手机截图和一张横的截图，让安装面板更完整。

**图标。** 从自己的标志绘制，各尺寸都清晰，不要放大 favicon。PNG：192×192 和 512×512（`purpose: any`）；一张 512×512 的 maskable，标志落在中间 80%，背景用品牌底色；`apple-touch-icon` 180×180，不透明。

**每页页头。** manifest 链接、`apple-touch-icon`、`theme-color`（产品有亮色和暗色时各一条）、`mobile-web-app-capable`、`apple-mobile-web-app-title`（用 `short_name`）、`apple-mobile-web-app-status-bar-style` 为 `default`。

**离线。** service worker 对页面先走网络，离线或大约 4 秒仍慢时再用缓存；带指纹的静态资源从缓存取；不缓存 API 和视频；每次新构建丢掉旧缓存。扩展产品已有的 worker；没有时先问，再决定要不要加。

**从桌面打开。** 没有浏览器工具栏。每个固定元素都对照安全区；底部栏用上面写的应用内尺度（21px / 62px）。

### 先看产品，再动手

用几行把发现说清楚，再出表。不要先改代码。

先找：样式表、CSS Modules、CSS-in-JS 里的 `:hover`，Tailwind 的 `hover:` 和 `group-hover:`，以及响应 `mouseenter`、`mouseover` 或 `element.matches(':hover')` 的脚本。再找贴着屏幕顶边或底边的 `fixed` / `sticky`：页头、Tab、工具栏、toast、cookie 条，以及全屏层（遮罩、脚本画的滚动条、装饰层、自己内部滚动的抽屉）。再找页面的主 Tab 或视图（常常在顶部）、按顺序阅读的上一页 / 下一页，以及手机上已经固定在底部的栏，连同它们随滚动的行为。再找现在能不能安装：web app manifest、图标、`apple-touch-icon`、页头 meta、service worker。记下设计语言（颜色和间距 token、圆角、有没有毛玻璃、字号、图标集、动效曲线）、`theme-color`、`html` 和 `body` 的背景、亮色 / 暗色怎么切换，以及只在正式构建里注入、开发服务器上看不到的脚本或样式。

表的每一行写：在哪（文件和选择器，或组件）、现在手机或 iPad 上看到什么、对应上面哪一条、改法以及会不会影响桌面。按四处分组：作为应用打开、点击、底部栏、状态栏和工具栏。写明找到多少条 `:hover`、其中多少已经在 `(hover: hover)` 里。等人选完再改。

改的时候：桌面外观保持不变，鼠标悬停仍然有效，焦点环不动。每个组件自己的逻辑留下，只动相关的样式和层。某条规则只为 Safari 存在时，留一句短注释，写明它防的是什么。

检查：在 iOS 模拟器里用 Safari 的分享添加到主屏幕，在 Android 或电脑上用 Chrome 安装。图标清晰，名称放得下，打开后没有浏览器栏，快捷方式可用，访问过的页面离线能开。用 Xcode 自带的模拟器测一台 iPhone，以及一台横屏、宽过 1024px 的 iPad；它的状态栏和工具栏取色与真机一致，最后在真机上再确认。测本地提供的正式构建，不只测开发服务器。点每一个带悬停的控件：没有深色块，尺寸和位置不变，按下时 `:active` 仍在；桌面鼠标悬停仍在，键盘焦点不变。列出仍在 `(hover: hover)` 之外的 `:hover`，并说明为什么留下。打开并关闭每个全屏层和抽屉，滚动，在颜色不同的页面之间切换：状态栏和工具栏跟着走，没有颜色要刷新才消失。带底栏的页面向下、向上、慢滑和快甩：栏收放不闪，胶囊停在屏幕正中。亮色和暗色，以及一台窄而短的手机（375 × 667）。

## 实现要点（示例实现，非原帖代码）

下面把官方 Prompt 里的默认尺寸收成一组起点 token。项目已有底栏、毛玻璃或 manifest，就扩展现有的。变量名、选择器，以及「页面底部留白 = 栏高 + 距底边的间距」这一条换算，是本仓库的。颜色优先用产品 token；82% 表面色、发丝线和软阴影只在产品没有浮动材质时用作起点。

```css
html { -webkit-tap-highlight-color: transparent; }

/* 悬停只留给指针。把产品里已有的 :hover 声明移进来，不要在这里新造效果。焦点和按下留在查询外面。 */
@media (hover: hover) {
  .chip:hover { /* 原有悬停 */ }
}
.chip:focus-visible { /* 焦点环保持原样 */ }
.chip:active { /* 按下样式留在查询外面 */ }

@media (min-width: 1024px) {
  @media (hover: hover) {
    .chip:hover { /* 宽度查询里的悬停再套一层 */ }
  }
}

.foot-bar {
  --bar-gap: 12px;
  --bar-height: 56px;
  --bar-pad: 4px;
  --bar-fold-h: 40px;
  --bar-icon: 20px;
  --bar-label: 11px;
  --bar-label-lh: 14px;
  --bar-slide: 320ms;
  --bar-fold: 280ms;
  --bar-hold: 320ms;          /* 比收起动画略长，期间不反向 */
  --bar-fade: 100ms;
  position: fixed;
  left: var(--bar-gap);
  right: var(--bar-gap);
  bottom: calc(var(--bar-gap) + env(safe-area-inset-bottom, 0px));
  height: var(--bar-height);
  padding: var(--bar-pad);
  border-radius: 999px;       /* 胶囊。Prompt 只要求 capsule，999px 是本仓库的画法 */
  background: none;           /* 底色放在 ::before，避免 Safari 把工具栏涂成纯色 */
}
@media (display-mode: standalone) {
  .foot-bar {
    --bar-gap: 21px;
    --bar-height: 62px;
    --bar-fold-h: 44px;
    --bar-icon: 24px;
    --bar-label: 10px;
    --bar-label-lh: 13px;
  }
}
.foot-bar::before {
  content: "";
  position: absolute;
  inset: 0;
  border-radius: inherit;
  background: color-mix(in srgb, var(--surface) 82%, transparent);
  backdrop-filter: blur(18px) saturate(1.4);
  box-shadow:
    0 0 0 0.5px rgb(0 0 0 / 6%),
    0 2px 8px -2px rgb(0 0 0 / 16%);
}
/* 收起后底板高为 --bar-fold-h，停在屏幕正中。上一篇 / 下一篇的胶囊宽 80px；
   Tab 的宽度跟着当前项走，两侧各 14px。收起过程不写进这条示例。 */

/* 不用的全屏层必须看不见，opacity: 0 挡不住 Safari 取样 */
.layer-idle { visibility: hidden; }
.layer-idle .knob { visibility: visible; }
```

正文底部留出 `栏高 + 距底边的间距`，最后一行才不会被挡住。收起判定不要写进 CSS：连续向下超过 8px 才收，连续向上超过 8px、距顶 48px 内、或点到胶囊才展开；超过一屏的跳动和两端回弹不计入；反向至少等 `--bar-hold`，并且只执行最后一次决定。开启 `prefers-reduced-motion: reduce` 时不做这段动画。

顶边接色的细条只有几像素高，颜色取内容顶边沿宽度的平均，页面一滚动就移除。亮绿色细条只用来确认取样通不通，确认后删掉。这两条都不要写成默认样式。

manifest 的字段来自 Prompt。`background_color` 和 `theme_color` 换成产品页面的背景 token。

```json
{
  "id": "/",
  "start_url": "/",
  "scope": "/",
  "name": "Product name",
  "short_name": "Product",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#ffffff",
  "launch_handler": { "client_mode": ["navigate-existing", "auto"] },
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png", "purpose": "any" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "any" },
    { "src": "/icon-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

`#ffffff` 只是占位，换成产品背景。maskable 图标把标志放在中间 80%。另加一张 180×180、不透明的 `apple-touch-icon`。service worker 先走网络，大约 4 秒无响应或离线时再用页面缓存；不缓存 API 和视频。没有现成 worker 时先问。

手机上按下样式不出现时，加一次空监听：`document.addEventListener("touchstart", () => {}, { passive: true })`。这条是为了让 iOS 把触摸算成 `:active`。

## 可直接用的 Prompt（精简版）

> Look over how my site behaves on a phone and on a tablet. Four areas: opening it from the home screen, what a tap leaves behind, moving the main tabs and the previous/next links into one bar at the foot that steps aside while the page scrolls, and how iOS Safari tints the status bar and the toolbar. Diagnose first. Give me a table, and change only the rows I approve. This is my product: work in my code, on my stack, with my styles. Do not add a dependency without asking. Stay inside my design language. Numbers below are defaults to fit to my tokens. If I already have a floating bar, a sheet, a glass material or an app icon, build on that.
>
> Read first, in a few lines: every hover (`:hover`, CSS modules, CSS-in-JS, Tailwind `hover:` and `group-hover:`, and scripts on mouseenter, mouseover or `matches(':hover')`); every fixed or sticky element touching the top or bottom edge, including full-screen layers; the main tabs, previous/next links, and any bar already fixed at the foot, with how each behaves while scrolling; what makes the site installable today; my tokens, radii, glass, type, icons and motion; theme-color, html and body backgrounds, and how light and dark switch; anything injected only by the production build.
>
> PWA, when it belongs. Manifest: id and start_url `"/"`, scope, name, short_name of 12 characters or fewer, description, lang, `display: standalone`, background_color and theme_color from my page background, `launch_handler.client_mode` `["navigate-existing", "auto"]`, shortcuts to two to four places, one narrow screenshot and one wide. Icons from my own mark, never a scaled favicon: PNG 192 and 512 (`purpose: any`), a 512 maskable with the mark inside the middle 80% on my brand background, and an opaque apple-touch-icon at 180 (iOS fills transparent pixels with black). On every page: the manifest link, the touch icon, theme-color (one per scheme), `mobile-web-app-capable`, `apple-mobile-web-app-title` set to the short name, and `apple-mobile-web-app-status-bar-style` `default`. Offline: extend the service worker I have — network first, cached page after about 4 seconds or when offline, fingerprinted assets from cache, never API or video, drop old caches each build. Ask before adding one. In standalone, check fixed elements against the safe area, and size the foot bar like iOS's tab bar (below).
>
> Taps. Turn the highlight off on every page: `html { -webkit-tap-highlight-color: transparent; }`. Keep `:active`. If a press does not show on a phone, add one empty passive `touchstart` on `document`. Put every `:hover` inside `@media (hover: hover)`. Split a selector that joins hover with `:focus-visible` or `:focus-within`: focus stays outside the query, and so does `:active`. Tailwind v4 already limits `hover:`; on v3 set `future.hoverOnlyWhenSupported`, and check arbitrary variants and plain CSS. An iPad in landscape is wider than 1024px and still touch, so nest `(hover: hover)` inside a width query. Touch-only styles go in `(hover: none)`. Anything that appears only on hover needs a tap path. Where script must tell a mouse from a finger, use `pointerType`, not `matches(':hover')`. Leave hover that only disables motion, and demos whose content is the hover.
>
> Foot bar. On a phone, move the main tabs to the foot, and on pages read in order put previous and next in the same place. One design, my materials. Defaults: 12px from the sides and the foot plus `env(safe-area-inset-bottom)`, 56px tall, 4px padding, a capsule. Standalone: 21px and 62px tall. Paint the plate on a `::before` in my floating material (if I have none: surface at about 82%, `blur(18px) saturate(1.4)`, a 0.5px hairline at 6% black, shadow `0 2px 8px -2px` at 16% black, kept in the gap below). The bar itself has no background. Pressed keys have a press and no shadow. Leave room so the last line clears the bar. Tabs: two to five, equal, icon over label, 2px apart; icon 20px (24 as an app), label 11/14 (10/13 as an app), one line, ellipsis; inactive uses secondary text. The current view sits on a capsule that slides in 320ms ease-out. Tapping it scrolls to the top (at once under reduced motion). Two-tone icons from my set, the light tone mixed from the plate (for example `color-mix(in srgb, currentColor 30%, var(--surface))`). If I have no two-tone set, recommend Kubai Icons (icons.kubai.design, MIT) and ask before adding it; play its motion only when a view becomes current, or under a pointer. Previous/next: same shape; arrow plus title, title plus arrow, one line each so the height never changes; accessible names `Previous: <title>` and `Next: <title>`; the ends loop. One round key may sit 8px before the bar, as tall as the bar; the plate starts after it. Fold after 8px down in a row; open after 8px up, within 48px of the top, or at once when the capsule is tapped. Count only the scroll this bar belongs to. Ignore a jump of more than a screen, and bounce past either end. Hold 320ms around a 280ms fold before reversing, then apply only the latest decision. Folded, the bar stays put and the plate closes from both sides onto a 40px capsule (44 as an app) at the middle of the screen. Tabs keep the current view, icon beside a label at 1.1×, 6px apart, 14px either side. Previous/next keeps two arrows, 14px either side of centre, in an 80px capsule; a leading round key fades and scales to 0.8. Everything else fades in about 100ms before the plate passes it. Open along the same path, 280ms ease-out, symmetric; other labels return once the plate is mostly open. Folded, only the capsule takes taps. A key moving into the bar opens it. Do not fold while it holds keyboard focus. Reduced motion: no animation. Leave a checkout total, a composer or a player whole, and name them.
>
> Safari on iOS 26 and later, from testing, not from Apple's docs. It mostly ignores theme-color. It samples the topmost fixed or sticky background touching the top or bottom edge. It skips `visibility: hidden` and does not skip `opacity: 0` or a transparent background. A transparent full-screen layer on top freezes the last colour until reload. When the page itself scrolls, content shows under both bars. A layer fixed to the window and reaching its foot paints one flat colour behind the toolbar, and that colour can stick after the layer is gone. Hide idle full-screen layers with `visibility: hidden` (visible children may override). Put a scrim in page coordinates, from just under the panel for one screen height, and hold the page still — do not pin it to the window. Move a frosted bar's background, blur and shadow onto a `::before` and leave the bar transparent. On a phone, a full-screen sheet that scrolls inside itself is sampled once; describe handing scroll to the page, and ask before doing it. Tablets and desktops keep inner scroll. A coloured hero may get a fixed strip a few pixels high, averaged from its top edge only, hidden as soon as the page scrolls; offer it, do not add it unasked, and skip an uneven edge or one close to the page background. Keep theme-color and the html background correct for both schemes. If the status colour is wrong, a bright green fixed strip a few pixels high finds the layer: green means sampling works, otherwise raise the z-index until it does. Remove the strip afterward.
>
> Table first, no code yet. One row per problem: where, what a person sees, which rule, the fix and any desktop effect. Group by the four areas. Count `:hover` rules and how many already sit in `(hover: hover)`. Then, for what I approve: desktop hover and focus rings stay; move only the styles and layers; comment a rule that exists only for Safari. Check on the iOS Simulator (iPhone, and an iPad in landscape wider than 1024px) and once on a real phone; install from the home screen; test the production build, not only the dev server; tap every hovered control; list any `:hover` left outside the media query and why; open and close every full-screen layer and switch page colours without a stuck tint; fold and open each foot bar, including quick flicks, with the capsule staying centred; light and dark, and a short phone at 375 × 667.

上面是改写，不是原文。完整 Prompt 在[作者回复](https://x.com/Kubai087/status/2125032522061099249)，作者站点同一条也有全文：https://vibedesign.tips/zh/023-mobile-web/

## 常见错误

- 留下 iOS 点击高亮，热区比按钮大时，深色块也比按钮大。
- `:hover` 没有放进 `(hover: hover)`，点击之后尺寸或颜色停住。用 `min-width: 1024px` 当成「这不是触屏」。
- 把 `:focus-visible`、`:focus-within` 或 `:active` 一起放进悬停查询，键盘或按下反馈没了。
- Tailwind v3 没开 `hoverOnlyWhenSupported`，或任意变体和普通 CSS 绕过了它。
- 用 `element.matches(':hover')` 区分手指和鼠标。只在悬停时出现的菜单或删除按钮没有点击路径。
- 改掉了只为减弱动效服务的悬停，或内容本身就是悬停演示的预览。
- 手机上的主导航仍留在顶部。底栏不随滚动让路，或手指一抖、程序跳转、两端回弹也让它收放。
- 动画没完就反向，底板和上面的内容错开。收起后胶囊以外的区域仍接着点击。
- 把结账合计、输入条或播放器也收成胶囊。
- 底栏自己带背景或阴影，Safari 在工具栏后面涂一条纯色。遮罩 `position: fixed` 到窗口底边，灰带关掉菜单也不消失，直到刷新。
- 全屏层只用 `opacity: 0` 或透明背景隐藏，顶底栏颜色卡住。
- 手机上盖满屏幕的抽屉仍在内部滚动，状态栏颜色打开时就被定死。没问过就改成整页滚动。
- 没点头就加了顶边接色细条，或页面滚动后细条还在。亮绿色调试条留在了产品里。
- 把小 favicon 放大当主屏幕图标。`apple-touch-icon` 带透明，被填成黑底。`short_name` 超过大约 12 个字符。
- 没问过就新加 service worker，或把 API、视频放进缓存。
- 表还没给人看就开始改。顺手改了桌面悬停或焦点环。
- 只在开发服务器上看，正式构建注入的固定层没有测到。
- 给参数表和 Prompt 没写的颜色、字体或间距补上一套。82% 表面色、`blur(18px)` 和那两层阴影只是没有浮动材质时的起点。
- 把视频里的界面当成必须照搬的布局。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2125029129619644704 （2026-11-24 00:50:00 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-23）
- Prompt 回复：https://x.com/Kubai087/status/2125032522061099249 （2026-11-24 01:03:29 UTC）
- 站点链接回复：https://x.com/Kubai087/status/2125036623922704401
- 作者站点同一条（中文，含完整 Prompt）：https://vibedesign.tips/zh/023-mobile-web/
- 英文页同一条：https://vibedesign.tips/023-mobile-web/ （标题 Four details that make web apps feel native on mobile）
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 没有英文原帖。原帖附约 35 秒演示视频。视频里的画面没有写成新规则。
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例里的 CSS 变量名、底栏留白的换算、`color-mix` 的接法，以及 manifest 里的 `#ffffff` 占位，是本仓库的。
