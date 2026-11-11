# 015 · 用好连续性，让你的产品更丝滑
**Use continuity to make your product feel smoother**

> 分类：动效 Motion · 原帖：[2026-11-10](https://x.com/Kubai087/status/2120320609255043510) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对原帖说明和附带 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。曲线、时长和比例取自原帖。原帖没有英文版。帖子时间戳为 2026-11-11 01:00 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-10。原帖附约 31 秒演示视频，用 iOS 里七个常见场景把连续性走一遍；画面里没写成文字的细节，这里不补，观感以视频为准。Prompt 在原帖下方作者的回复里，完整原文从原帖进入该回复。「实现要点」代码为本仓库示例。

## 原理

人的视线还停在刚刚点过的那一块。状态一切换，若这块先没了、下一屏再从别处冒出来，来路就断了。连续性是让**同一块**从起点的位置和形状走到终点的位置和形状，用户始终能对上「刚才点的是这个，现在它在这里」。

硬切、整屏交叉淡化、从一条不相干的边上滑进来，都算跳。要改的是「这一态怎么变成下一态」。每个操作做什么、每屏放什么、静止时落在哪，都保持原样。

这是改现有产品，不是做一条演示。在现有代码、技术栈、组件、token 和布局里改。不要另做一个手机框、衬底、示例文案或一套新字体。没有征得同意，不要加依赖，也不要改路由方案。下面的数值都是起点；项目里已有时长、曲线、弹簧，就沿用那些。

## 规则 / 数值

### 先看产品，再动手

1. 找出屏幕、路由、面板、抽屉、对话框、菜单和列表，以及它们怎样被打开：导航栈或路由、模态和抽屉、弹出层和菜单、列表和网格。
2. 找出现在在动的东西：CSS transition、View Transitions API、Motion / Framer Motion、GSAP、SwiftUI 或 UIKit、Compose、Flutter 的 Hero 或隐式动画。记下已有的时长、曲线、弹簧，以及减弱动效怎么处理。
3. 记下有哪些布局（手机、平板、桌面窗口、分栏）和各自的输入（触摸、鼠标、键盘）。
4. 用几行把以上发现说清楚，再往下走。

然后给一份排序后的建议，**最多 10 条**。频繁、断了就容易迷路、改动又小的排在前面。每条都写：

| 栏 | 写什么 |
|---|---|
| 哪里 | 屏幕或组件，触发动作，手机 / 桌面 / 两边 |
| 现在 | 今天实际发生的事，一行 |
| 模式 | 下面 a–h 的哪一种，哪一个元素该被带走 |
| 改法 | 什么移到哪里，时长和曲线，两三行 |
| 价值 | 高 / 中 / 低。看人做这件事的频率，以及断掉之后会不会迷路 |
| 成本 | 要动的代码，以及风险（路由、虚拟列表、共享状态） |

列完就停，问要做哪几条，等回答。选中之后，先按平台说明做法、列出要改的文件，再等一次确认。然后一条一条做。

### 原帖演示的七个场景

下面是介绍帖里点名的七段，和视频对应。更细的接法在后面的 a–g。

| 场景 | 接上的方式 |
|---|---|
| 导航 | 行里的标题长成这一页的标题，上一页的标题收成返回按钮；页面向下滚时，大标题收进顶栏 |
| 卡片 | 卡片长成正文，图片长成头图；往下拖时，沿同一条路径缩回去 |
| 播放器 | 迷你播放条长成完整播放页，封面、标题和按钮各自走到页里的位置 |
| 搜索 | 输入框移到顶部；一边打字，命中的条目往结果区移，其余的淡出 |
| 图表 | 数据卡长成图表，卡上的小柱变成图表里的柱 |
| 菜单 | 菜单从按钮所在的位置长出来；新加的一项落进第一格，其余项依次让开 |
| 标签栏 | 选中胶囊先被拉长，再落到新标签上 |

### 八种该接上的跳变

| | 什么时候 | 怎样才算接上 |
|---|---|---|
| a | 一行或一条目打开详情 | 行标题长成页标题，图片长成头图，上一页标题缩成返回按钮。页面向下滚时，大标题收进顶栏，并落在控制行的中线。返回把整段倒放 |
| b | 卡片打开全屏或对话框 | 卡片自己的框、圆角和图片长成目标视图。标题还是同一行字，只缩放。其余内容等卡片大体打开后再淡入。关闭，或往下拖，缩回原来那张卡片的原来位置 |
| c | 一条紧凑的条打开成整块面板：迷你播放器、聊天气泡、上传或下载胶囊、带详情的 toast | 这条就是那块面板。封面、标题和主按钮各自长到面板里的位置。面板在整个过程里不透明。后面的页面退开：缩到大约 **0.94**，并压暗大约 **30%** |
| d | 搜索或筛选 | 输入框上移到顶行，页标题让位，取消或关闭从一侧进入。打字时，命中项从当前位置滑到结果里的位置，其余淡出。桌面上，从输入框打开的命令面板同样处理 |
| e | 摘要打开成详情：指标块到图表、进度环到报告、缩略图到画廊 | 小图形就是大图形（折线摘要是图表的那条线，小柱是图表的柱）。标签移到标题。数字放大，不要拆掉再画一个新数字 |
| f | 控件打开它管的东西：`⋯` 或 `+` 打开菜单或表单，选择器打开列表，日期字段打开选择器 | 面板从控件自己的那个角长出来。条目在面板大体打开之后淡入，相邻大约 **25ms**。高亮跟着指针或手指在条目之间滑。新增、删除或移动时，对象出现在它的位置上，其余的让出空位 |
| g | 标签、分段控制、底栏、侧栏导航 | 只有一个指示器，从旧标签走到新标签。前缘先走，后缘跟上，所以路上是拉长的，落地时收回来。拉到最长时，高度上的挤压大约 **15%** 就够。标签文字随其余项让位而收起或展开。新内容从被选中的那一侧进来 |
| h | 布局变了：网格和列表互切、排序、拖拽重排、插入、删除、展开和折叠 | 每一项从旧位置走到新位置。换行的项从这一行的末尾离开，从下一行的开头进入，不要斜穿网格 |

### 怎么接

- **一块，不是两块。** 在动的是同一个元素；框架按 id 配对的共享元素也算。量出起点的框和终点的框，用 transform 在两框之间过渡。会触发布局的 `width`、`height`、`top`、`margin` 不要拿来做这段动画。
- **只有一端才有的内容用淡入淡出。** 只在终点存在的内容，等位移走到大约 **45–55%** 再淡入。只在起点存在的内容，在前 **25–35%** 里淡出。字号变了就缩放这行字，不要拿另一份拷贝做交叉淡化。
- **层要老实。** 长出来的面板全程不透明，底下不能透出来。移动中的元素压在邻居上面；落地后回到原来的堆叠顺序。
- **圆角跟着走。** 半径在起点和终点之间平滑插值，用产品自己的圆角曲线。嵌套形状保持等距内缩、圆角同心。
- **时间。** 打开和关闭都是起步快、收尾长：`cubic-bezier(.32, .72, 0, 1)`，或产品自己的弹簧。手机：长成全屏 **450–700ms**，菜单和抽屉 **350–500ms**，指示器 **400–450ms**。桌面大约是这些的 **三分之二**，键盘触发再短一截（原帖没给键盘那一截的毫秒数，不要另造一个；比桌面时长再收一点，拿不准就问）。屏内换位用 ease-in-out，原帖没给这条的控制点，用项目里已有的屏内曲线，没有就用 `ease-in-out`，不要拿上面那条快出慢收的曲线来做换位。错开落在 **20–50ms**。
- **手势。** 手指或指针拖着的时候 **1:1** 跟着走。被拖的卡片一路缩到大约 **0.82**。松手后，用松手的速度收到更近的那一端。
- **随时能打断。** 新的操作从动画当前这一帧接上。不要等动画结束才接受输入，输入也不能被动画挡住。
- **减弱动效。** 状态照样切换，再加一段大约 **150ms** 的交叉淡化。位移和缩放去掉。
- **无障碍。** 焦点移到新状态的标题，或第一个控件；关闭时回到打开它的那个元素。读屏对新状态的宣告保持产品现有的方式。

### 手机与桌面

- **手机：** 上面全部适用。返回滑动、下拉关闭，和按钮走同一条动画。每一帧都守住安全区。
- **桌面：** 同一套模式，更小、更快。卡片或一行通常打开成侧栏或对话框，而不是占满一扇窗；从那张卡片长进那块面板。不要缩放整扇窗口。悬停不要启动会改布局的动画。动画播放时，点击热区留在指针瞄准的地方，第二次点击还能点中。分栏里只动画发生变化的那一栏。
- 窗口缩放和响应式断点不是过渡，不要给它们做动画。

### 做完一条要核对这些

- 第一帧和最后一帧与真实的两态重合。共享元素交还给真实节点时没有一跳。
- 用真实数据：长短文本、缺图、第一项和最后一项、空列表、已经滚过的列表。
- 连点、动画途中返回、手势拖到一半取消。
- 手机和桌面、亮色和暗色、减弱动效、键盘和读屏。
- 静止时每一屏和改之前一样，已有测试仍然通过。

## 实现要点（示例实现，非原帖代码）

原帖点名的曲线是 `cubic-bezier(.32, .72, 0, 1)`。下面三个时长是各区间里的一个点，不是唯一合法值。桌面用大约三分之二，跟项目现有的布局判断走，不要为这条新造断点。`0.5` 和 `0.3` 是落在 45–55% 与 25–35% 里的示例系数。

```css
:root {
  --continuity-curve: cubic-bezier(.32, .72, 0, 1);
  --continuity-full: 580ms;       /* 手机全屏，450–700 */
  --continuity-sheet: 420ms;      /* 菜单 / 抽屉，350–500 */
  --continuity-indicator: 420ms;  /* 指示器，400–450 */
  --continuity-fade-in-at: 0.5;   /* 只在终点出现的内容 */
  --continuity-fade-out-by: 0.3;  /* 只在起点存在的内容 */
}

/* 桌面大约三分之二。选择器换成项目自己的桌面布局标记。 */
:root[data-layout="desktop"] {
  --continuity-full: calc(580ms * 2 / 3);
  --continuity-sheet: calc(420ms * 2 / 3);
  --continuity-indicator: calc(420ms * 2 / 3);
}

.carry { view-transition-name: var(--vt-carry); }

::view-transition-group(.carry) {
  animation-duration: var(--continuity-full);
  animation-timing-function: var(--continuity-curve);
}
::view-transition-new(.carry) {
  animation-delay: calc(var(--continuity-full) * var(--continuity-fade-in-at));
  animation-duration: calc(var(--continuity-full) * (1 - var(--continuity-fade-in-at)));
}
::view-transition-old(.only-start) {
  animation-duration: calc(var(--continuity-full) * var(--continuity-fade-out-by));
}

@media (prefers-reduced-motion: reduce) {
  /* 去掉组上的位移 / 缩放，只留一段短淡化 */
  ::view-transition-group(*) { animation: none; }
  ::view-transition-old(*),
  ::view-transition-new(*) {
    animation-duration: 150ms;
    animation-delay: 0s;
    animation-timing-function: linear;
  }
}
```

没有 View Transitions，或需要跨文档而当前环境不支持时，退回一次即时更新，或用 FLIP：先量、再改状态、再量，用 transform 把差值补上。Motion 的 `layout` / `layoutId`、Svelte 的 `flip`、Vue 的 `TransitionGroup` 做的就是这件事。由路由驱动的页面，从路由的导航钩子里启动过渡。

```ts
export const CONTINUITY = {
  curve: "cubic-bezier(.32, .72, 0, 1)",
  phoneMs: { fullscreen: 580, sheet: 420, indicator: 420 },
  desktopFactor: 2 / 3,
  dragMinScale: 0.82,
  backdropScale: 0.94,
  backdropDim: 0.3,
  menuStaggerMs: 25,
  reducedFadeMs: 150,
} as const;

type Box = { x: number; y: number; w: number; h: number };

/** 宽或高为 0 时不要做除法，调用方改为淡入。transform-origin 用左上，平移加缩放才能对上两个框。 */
export function flipFrom(first: Box, last: Box): string {
  const sx = first.w / last.w;
  const sy = first.h / last.h;
  return `translate(${first.x - last.x}px, ${first.y - last.y}px) scale(${sx}, ${sy})`;
}
```

拖拽过程中按进度把这串 transform 从起点插到终点，卡片缩放走向 `0.82`，不要再套一条缓动。松手才把速度交给弹簧或那条快出慢收的曲线，收到更近的一端。打断时取消当前动画，从计算后的 transform 接到下一段。背景退开（约 0.94、压暗约 30%）只用于 c 这类「条长成面板」，不要套到每一次打开上。

### 各平台原帖点名的入口

对照项目里的实际版本，下面只是入口，不是要求新加一套框架。

| 平台 | 原帖点到的做法 |
|---|---|
| Web | 共享元素上设 `view-transition-name`（同文档，以及分页时的跨文档）；不支持就做一次普通更新。否则 FLIP。Motion 的 `layout` / `layoutId`，Svelte 的 `flip` 与 `crossfade`，Vue 的 `TransitionGroup` |
| iOS | 同一视图里用 `matchedGeometryEffect`。iOS 18 及以后，push 和 sheet 用 `navigationTransition(.zoom(sourceID:in:))` 配 `matchedTransitionSource`。UIKit 在 iOS 18 用 `preferredTransition = .zoom`。要能被手势打断时，用自定义 `UIViewControllerAnimatedTransitioning` 加 `UIViewPropertyAnimator` |
| Android | Compose 的 `SharedTransitionLayout`，`Modifier.sharedElement` 与 `sharedBounds`，`AnimatedContent`，列表用 `Modifier.animateItem`，返回手势用 predictive back。View 体系用 `MaterialContainerTransform` 和共享元素过渡 |
| Flutter | 路由之间用 `Hero`。卡片到页面用 animations 包的 `OpenContainer`。布局变化用 `AnimatedList` 和隐式动画 |
| 桌面应用 | Electron 和 Tauri 走 Web 这条。Tauri 用的是系统 WebView，View Transitions 要在那里实测，并留着回退。原生应用用各自的动画工具，思路相同 |

## 可直接用的 Prompt（精简版）

> Look through this product before editing. Report, in a few lines, the screens, routes, sheets, dialogs, menus, and lists, how each opens, the animation stack and motion tokens already in use, and how reduced motion is handled. Then give at most ten ranked suggestions. Put frequent, disorienting, low-cost jumps first. For each one write where (including phone, desktop, or both), what happens today, which pattern it is, which element should travel, the duration and curve, the value (high, medium, or low), and the code cost or risk (routing, virtualized lists, shared state). Stop and wait for a choice. After a choice, name the approach and the files, then wait again. Build one at a time. This is a change to the existing product: keep actions, content, and resting layout, and change only how one state becomes the next. Do not add a demo frame, sample copy, a new font, a dependency, or a router change without asking. Reuse motion tokens when they exist. The numbers below are defaults.
>
> Travel one shared element, measured at both ends, with transforms rather than width, height, top, or margin. Scale text that changes size. Fade end-only content in after about 45–55% of the move, and fade start-only content out during the first 25–35%. Keep a growing panel opaque, and return it to its stacking order when it lands. Interpolate the corner radius on the product's own corner curve, with concentric nested corners. Open and close on cubic-bezier(.32, .72, 0, 1) or the product's spring. Phone: 450–700ms to a full screen, 350–500ms for menus and sheets, 400–450ms for an indicator. Desktop is about two thirds of that, shorter still from the keyboard. In-screen moves use ease-in-out. Stagger 20–50ms. A drag tracks 1:1, a dragged card scales toward about 0.82, and release finishes toward the nearer end with the release velocity. A new action interrupts from the current frame and never waits on the animation. Reduced motion keeps the state change plus about a 150ms cross-fade, without travel or scale. Move focus to the new heading or first control, and back to the opener on close.
>
> Patterns: (a) a row's title and picture become the detail title and header, the previous title shrinks into Back, and the large title tucks into the bar on scroll; (b) a card grows into a full view or dialog and shrinks back into the same card; (c) a compact bar becomes the full panel while the page behind scales to about 0.94 and dims about 30%; (d) search or filter, including a command palette opened from its field; (e) a sparkline, small bars, ring, or thumbnail becomes the detail graphic, and the number scales; (f) a menu or form grows from the control's corner, items fade in about 25ms apart, and inserts or deletes make their neighbors slide; (g) one tab indicator, leading edge first, with about 15% squash at most; (h) grid, list, sort, insert, and delete, with items that change row leaving at the row's end and entering at the next row's start. On a phone, swipe-back and drag-to-dismiss drive that same transition, inside the safe area on every frame. On desktop, grow a card into a side panel or dialog, do not zoom the window, do not start layout animation from hover, keep the hit target where the pointer is aiming, and animate only the pane that changed. Do not animate resizing or breakpoints.
>
> Check the first and last frames, long and short text, missing images, empty and scrolled lists, interrupted gestures, both color schemes, reduced motion, keyboard, and screen reader. Resting screens and existing tests stay as they were. Web: View Transitions or FLIP (Motion layoutId, Svelte flip, Vue TransitionGroup). iOS: matchedGeometryEffect, or navigationTransition(.zoom) with matchedTransitionSource on iOS 18 and later; UIKit preferredTransition .zoom. Android: SharedTransitionLayout, sharedElement, sharedBounds, animateItem, predictive back. Flutter: Hero, OpenContainer, AnimatedList. Electron and Tauri use the web path, with a fallback where the system WebView lacks View Transitions. Match the API to the versions already in the repo.

上面是改写，不是原文。完整 Prompt 见原帖下方作者的回复。

## 常见错误

- 整屏交叉淡化，或从一条和点击处无关的边上滑入，就算做完了过渡。
- 起点放一个元素、终点再放一个，中间没有同一块东西在走。
- 用 `width`、`height`、`top`、`margin` 把会重排的布局动画出来。
- 字号变化时交叉淡化两份文字，而不是缩放同一行。
- 面板一边长大一边半透明，底下的内容透出来。
- 只在终点才有的内容从第一帧就在，或只在起点的内容一直留到最后。
- 圆角在两态之间跳变，或嵌套圆角不再同心。
- 桌面用了和手机一样长的时长，或悬停就启动会改布局的动画。
- 缩放整扇窗口，或把窗口拉伸、断点切换做成过渡。
- 拖拽还套着缓动，松手却不看速度；卡片被拖时也不缩小。
- 动画播完才允许点击，或新操作把元素打回起点再播。
- `prefers-reduced-motion` 下仍在位移或缩放；或者连状态切换和那约 150ms 的淡化也去掉了。
- 打开后焦点还停在背后，关闭后也不回到当初的按钮。
- 没等用户圈定，就把十条建议一次做完；或顺手加了依赖、改了路由。
- 另做一套带手机框的演示页，而不是改产品里真实的那一屏。
- 第一帧或最后一帧和真实布局有偏差，交接时闪一下。

## 出处

- 原帖：https://x.com/Kubai087/status/2120320609255043510
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例时长的具体取值、淡入淡出系数、桌面时长的 CSS 变量和 FLIP 函数是本仓库示例。键盘触发「再短一截」原帖没有给毫秒数。
