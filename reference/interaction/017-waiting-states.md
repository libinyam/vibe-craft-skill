# 017 · 巧用等待状态，让产品响应更丝滑
**Show the wait, and keep the layout still**

> 分类：交互 Interaction · 原帖：[2026-11-13](https://x.com/Kubai087/status/2121449185014845794) · Prompt：[作者回复](https://x.com/Kubai087/status/2121449497171841362) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对原帖说明和附带 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。时长、尺寸和颜色取自原帖与作者回复；原帖没写成文字的细节不补。帖子时间戳为 2026-11-14 03:44:34 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-13。原帖附演示视频，只在视频里的细节本文不写。Prompt 在原帖下方作者的回复里。「实现要点」代码为本仓库示例。

## 原理

等待躲不掉。告诉用户的方式，会改变心情，也会改变顺不顺。有时先把反馈摆出来，回头再改。

按这件事本身来选：

- **转圈**：按钮，加上一条 Toast。适合几秒就能做完的动作，比如刷新、同步。点下去，按钮上的图标就地转；做完，Toast 说结果。轻，不打断手头的事。
- **骨架**：页面结构已经知道。先摆骨架，再在原地换成真内容，版面不跳。信息流、图表、大块内容区常用。
- **进度**：告诉用户走到哪，以及做完是什么样。上传、导入、诊断常用。原帖说它可以放在 Toast 或通知卡片里，也可以嵌在页面或表单里。
- **乐观更新**：这是策略，不是一种图案。几乎总会成功、又好撤销的动作，先显示结果，保存放在后面。失败就退回、说明原因、给出重试。

不到 **300ms** 的等待，通常不单独做加载态。单独做了，会闪一下就没。

这是改现有产品，不是做一页演示。在现有代码、技术栈、组件、token、动效数值和文案里改。没有问过，不要加依赖、数据层库，或一种产品里没有的视觉样式。下面的数值是起点。项目里已有时长、颜色、动效，就沿用那些。

## 规则 / 数值

先判断，再建议。用户点头的那些才改。

### 先看产品，再动手

先用几行把发现说清楚，再出表。

找出每一处等待：页面和路由加载，列表和表格，搜索和筛选，会打到服务器的按钮（保存、发送、同步、删除），上传，导入和导出，后台任务，AI 调用。

每一处记下：

- 什么把它启动。
- 通常要多久。能测就测。估的话分成：缓存、一次请求、好几次、一个任务。
- 现在显示什么：转圈、骨架、什么都没有、挡住整页的遮罩。
- 数据到了，版面会不会动。
- 人能不能继续干活。
- 失败时怎样。
- 能不能取消。

再找到共用的转圈、骨架、进度组件，数据层（fetch 封装、React Query、SWR、Apollo，或一个 store），以及全局的加载条或遮罩。

### 诊断表

改代码之前交一张表。一行一处等待，按种类分组。

| 栏 | 写什么 |
|---|---|
| 哪里 | 屏幕或组件，以及什么动作触发 |
| 现在 | 时长，以及现在显示什么 |
| 该是 | 不显示 / 转圈 / 骨架 / 进度 / 乐观更新 / AI 回答 / 后台，外加为什么 |
| 改动 | 要动什么 |

表后面列出要加或要改的共用件：延时助手、骨架形状、进度通知。等用户选哪几行。

### 按长短

在慢速移动网络上量，不要在 localhost 上量。

| 时长 | 用什么 |
|---|---|
| 不到 300ms | 不单独做加载。例外：按钮图标本身就表示这个动作，按下就可以开始转 |
| 300ms–1s | 转圈，或骨架 |
| 1–10s | 骨架，或写明正在做什么的进度 |
| 超过 10s | 进度、剩余时间，以及一条退路 |

**300ms 之后才出现的东西，至少留 500ms。** 全产品只用一个助手。

一次动作、时长未知、大约 0.3–3 秒：看下面的「短等待」。整块内容、结构已知：用骨架。数得清、大约 3 秒以上的任务：用进度。两处都说得通时，写进诊断表里问，不要替用户拍板。

一次等待只留一个指示。不要按钮上转圈、再加遮罩、再加顶栏，三处一起转。

刷新时继续显示旧数据，旁边一个安静的刷新状态。不要换成骨架，把还有效的内容清掉。

版面在内容到来时不许动。骨架用真实尺寸，给最小高度，图片用固定比例。原帖没给具体比例，沿用产品里已有的图片比例。

失败留在发生的地方。说清失败了什么、接下来做什么，并给出重试。人必须去修的事，一条会消失的 Toast 不够。

### 不到 300ms

不放单独的转圈、骨架或进度条。

结果出现在人正在看的地方，结果本身就是反馈。结果不在眼前（同步、复制、屏幕外的保存），用一条短 Toast 确认。

### 短等待：大约 0.3–3 秒

一次动作，时长未知。显示在人动手的地方。标签和按钮宽度不变。

图标本身就表示这个动作（同步、刷新）：从按下就开始转这个图标。哪怕答案马上回来，也至少转满一整圈。答案没到就一直转。停的时候落在整圈上。圈速见下面的外观。

图标不表示这个动作：换成转圈。转圈在 **300ms** 之后才出现，出现了就至少留 **500ms**。

进行中不接受第二次激活。不要把整页变灰。忙碌中的按钮仍然要能用键盘走到。原帖没指定用哪个属性，沿用产品已有的写法，只要它还留在 Tab 顺序里。

做完，按钮回到原样。一条短 Toast 给结果。例子：「Synced」「1 task updated」。内容区正中偏上，几个词，大约 **3 秒**后离开。

### 结构已知：骨架

页面、列表、卡片、表格，结构已经知道，用骨架。

跟数据无关的部分立刻出现：标题、标签页、筛选、列头。

**300ms** 之后，骨架块放在真实的位置和尺寸上。行高和头像尺寸与真行相同。线条比正文略短，长短略有差别。原帖没给「略短」的百分比，不要写成一条新规定。

一道亮带每 **1.2s** 扫过整份列表，一起动，不是一行一行各扫各的。

数据到了，真行占同一位置，从上到下逐行淡入。每行 **200ms**，彼此错开 **40ms**。什么都不跳。返回的行更少，多出来的骨架块淡出。淡出的时长原帖没给。

空页面正中不要只放一只转圈。

### 数得清的长任务：大约 3 秒以上

上传、导入、导出、批处理。用进度。

说清三件事：正在做什么；走到哪（条数或百分比，再加一条条）；还要多久。有 **1 秒**的数据能用来估，才说剩余时间。在那之前写「Estimating…」。

人会离开这页、工作还继续（导入、导出、批处理）：放在右上的通知里。页面保持能用，每一页都在，留到被处理。

只跟某一项有关的工作，放在那一项自己的位置上，尺寸不变，这样版面不动。

页面保持能用，并提供取消。工作可以不靠人继续，就放到后台，做完再告诉。

工作跑了 **300–500ms** 再显示进度。更早做完的，只给结果。

结束时，同一张卡片在原地变成结果。点它就打开那份结果。

数不清的长任务：写出阶段名，例如「Connecting」「Processing 3 files」。不要一只转个不停的圈。大约 **10s** 之后，说明比平时久，并提供放到后台继续。

原帖允许进度出现在 Toast、通知卡片，或嵌在页面 / 表单里。Prompt 把「人会走开、工作还在跑」放右上通知，把「只跟一项有关」嵌在该项里。短动作做完，用 Toast 报结果。放哪拿不准，写进诊断里问。

### 乐观更新

几乎总会成功、又好撤回来的：标完成、点赞、改名、重排、移动、加一条评论。

结果立刻出现在所有该出现的地方：勾、状态、合计。保存放在后面。

状态安静地放在这一项的固定位置：「Saving」，然后「Saved」，过一会儿淡掉。原帖没给这段淡出的毫秒。不要在改动本身上面盖一只转圈。

失败：每一处都退回。在这一项上用几个词说原因，例如「Couldn’t save」。重试放在原本「Saving」的位置。人打过的字留着。

不要用在：付款、永久删除、发给别人，以及任何不能悄悄放回去的事。

### AI / 流式回答

先用步骤说出正在做什么。例子：「Reading 4 tasks」「Checking 3 events」「Writing the summary」。每一步做完变成对勾。

然后按到来的词往下写。下一个词开始时，前一个从一点模糊里淡入。原帖没给这点模糊的像素。最后一个字后面留一根光标。

从一开始就给回答留位置：一张最终尺寸的卡片，或一个只会向下长的最小高度。

结束时给出 Copy 和 Retry。只有回答经常很长时，才提供停止。

目标是 **1–2s** 内看到第一个词。做不到的话，步骤必须还在往前走。

### 人不必看见的等待

预取、草稿保存、后台同步：什么都不显示，或在固定位置放一个安静状态，例如「Saved」。不要为它们挡住页面。

### 减弱动效与无障碍

减弱动效：每一种状态都留着。去掉扫过的亮带，也去掉旋转。改成一层很轻的透明度脉动，或者什么动效都不要。脉动的时长原帖没给。

无障碍：加载区域加 `aria-busy`。完成和失败用礼貌的 live region。焦点留在原来的地方。忙碌中的按钮仍然能用键盘到达。

### 外观默认值

按产品自己的设计收。下面是起点。

**转圈。** 用产品图标集里的加载图标。没有的话，用淡轨道上的一段环。轨道有多淡，原帖没给。尺寸 **12–16px**，**0.9s** 一圈，强调色或文字色。

在按钮里，图标本身就表示这个动作时，转的是这个图标。前 **18%** 加速，后 **18%** 减速，从静止开始，停在静止。最短一圈 **640ms**。等得更久时大约每秒 **600°**（**1.7s** 转三圈）。停的时候总是整圈。原帖把「大约每秒 600°」和「1.7s 三圈」写在一起，没有收成一个公式，这里也不收。

图标不表示这个动作时，**180ms** 里交叉淡入成转圈，带一点模糊和缩放。模糊和缩放的数值原帖没给。标签和宽度不变。

**结果 Toast。** 高 **40**，圆角 **16**。浅色小块上一个对勾，再加几个词。小块的尺寸和颜色原帖没给，沿用产品图标。内容区正中偏上。从上方 **10px**、一层轻微模糊里落下，**500ms**。大约 **3 秒**后离开。轻微模糊的像素、离开时的动效时长，原帖都没给。

**骨架。** 白底上用抬起的灰 `rgb(240 240 242)`。暗色是白色 **8%**。线条圆角 **6**。图片用圆。**14px** 的字配 **12px** 高的线条。亮带是白色 **75%**、宽 **320**，一次扫过整份列表。暗色下的亮带，原帖没有另给一档。

**进度通知。** 右上。宽 **344**，圆角 **24**，内边距 **14**，右侧 **16**。**32** 的图标块，和旁边的字垂直居中。文件类型是 **18px** 的双色填充图标。双色的具体色值原帖没给。标题例：「Importing tasks.csv」。条数和剩余时间同一行，例：「568 of 1,240 rows · About 2s left」。条高 **4**，强调色，轨道是强调色的 **10%**。不要一排按钮。点卡片打开它说的那件事。悬停时出现关闭，关闭即取消。

进入：从上方 **56px** 落到页面上，从 **4px** 模糊和 **96%** 里出来，**420ms**。这条的缓动曲线原帖没给。

做完，同一张卡片原地换。旧字 **160ms** 模糊出去，新字 **420ms** 进来。例：「Imported 1,240 tasks」「Added to Backlog.」。进度条收起。图标块变成实心绿，上面一个粗的白色对勾，仍是 **18px**。然后留到被打开或关掉。绿色的色值、对勾的笔画粗细、模糊的像素、进度条收起的单独时长，原帖都没给。

**乐观更新。** **20px** 的复选框放在任务图标的位置，悬停才出现，勾上填绿。合计在 **320ms** 里滚到新数字。Saving 用 **12px** 转圈，然后 Saved 加对勾，放在行末那一列的开头，**12px** 灰字。失败时这一行晃 **6px**，两下，共 **360ms**。原因用红色，占掉这一行原本的计数。Retry 占掉 Saving 的位置，直到修好。绿色、红色、灰色的色值原帖没给，用产品已有的成功色、危险色和弱化色。

**AI 回答。** 步骤 **13px**，**16px** 转圈，做完变成对勾。回答 **15/24**。光标宽 **2**，强调色。末尾是 Retry 和 Copy。

### 平台

优先用产品已经在用的。

- **Web**：延时和最短停留做成一个助手，例如 `useDelayedFlag(loading, 300, 500)`。React 用 Suspense，fallback 是骨架；`startTransition` 留住旧内容；`useOptimistic`，或 mutation 的 `onMutate` 加回滚。React Query 或 SWR 在重新请求时留着上一次的数据。流式回答用 `fetch` 的 `ReadableStream`，或 AI SDK 的 `useChat`。亮带画成贴在整份列表上的一条渐变，用 `background-position` 移动。`prefers-reduced-motion` 下停掉。
- **iOS**：`.redacted(reason: .placeholder)`、`ProgressView`、`Task` 取消、在 view model 里做乐观更新并回滚、`AsyncSequence` 接流式文字。
- **Android**：Compose 里在占位形状上用 shimmer 笔刷、带数值的 `LinearProgressIndicator`、Paging 的 `LoadState`、ViewModel 里的乐观状态加回滚、`Flow` 接流式文字。
- **Flutter**：真组件当占位，上面加 shimmer（或 Skeletonizer）、`LinearProgressIndicator(value:)`、`Stream` 接流式文字。
- **Rust / 原生桌面**（egui、iced、GPUI、Tauri）：同几种状态放在 UI model 里。**300ms** 和 **500ms** 的计时放在更新循环里。

### 做完核对

- 限到 Slow 4G，把每一处等待计时。不到 300ms 没有单独的加载。没有闪一下。版面不跳。
- 乐观更新进行中把网络关掉。退回，说明原因，Retry 能用。
- 取消一个长任务，以及中途离开页面。没有丢东西。也没有圈还在转。
- 读屏能听到加载中，也能听到完成。
- 减弱动效下没有扫动的亮带。
- 亮色和暗色都看过。
- 已有测试仍然通过。

## 实现要点（示例实现，非原帖代码）

常量是原帖和 Prompt 里的数。示例把「大约每秒 600°」和「1.7s 三圈」并列留下，不合成一个函数。转圈尺寸在 12–16 里取了上限 16。骨架亮带的起止位置、渐变两端透明，进度换字时的模糊像素，行晃动的节拍，都在注释里标了，并记在文末出处。

```ts
export const WAIT = {
  nothingUnderMs: 300,
  showDelayMs: 300,
  minVisibleMs: 500,
  shortUpToMs: 3000,
  progressShowMs: [300, 500] as const,
  estimateAfterMs: 1000,
  longWayOutMs: 10_000,
  firstWordsMs: [1000, 2000] as const,
  spinnerTurnMs: 900,
  spinnerPx: 16, // 原帖给的是 12–16。16 是本仓库取值，取区间上限。
  actionMinTurnMs: 640,
  actionCruiseDegPerSec: 600,
  actionThreeTurnsMs: 1700,
  actionEasePortion: 0.18,
  iconSwapMs: 180,
  rowInMs: 200,
  rowStaggerMs: 40,
  bandPeriodMs: 1200,
  bandWidthPx: 320,
  toastHeight: 40,
  toastRadius: 16,
  toastDropPx: 10,
  toastEnterMs: 500,
  toastLeaveMs: 3000,
  progressWidth: 344,
  progressRadius: 24,
  progressPad: 14,
  progressPadEnd: 16,
  progressTile: 32,
  progressIcon: 18,
  progressBar: 4,
  progressTrack: 0.1,
  progressEnterMs: 420,
  progressEnterDrop: 56,
  progressEnterBlur: 4,
  progressEnterScale: 0.96,
  progressWordsOutMs: 160,
  progressWordsInMs: 420,
  checkPx: 20,
  totalRollMs: 320,
  savingSpinnerPx: 12,
  savingFontPx: 12,
  shakePx: 6,
  shakeTimes: 2,
  shakeMs: 360,
  stepFontPx: 13,
  stepSpinnerPx: 16,
  answerSize: "15px/24px",
  caretPx: 2,
  skeletonLineRadius: 6,
  skeletonLinePx: 12,
  skeletonTextPx: 14,
} as const;

export type WaitShow =
  | "nothing"
  | "spinner"
  | "skeleton"
  | "progress"
  | "optimistic"
  | "ai"
  | "background"
  | "ask";

/** 诊断表的一行。shouldBe 为 ask 时，两处规则都说得通，先问再改。 */
export type WaitRow = {
  where: string;
  now: string;
  shouldBe: WaitShow;
  why: string;
  change: string;
};

/** 有 1 秒样本才报剩余时间。estimate 由调用方算好再传进来。 */
export function timeLeftLabel(sampleMs: number, estimate: string | null): string {
  if (sampleMs < WAIT.estimateAfterMs || !estimate) return "Estimating…";
  return estimate;
}
```

```tsx
import { useEffect, useRef, useState } from "react";

/**
 * 加载变真之后等 delay 再显示；一旦显示，至少留 min。
 * 中途结束、还没显示过，就什么都不出现。
 * 签名和默认值对应 Prompt 里的 useDelayedFlag(loading, 300, 500)。
 */
export function useDelayedFlag(loading: boolean, delay = 300, min = 500): boolean {
  const [shown, setShown] = useState(false);
  const shownRef = useRef(false);
  const visibleAt = useRef(0);

  useEffect(() => {
    let timer: ReturnType<typeof setTimeout> | undefined;

    if (loading) {
      if (shownRef.current) return;
      timer = setTimeout(() => {
        visibleAt.current = Date.now();
        shownRef.current = true;
        setShown(true);
      }, delay);
    } else if (shownRef.current) {
      const rest = Math.max(0, min - (Date.now() - visibleAt.current));
      timer = setTimeout(() => {
        shownRef.current = false;
        setShown(false);
      }, rest);
    }

    return () => clearTimeout(timer);
  }, [loading, delay, min]);

  return shown;
}
```

按钮图标本身表示动作时，从按下就开始转，不走这个 300ms 延时。最短一圈 640ms；更久的等待大约每秒 600°，或按 1.7s 三圈来理解，停在整圈上。前 18% 加速、后 18% 减速。原帖没给贝塞尔，示例不补一条曲线。

```css
.spin {
  width: 16px; /* 12–16 的上限，本仓库取值 */
  height: 16px;
  color: var(--accent, currentColor);
  /* 0.9s 一圈。原帖没说这一圈要不要加减速，示例用匀速。
     按钮上表示动作的图标另算：前 18% 加速，后 18% 减速。 */
  animation: spin 0.9s linear infinite;
}
@keyframes spin { to { rotate: 1turn; } }

/* 图标不表示动作时，180ms 交叉换成转圈。
   原帖还要一点模糊和缩放，但没给数值，示例只做透明度。 */
.button[data-busy="spinner"] .glyph { transition: opacity 180ms; }

.toast {
  height: 40px;
  border-radius: 16px;
  /* 顶边居中由挂载点负责。原帖没给最大宽度。 */
}
.toast[data-phase="in"] { animation: toast-in 500ms; }
@keyframes toast-in {
  from { translate: 0 -10px; opacity: 0; filter: blur(4px); } /* 4px 是本仓库取值 */
  to { translate: 0 0; opacity: 1; filter: none; }
}

.skel,
.skel-avatar {
  background: rgb(240 240 242);
}
.skel {
  height: 12px;
  border-radius: 6px;
  /* 要比正文略短，长短略有差别。原帖没给宽度百分比。 */
}
.skel-avatar { border-radius: 50%; }

:root[data-theme="dark"] .skel,
:root[data-theme="dark"] .skel-avatar {
  background: rgb(255 255 255 / 8%);
}

/* 一条渐变贴在整份列表上，用 background-position 移动。
   两端透明、从 -320px 到 100% 是本仓库取值，用来让 320px 的带走完全程。 */
.skeleton-list {
  background-image: linear-gradient(
    90deg,
    transparent,
    rgb(255 255 255 / 75%),
    transparent
  );
  background-repeat: no-repeat;
  background-size: 320px 100%;
  animation: skeleton-band 1.2s linear infinite;
}
@keyframes skeleton-band {
  from { background-position: -320px 0; }
  to { background-position: 100% 0; }
}

.row { animation: row-in 200ms both; }
.row:nth-child(1) { animation-delay: 0ms; }
.row:nth-child(2) { animation-delay: 40ms; }
.row:nth-child(3) { animation-delay: 80ms; }
@keyframes row-in { from { opacity: 0; } to { opacity: 1; } }

.progress {
  width: 344px;
  border-radius: 24px;
  padding: 14px 16px 14px 14px;
}
.progress .tile {
  width: 32px;
  height: 32px;
}
.progress .file { width: 18px; height: 18px; }
.progress .bar {
  height: 4px;
  background: color-mix(in srgb, var(--accent) 10%, transparent);
}
.progress .bar > span {
  display: block;
  height: 100%;
  background: var(--accent);
}
.progress[data-phase="in"] { animation: progress-in 420ms; }
@keyframes progress-in {
  from { translate: 0 -56px; filter: blur(4px); scale: 0.96; }
}
.progress[data-swap="out"] .copy { animation: words-out 160ms forwards; }
.progress[data-swap="in"] .copy { animation: words-in 420ms both; }
@keyframes words-out { to { filter: blur(4px); opacity: 0; } } /* 4px 是本仓库取值 */
@keyframes words-in { from { opacity: 0; } } /* 新文案原帖只说 in，没说也要模糊 */

.progress[data-done="true"] .tile {
  /* 原帖：实心绿、粗白对勾，仍是 18px。色值和笔画粗细用产品的成功色。 */
  background: var(--success);
  color: white;
}
.progress[data-done="true"] .file { width: 18px; height: 18px; }

.check { width: 20px; height: 20px; }
.saving {
  font-size: 12px;
  color: var(--quiet); /* 原帖只说 grey */
}
.saving .spin { width: 12px; height: 12px; }

.row[data-failed="true"] { animation: row-shake 360ms; }
@keyframes row-shake {
  /* 四拍是本仓库取值。原帖只给 6px、两下、360ms。 */
  0%, 100% { translate: 0 0; }
  25%, 75% { translate: 6px 0; }
}

.step { font-size: 13px; }
.step .spin { width: 16px; height: 16px; }
.answer { font: 15px/24px var(--font-sans); }
.caret { width: 2px; background: var(--accent); }

@media (prefers-reduced-motion: reduce) {
  .spin,
  .skeleton-list,
  .button[data-busy="icon"] .glyph {
    animation: none;
  }
  /* 示例去掉亮带和旋转。原帖也允许一层很轻的透明度脉动，但没给时长，这里不补。 */
}
```

第 n 行的淡入延迟是 `(n - 1) × 40ms`，每行本身 200ms。上面只写了前三行。合计数字滚到新值用 320ms，沿用产品已有的数字过渡；没有的话再问，不要新装一个依赖。

绿色、红色不要写成新的十六进制。把 `--success`、危险色、`--quiet` 换成产品里已有的 token。暗色骨架用白色 8%，不要把 `rgb(240 240 242)` 原样留在暗色里。

## 可直接用的 Prompt（精简版）

> Read the product before editing. In a few lines, name every wait: page and route loads, lists and tables, search and filters, buttons that call the server (save, send, sync, delete), uploads, imports and exports, background jobs, and AI calls. For each, say what starts it, how long it usually takes (measure, or estimate as cached, one request, several, or a job), what is shown (spinner, skeleton, nothing, or a blocking overlay), whether the layout moves when data arrives, whether the person can keep working, what happens on failure, and whether it can be cancelled. Also name shared spinner, skeleton, and progress pieces, the data layer (a fetch wrapper, React Query, SWR, Apollo, or a store), and any global loading bar or overlay.
>
> Then give a table grouped by kind, and do not change code. One row per wait: where; what happens today (duration and what is shown); what it should be (nothing, spinner, skeleton, progress, optimistic, an AI answer, or background, plus why); what would change. After the table, list shared pieces to add or change (a delay helper, skeleton shapes, a progress notification). Wait for the user to choose. Change only what they approve. Work in the existing code, stack, components, tokens, motion values, and copy. Do not add a dependency, a data library, or a new visual pattern without asking. The numbers below are defaults; reuse this product's timings and colors when they exist.
>
> Under 300ms, no separate loader, except a button whose icon already means the action: that icon may start turning on press. If the result appears where the person is looking, the result is the feedback. If it does not (sync, copy, a save off screen), confirm with a short toast. A single action of unknown length, about 0.3 to 3 seconds, shows where they acted. Label and width stay. An icon that already means the action (sync, refresh) turns from the press: at least one full turn even when the answer is instant, keep turning until the answer lands, and always stop on a full turn. Otherwise the icon becomes a spinner only after 300ms, and that spinner stays at least 500ms. Block a second press. Never grey out the page. When it finishes, the button returns and a short toast gives the result (Synced, 1 task updated): top center, a few words, gone after about 3 seconds.
>
> Known layout (page, list, card, table): a skeleton. Show chrome that does not depend on data at once (title, tabs, filters, column headers). After 300ms, shapes sit at the real size and place: same row height and avatar size, lines a little shorter than the text, lengths varying a little. One light band crosses the whole list together every 1.2 seconds. Real rows take those places, top to bottom, each fading in over 200ms, 40ms apart. Nothing jumps. Extra shapes fade out if fewer rows return. Never a lone spinner in the middle of an empty page. A refresh keeps the old data and a quiet refresh state, rather than swapping in a skeleton.
>
> Countable work over about 3 seconds (upload, import, export, batch): progress. Say what it is doing, how far (a count or a percent, and a bar), and time left once there is a second of data to estimate from. Before that, say Estimating…. When the person moves on while the work continues, show a notification at the top right: the page stays free, the card stays on every page, and it stays until it is handled. Work about one item can sit in that item's place at the same size. Keep the page usable and offer Cancel. If it can finish without them, run it in the background and tell them when it is done. Show progress once the work has run 300 to 500ms. A job that finishes sooner needs only the result. At the end the same card becomes the result, and a click opens it. Work you cannot count: name the stage (Connecting, Processing 3 files) instead of an endless spinner. After about 10 seconds, say it is taking longer than usual and offer to keep it running in the background.
>
> Optimistic is for actions that almost always work and are easy to undo (mark done, like, rename, reorder, move, add a comment). Show the result at once everywhere it appears, and save behind it. Put a quiet status in a fixed place on the item (Saving, then Saved, fading after a moment), never a spinner over the change. On failure, revert every part, say why on the item in a few words (Couldn’t save), put Retry where Saving was, and keep what the person typed. Do not use it for payments, permanent delete, sending to other people, or anything that cannot be quietly put back.
>
> Streamed answers: first say what is happening as steps (Reading 4 tasks, Checking 3 events, Writing the summary), each turning into a check. Then write the words as they arrive, each fading in from a slight blur as the next begins, with a caret after the last. Reserve the answer's space from the start (a card at its final size, or a minimum height that only grows downward). At the end, show Copy and Retry. Offer a way to stop only when answers often run long. Aim for first words within 1 to 2 seconds; when that is not possible, the steps must keep moving. Prefetch, draft save, and background sync: show nothing, or a quiet Saved in a fixed place. Never block the page for them.
>
> Thresholds, measured on a slow mobile network and not on localhost: under 300ms, nothing but a button's own icon turning; 300ms to 1s, a spinner or a skeleton; 1 to 10s, a skeleton, or progress that says what it is doing; over 10s, progress, time left, and a way out. Whatever appears after 300ms stays at least 500ms. One helper, used everywhere, such as useDelayedFlag(loading, 300, 500). The layout never moves when content arrives. One indicator per wait. Failures stay where they happened, say what failed and what to do, and offer Retry. A toast that disappears is not enough when the person must fix something. Reduced motion keeps every state, drops the moving band and the spin, and uses a soft opacity pulse or nothing. Put aria-busy on the loading region, a polite live region for done and failed, leave focus where it was, and keep a busy button reachable from the keyboard.
>
> Fit the look to the product. Spinner: the product's loading icon, or a ring segment on a faint track, 12 to 16px, a turn every 0.9s, accent or text color. In a button, an icon that already means the action turns instead, speeding up over the first 18% and slowing over the last 18% so it starts and stops at rest: one turn in 640ms at the least, and about 600° a second for longer waits (three turns in 1.7s), always ending on a whole turn. Otherwise the icon cross-fades into the spinner over 180ms with a little blur and scale. Label and width stay. Toast: 40px tall, 16px corners, a check on a soft tile and a few words, top center, down 10px from a slight blur over 500ms, gone after about 3s. Skeleton: rgb(240 240 242) on white, white at 8% on dark, 6px corners on lines, circles for pictures, 12px lines for 14px text. The band is white at 75%, 320px wide, crossing the whole list at once. Progress notification: 344px wide, 24px corners, 14px padding and 16px on the right, top right. A 32px tile centered on the words, an 18px two-tone file icon, a title such as Importing tasks.csv, one line such as 568 of 1,240 rows · About 2s left, and a 4px bar in the accent on the accent at 10%. No row of buttons: the card opens what it is about, and a close mark on hover cancels. It comes down 56px out of a 4px blur and from 96% over 420ms. When done, the old words blur out over 160ms and the new ones come in over 420ms (Imported 1,240 tasks, Added to Backlog.), the bar folds away, the tile turns solid green with a heavy white check at the same 18px, and the card stays until it is opened or closed. Optimistic: a 20px checkbox in the task icon's place, shown on hover and filled green when ticked; totals roll to the new figure over 320ms; Saving with a 12px spinner, then Saved with a check, at the start of the row's last column, in 12px grey. On failure the row shakes 6px, twice, over 360ms; the reason in red takes the place of the counts; Retry takes the place of Saving until it is fixed. AI: steps at 13px with a 16px spinner that becomes a check, the answer at 15/24, a 2px caret in the accent, Retry and Copy at the end.
>
> Prefer what the product already uses. On the web, one hook for the delay and the minimum. In React: Suspense with a skeleton fallback, startTransition to keep old content, useOptimistic or onMutate with rollback; React Query or SWR keep the previous data while refetching. Stream text from fetch's ReadableStream or the AI SDK's useChat. Draw the band as one gradient fixed to the list and move it with background-position; stop it under prefers-reduced-motion. iOS: .redacted(reason: .placeholder), ProgressView, Task cancellation, optimistic changes in the view model with rollback, AsyncSequence for streamed text. Android: a shimmer brush over placeholder shapes in Compose, LinearProgressIndicator with a value, Paging LoadState, optimistic state in the ViewModel with rollback, Flow for streamed text. Flutter: real widgets as placeholders under a shimmer or Skeletonizer, LinearProgressIndicator(value:), a Stream for streamed text. Rust or native desktop (egui, iced, GPUI, Tauri): the same states in the UI model, with 300ms and 500ms timers in the update loop.
>
> Check on Slow 4G and time every wait: no separate loader under 300ms, no flash, no layout jump. Turn the network off during an optimistic change: it reverts, explains, and Retry works. Cancel a long job, and leave the page during one: nothing is lost and nothing keeps spinning. A screen reader hears loading and done. Reduced motion has no moving band. Check light and dark. Existing tests still pass.

上面是改写，不是原文。完整 Prompt 见[作者回复](https://x.com/Kubai087/status/2121449497171841362)。

## 常见错误

- 不到 300ms 就单独做加载，闪一下又消失。
- 结构已经知道，却在空页面正中放一只转圈，不用骨架。
- 内容到来时版面跳动。骨架不是真实行的尺寸，图片也没有固定比例。
- 刷新时用骨架换掉还有效的旧数据。
- 付款、永久删除、发给别人，也用乐观更新。
- 人必须去修的失败，只放在一条会消失的 Toast 里。
- 同一次等待叠了好几个指示：按钮转圈，再加遮罩，再加顶栏。
- 一个按钮动作把整页变灰。
- 数不清的长任务只留一只转个不停的圈。大约 10 秒后仍然没有阶段名，也没有放到后台的退路。
- 诊断表还没点头，就开始改代码。
- 另做一页演示，而不是改产品里真实的等待。
- 表示动作的按钮图标转了一半就停。
- 乐观更新时，转圈盖在刚改过的那一块上面。失败后退回了界面，人打过的字却丢了。
- 流式回答不先留位置，字一到，下面的版面被顶下去。
- 预取、草稿保存、后台同步把页面挡住。
- `prefers-reduced-motion` 下亮带还在扫、图标还在转。
- 在 localhost 上量时长，拿去决定要不要加载态。
- 没问过就装上新的数据层或新的动效依赖。
- 原帖没给的模糊像素、绿色色值、线条宽度百分比，被写成了新的规定。

## 出处

- 原帖：https://x.com/Kubai087/status/2121449185014845794 （2026-11-14 03:44:34 UTC，按本仓库约定记为 2026-11-13）
- Prompt 回复：https://x.com/Kubai087/status/2121449497171841362 （2026-11-14 03:45:49 UTC）
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 本文是改写整理，不是原帖，也不是原版 Prompt。原帖附演示视频，只在视频里的细节没有写入。
- 示例里这些数是本仓库取值，原帖没有：转圈取 12–16px 的上限 16；普通转圈按匀速理解（原帖只给 0.9s 一圈）；Toast 进入模糊 4px；进度换字时旧文案模糊 4px；骨架亮带两端透明，以及从 -320px 移到 100%；行晃动按 0% / 25% / 75% / 100% 拆成四拍。绿色、红色、灰色用产品 token，不新造色值。减弱动效的示例去掉亮带和旋转，不另补脉动时长。按钮图标的前 18% / 后 18% 没有配一条贝塞尔。
