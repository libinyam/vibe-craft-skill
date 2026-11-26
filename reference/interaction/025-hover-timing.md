# 025 · 选用正确的延时，让你的悬停与切换更加人性化
**Hover and switching delays that follow user intent**

> 分类：交互 Interaction · 原帖：[2026-11-25](https://x.com/Kubai087/status/2125741330328391991) · Prompt：[作者回复](https://x.com/Kubai087/status/2125763356636442921) · 演示：[作者站点](https://vibedesign.tips/zh/025-hover-timing/) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对介绍帖、作者站点同一条和附带 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。四条规则与区间数值来自介绍帖和作者站点；Prompt 里的默认值、分类、指针速度和动效时长会另行标明。「实现要点」代码为本仓库整理 / 示例实现。介绍帖时间为 2026-11-26 00:00:02 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-25。Prompt 回复为 2026-11-26 01:27:33 UTC。介绍帖附约 51 秒演示视频，只在视频里的细节不当成新规则。中文页题为「[选用正确的延时，让你的悬停与切换更加人性化](https://vibedesign.tips/zh/025-hover-timing/)」，英文页题为 [Hover and switching delays that follow user intent](https://vibedesign.tips/025-hover-timing/)。本文中文标题用介绍帖，英文标题用英文页标题。没有英文原帖。

## 原理

悬停延时不该写成一个死数。太短，光标路过就会沿途弹出，满屏闪烁；太长，界面又显得钝。介绍帖的核心是按「用户意图」调整等待和容错：还没看清意图时多等一会儿，已经在看了就不要再让人等。

四条规则：

1. **组内热启动。** 密排的一组里，第一次要等；组内再换，立刻出。
2. **移入容错。** 浮层要让人移进去点，缝隙不能断，离开也不要立刻关。
3. **按体量定快慢。** 遮得越多、里面越好点，打开和离开后的停留就越久。
4. **级联防误触。** 斜向移向子菜单时，途经的选项不要马上把当前层切走。

这是改现有产品，不是另做一页演示。下面的数是起点。项目里已有时长、组件库选项和 token，就沿用那些。没有问过，不要加依赖。

## 规则 / 数值

先看清楚，再建议。点头的那些才改。

### 先看产品

这些来自官方 Prompt。动手之前先找到所有会在悬停时打开的东西：

- 图标按钮和截断文字上的 Tooltip；人、链接、提及、标签的 Hover Card；列表旁的文档或文件预览、链接预览、悬停放大的图、从导航项展开的面板；悬停打开的菜单和子菜单。
- 每一处记下：什么触发、触发器挨得多近（工具栏、头像组、列表）；浮层有多大、挡住什么；里面有没有可点的东西；离触发器多远；现在怎么计时（没有延时、一个固定值、自己的计时器、或某个库的默认）。
- 再找到共用的 Tooltip / HoverCard / Popover、背后的库（Radix、Floating UI、Headless UI、MUI、Ant Design，或手写 hook）、它们的计时器，以及触摸和键盘怎么处理。

用几行说清发现，再往下分拣。

### 组内“热启动”

介绍帖和作者站点写的是区间。并列元素排得密（工具栏、头像组、底部 Dock / 导航栏）时：

- 第一次悬停等 **300–500ms**，把路过滤掉，避免沿途闪出一排提示。
- 第一个已经出现，说明人在主动看：组内再换，**0ms** 立刻切换。
- 光标离开整组约 **400ms** 之后，冷却，下一次又要等。

官方 Prompt 把「标签」（只有几个字、里面没有可点的东西）写成一套可落地的默认：

- 指针在触发器上停住 **500ms** 再打开。移到另一个触发器，等待重新计。
- 已经有一个标签在显示，移到邻居就立刻换。指针还在组里时（工具栏，包括按钮之间的缝），标签跟到新位置，文字就地改，不要先淡出再淡入。
- 离开整组，标签立刻关掉。离开后 **400ms** 内，组里下一个标签仍然立刻出现；过了这 400ms，又要等 500ms。这 400ms 是跳过打开等待的窗口，不是让标签在离开后再留 400ms。
- 按下触发器会收起它的标签；指针离开这个触发器之前，不再出现。

适用：工具栏、头像组、底部 Dock / 导航栏，以及其它密排的轻量提示。

### 移入容错

悬停展开、而且要移进去点击的浮层，最怕光标刚越过边界就关。

介绍帖和作者站点：

- 触发器与浮层之间的缝，铺一层看不见的桥接区，避免半路断触。
- 光标离开浮层后，留 **150–300ms** 缓冲；大面积预览可放到 **400ms**。这段时间内回来，浮层保持打开。

官方 Prompt 按种类给了默认，并把缝算进「里面」：

- **资料卡**（里面有 Follow、Message、Open 这类按钮或链接）：离开触发器或卡片，开始 **300ms** 宽限；宽限内回来就不断。缝跟卡片算同一块，用一块和卡片同宽的不可见区域盖住，穿过它不要开始宽限计时。
- **大预览**（列表旁的文档、链接或文件预览）：离开后 **400ms** 宽限。列表、面板、以及它们之间的缝，都算在里面。

Esc 或点击外部立刻关闭。

适用：下拉菜单、悬浮资料卡、气泡操作面板、大面积预览。

### 按体量定快慢

打开的门槛跟它挡住多少、打扰多少成正比。面积越大、信息越重，越要等用户真的想看再出。

作者站点给的区间：

| 种类 | 打开等待 |
|---|---|
| 一行轻量提示（如快捷键说明） | 300–500ms（与热启动相同） |
| 资料卡这类中等浮层 | 500–700ms |
| 文档预览、富媒体大卡 | 700–900ms |

官方 Prompt 在这些区间里取了可直接用的默认，并补上离开后的停留：

| 种类 | 打开 | 离开后 |
|---|---|---|
| 标签（几个字，不可点） | 500ms | 立刻关掉；组内跳过等待见上一节 |
| 资料卡（里面有可点的东西） | 700ms | 300ms 宽限 |
| 大预览（挡住页面的一块） | 900ms | 400ms 宽限 |

按浮层实际是什么来归类，不要按今天用的组件名。一个名叫 Tooltip、里面却有按钮的，是资料卡。

有些东西不该在悬停时打开：主业是一个动作的、需要打字的、以及「选一项」而不是「看一眼」的菜单。这些应在点击时打开，并在诊断里单独列出来。

### 级联防误触

光标移向右侧子菜单时，轨迹常是斜线，会扫过上下相邻的选项。介绍帖和作者站点：途经其它选项时加 **150–250ms** 切换延时，避免半路把当前子菜单切走。要零等待又不要误切，用基于指针轨迹的安全三角区，见 [002 级联菜单安全三角区](002-cascading-menu-safe-triangle.md)。

适用：多级右键菜单、级联分类导航、嵌套浮层。

官方 Prompt 对「已经打开的资料卡或预览、再换到下一个」另有一套读指针的办法，见下一节。侧向的大面板如果速度判断还不够，再补安全三角，从指针连到面板近边，做法与级联菜单相同。

### 读指针与就地替换

这些来自官方 Prompt。

- 记下指针最近的位置，算过去 **90ms** 的速度。超过大约 **0.1px/ms**（100ms 里走了 10px）算快；低于这个值，是在减速或停住。
- 等待打开时：指针进入一个触发器就开始计时，进入另一个就重来。这样一划而过的一排触发器，什么都打不开。
- 已经打开时：另一个同类触发器要接走这块资料卡或预览，必须指针在它上面慢下来约 **60ms**（停下后大约 100–150ms）。快速划过的不算。
- 接走时就地换：浮层滑到新触发器，内容交叉淡入，不要先关再开。
- 所有计时器走同一座钟。指针离开就取消。组件卸载时清掉。同一种浮层同时只开一个。

### 每一种都要守的

这些来自官方 Prompt。

- **动效。** 标签淡入约 140ms、淡出约 60ms。资料卡淡入并下落 4px，约 200ms，出去约 120ms。预览淡入并滑入 8px，约 250ms，出去约 140ms。就地替换交叉淡入约 200ms。关掉永远比打开快。
- 浮层开关时，触发器周围的东西不要跟着动。
- **键盘。** 焦点落到触发器，标签立刻出现。资料卡和预览在 Enter 时打开，或按与悬停相同的延时打开。Esc 关闭，焦点回到触发器。标签用 `aria-describedby` 关联。
- **触摸没有悬停。** 在 `@media (hover: none)` 下，标签用长按或直接写成可见文字，资料卡和预览用点按打开。`:hover` 样式只写在 `@media (hover: hover)` 里，避免点过的按钮还停在悬停外观。
- **减弱动效。** 不要下落、滑入或缩放，只保留淡入淡出。
- **已有组件库时，改它自己的选项，不要在旁边再叠一套计时器。** 例如：Radix `Tooltip.Provider` 的 `delayDuration` 500、`skipDelayDuration` 400；Radix `HoverCard` 的 `openDelay` 700、`closeDelay` 300；Floating UI 的 `useHover` 用 `delay: { open, close }`，资料卡和预览加 `handleClose: safePolygon()`，标签组用 `useDelayGroup`。

### 诊断表

改代码之前交一张表，一行一处：

| 哪里 | 种类（标签 / 资料卡 / 预览 / 应改成点击） | 现在（打开、关闭、缝、换到下一个） | 问题 | 改动 | 数值 |

共用的改动（悬停意图助手、库的选项）和每一处的改动分开写。问要改哪几行，只改点头的那些。

### 改完怎么验

这些来自官方 Prompt。

- 光标扫过工具栏，不出现标签；在一个上停住，500ms 后出现；沿工具栏移动，就地立刻换、不闪；离开超过 400ms，等待回来。
- 资料卡 700ms 后打开，指针能穿过缝并点到里面的按钮。滑出后再在 300ms 内回来，保持打开。
- 从列表斜向移向预览、途经其它行，预览不换。在另一行上停住，就地替换，不要先关再开。
- 触摸和键盘能到达悬停能到达的一切。

## 实现要点（示例实现，非原帖代码）

把官方 Prompt 里的默认值收成一组命名常量。区间（300–500、500–700、700–900、150–250）写在注释里，代码里用 Prompt 给出的那个默认。

```ts
// hover-timing.ts —— 本仓库整理，数字来自官方 Prompt 的默认值
export const HOVER_TIMING = {
  labelOpen: 500,       // 站点区间 300–500ms
  groupSkipDelay: 400,  // 离开整组后，仍可 0ms 切换的窗口
  cardOpen: 700,        // 站点区间 500–700ms
  cardGrace: 300,       // 站点区间 150–300ms
  previewOpen: 900,     // 站点区间 700–900ms
  previewGrace: 400,    // 站点：大预览离开缓冲 400ms
  cascadeSwitchMin: 150,
  cascadeSwitchMax: 250,
  speedWindow: 90,      // 用过去 90ms 算速度
  fastPxPerMs: 0.1,     // 超过则算快（100ms 走 10px）
  swapRest: 60,         // 已打开时，在新触发器上慢下来约 60ms 再接走
  labelIn: 140,
  labelOut: 60,
  cardIn: 200,
  cardDrop: 4,
  cardOut: 120,
  previewIn: 250,
  previewSlide: 8,
  previewOut: 140,
  swapCrossfade: 200,
} as const;

export type Point = { x: number; y: number; t: number };

/** 过去 windowMs 内的平均速度，单位 px/ms。样本不足时视为静止。 */
export function pointerSpeed(samples: Point[], now: number, windowMs = HOVER_TIMING.speedWindow): number {
  const cutoff = now - windowMs;
  const inWindow = samples.filter((p) => p.t >= cutoff);
  if (inWindow.length < 2) return 0;
  const a = inWindow[0], b = inWindow[inWindow.length - 1];
  const dt = b.t - a.t;
  if (dt <= 0) return 0;
  return Math.hypot(b.x - a.x, b.y - a.y) / dt;
}

export const isFast = (pxPerMs: number) => pxPerMs > HOVER_TIMING.fastPxPerMs;
```

用法要点：

- 打开等待在「进入触发器」时启动，进入另一个触发器就取消并重来。组内已有标签在显示时，邻居的打开等待为 0。
- 离开整组立刻收起标签，但把 `groupSkipDelay` 的窗口留下；窗口内再进入组，仍然 0ms。
- 资料卡 / 预览的缝用一块不可见命中区连上，指针在缝里不要开始宽限计时。
- 已打开时，用 `pointerSpeed` 判断是路过还是停住；只有慢下来约 `swapRest` 才就地替换。斜向经过其它行还不够稳时，再用 [002](002-cascading-menu-safe-triangle.md) 的安全区。
- 计时器要能取消，卸载时清掉。同一种浮层只保留一个打开。
- 项目已在用 Radix 或 Floating UI，就改上一节列过的选项，不要在库外面再套一层 setTimeout。

## 可直接用的 Prompt（精简版）

> Review everything in my product that opens on hover. Diagnose first, recommend second, and change only the rows I approve. Work in my code and stack; do not add a dependency without asking. Values below are defaults.
>
> The larger an overlay is, the more it covers, and the more there is to click inside it, the longer it waits to open and the longer it stays after the pointer leaves. Once the person has shown they want to look, stop making them wait.
>
> Find every tooltip, hover card, preview and hover menu. For each one, note the trigger, how close neighbours sit, the overlay's size, whether anything inside is clickable, the gap to the trigger, and how it is timed today. Also find the shared Tooltip / HoverCard / Popover, the library if any, and how touch and keyboard work. Tell me what you found in a few lines.
>
> Sort by what the overlay actually is, not by the component name. A "tooltip" with a button in it is a card. Labels (a few words, nothing to click): open after 500ms at rest; once one is showing, a neighbour appears at once and the text changes in place; leaving the group hides it at once, but for 400ms the next label in the group still appears immediately. Pressing a trigger hides its label until the pointer leaves it. Cards (buttons or links inside): open after 700ms; 300ms grace on leave; the gap counts as the card, covered by an invisible area as wide as the card. Previews (a large panel): open after 900ms; 400ms grace; the list, the panel and the gap all count as inside. Things whose main job is an action, that need typing, or that are a choice rather than a glance, should open on click — list those.
>
> Track pointer speed over the last 90ms; faster than about 0.1px/ms is a pass. The open timer starts on enter and restarts on another trigger, so a sweep opens nothing. While a card or preview is open, another trigger takes it only after the pointer has been slow on it for about 60ms (about 100–150ms after it stops); then swap in place (slide to the new trigger, cross-fade the content), do not close and reopen. If a side panel is still too jumpy, add a safe triangle to its near edge, as in tip 002. One clock, cancelled when the pointer moves on, cleared on unmount. Only one overlay of each kind at a time.
>
> Motion: label in 140ms / out 60ms; card in 200ms with a 4px drop / out 120ms; preview in 250ms with an 8px slide / out 140ms; in-place swap cross-fades in 200ms. Close is always faster than open. Nothing around the trigger moves. Keyboard: focus shows a label at once; a card or preview opens on Enter or after the same delay as hover; Escape closes and returns focus; labels use aria-describedby. Touch has no hover: under @media (hover: none), labels are a long press or visible text, cards and previews open on tap; keep :hover styles inside @media (hover: hover). Reduced motion: fades only, no drop, slide or scale. Prefer the library's own options (Radix Tooltip delayDuration 500 and skipDelayDuration 400; HoverCard openDelay 700 and closeDelay 300; Floating UI useHover delay plus safePolygon for cards and previews, useDelayGroup for labels).
>
> Before any code change, give me one table: Place, Kind (label, card, preview, or click instead), Today, Problem, Change, Values. List shared changes separately. Change only the rows I approve. Then check: a toolbar sweep shows nothing, a rest opens a label at 500ms, moving along the bar swaps in place, leaving for more than 400ms brings the wait back; a card opens at 700ms and can be entered across the gap; slipping out and back within 300ms keeps it; an angled move from a list to its preview does not swap, a rest on another row does, in place; touch and keyboard reach everything hover reaches.

上面是改写，不是原文。完整 Prompt 见[作者回复](https://x.com/Kubai087/status/2125763356636442921)和[作者站点](https://vibedesign.tips/zh/025-hover-timing/)。

## 常见错误

- 全产品只用一个写死的延时，轻量提示和大预览一样快。
- 密排的工具栏第一次就是 0ms，光标路过闪出一排 Tooltip。
- 第一个已经出现，组内再换还要再等 300–500ms。
- 把「离开整组后 400ms 再冷却」理解成标签离开后还要再显示 400ms。正确做法是立刻收起，这 400ms 只用来跳过下一次打开等待。
- 触发器与浮层之间有缝，没有透明桥接，指针一过缝就关。
- 指针刚离开就关资料卡或预览，没有 150–300ms（大预览 400ms）宽限。
- 里面有按钮的浮层，仍按标签的 500ms、离开即关来做。
- 主业是动作、需要打字、或「选一项」的菜单，也在悬停时打开。
- 级联菜单斜向移动时立刻切换当前层，既没有 150–250ms 切换延时，也没有安全三角区。
- 已打开的预览，指针快速划过其它行也会换内容。
- 换到下一个时先关掉再打开，而不是就地替换。
- 计时器不能取消，或同一种浮层同时开了好几个。
- 项目已有 Radix / Floating UI，却在旁边再叠一套 setTimeout。
- `:hover` 样式没有包在 `@media (hover: hover)` 里，触摸点击后还停在悬停外观。
- 浮层开关时，触发器周围的布局跟着动。
- 减弱动效下仍做下落、滑入或缩放。
- 诊断表还没点头就开始改代码，或另做一页演示而不是改产品里真实的悬停。
- 把只在约 51 秒演示视频里出现的细节写成规则。
- 把站点的区间和 Prompt 的默认混成一套新数，或补一条来源里没有的换算。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2125741330328391991 （2026-11-26 00:00:02 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-25）
- Prompt 回复：https://x.com/Kubai087/status/2125763356636442921 （2026-11-26 01:27:33 UTC）
- 作者当日转发介绍：https://x.com/Kubai087/status/2125931439304929339
- 作者站点同一条（中文页，含完整 Prompt）：https://vibedesign.tips/zh/025-hover-timing/
- 英文页同一条：https://vibedesign.tips/025-hover-timing/ （标题 Hover and switching delays that follow user intent）
- 级联安全三角区见本仓库 [002](002-cascading-menu-safe-triangle.md)
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 没有英文原帖。介绍帖附约 51 秒演示视频。只在视频里的细节没有写成新规则。
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例里的 `pointerSpeed` / `isFast` 写法，以及把 Prompt 默认值收成 `HOVER_TIMING` 常量，是本仓库整理 / 示例实现。
