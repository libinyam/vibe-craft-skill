# 016 · Toast 和 Notification 怎么选
**Choosing a toast or a notification**

> 分类：组件 Components · 原帖：[2026-11-11](https://x.com/Kubai087/status/2120682993563922586) · Prompt：[作者回复](https://x.com/Kubai087/status/2120685913441456371) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对原帖说明、演示视频字幕和附带 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。时长、尺寸、颜色、阴影和曲线取自原帖；原帖没写成文字的细节不补。帖子时间戳为 2026-11-12 01:00 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-11。原帖附约 37 秒演示视频，观感以视频为准。Prompt 在原帖下方作者的回复里。「实现要点」代码为本仓库示例。

## 原理

网页上的操作反馈，分量要和这件事匹配。

**Toast** 回答用户刚刚在当前页面做完的那一步。它短，不打断，到点自己走。用户知道结果即可。能收回的操作先做完，把退路（通常是撤销）留在这条 Toast 上，用倒计时告诉用户还来得及点。费一点时间的操作先在同一条里显示进度，做完就在原地变成结果。

**Notification** 是用户此刻没有亲手触发的消息：别人发来的，或系统自己送来的，可以和当前操作无关。它比弹窗轻，不必立刻停下手上的事；又比 Toast 重，会一直留着，直到用户处理或关掉，并且能在别处再找到。多条叠在一起，也可以带操作。

必须由用户处理的事，不要交给会消失的 Toast。收不回来的操作，在做之前确认。必须修好才能继续的失败，留在那一项上，直到解决。

这是改现有产品，不是做一条演示。在现有代码、技术栈、组件、token、圆角刻度、字体和文案口气里改。没有征得同意，不要加依赖、新的状态库，或一套新产品里没有的视觉样式。下面的数值都是起点；项目里已有时长、圆角、颜色，就沿用那些。

## 规则 / 数值

### 先看产品，再动手

1. 找出每一处在报告某件事的界面：Toast、snackbar、alert、banner、通知列表、收件箱或动态、角标和数字、行内的成功或错误、浏览器通知或系统推送。snackbar 和 Toast 算同一种临时反馈，原帖没有把它单列成第三种容器。
2. 每一条记下：谁触发的（用户刚刚自己、另一个人、系统、后台任务）、写了什么、有没有按钮、出现在哪、留多久、过后还能不能找到。
3. 找到背后的组件，以及挂在哪（每一页一个，还是整个应用一个）。记下它们怎样向读屏宣告。
4. 用几行把以上发现说清楚，再往下走。

然后给一张表，**先不要改代码**。一行一条消息：

| 栏 | 写什么 |
|---|---|
| 哪里 | 屏幕或组件，以及什么动作触发它 |
| 现在 | 今天实际怎么显示，几个词 |
| 该是 | 状态 Toast、动作 Toast、进度 Toast、Notification，或都不用（并写明用哪一种替代） |
| 为什么 | 用下面的判断题，一行 |
| 改动 | 种类、措辞、按钮、停留、事后去哪找 |

已经做对的标出来。两种都说得通的，写成问句，说明卡在哪一题上。表后面补上缺的：该提供撤销却没有的操作、失败只闪了一条 Toast、后台任务做完从不说、消息过后无处可寻。然后问要改哪几行，等回答。

### 三种 Toast

都出现在用户正在工作的地方，短，不打断，到点自己走。一次只显示一条：新的一条换掉旧的，或排队等前一条走完。两种做法原帖都允许，项目里已有其中一种就沿用，没有就在诊断里问，不要两种都做。

| 种类 | 什么时候 | 里面有什么 | 留多久 |
|---|---|---|---|
| 状态 | 确认一个结果，用户不用再去对一遍。例：「Link copied」 | 一个图标加几个词。没有按钮 | 视频里的短句示例是 **3 秒**后自己离开。Prompt 写成 **3–5 秒**，句子更长就更长。原帖没有按字数换算的公式，在这个区间里取，拿不准就问 |
| 动作 | 这件事已经做完，而且还能收回。先做，不要先问。退路放在 Toast 里，通常是撤销 | 一行字、一个按钮、一个关闭。倒计时用一只渐渐走空的环，秒数写在环心 | 视频写明环倒数 **5 秒**。Prompt 写的是**至少 5 秒**。不点撤销，改动保留。点了撤销，改动撤回，Toast 变成结果（例：「Task restored」），大约 **1.5 秒**后离开 |
| 进度 | 要花一点时间的工作。先表示正在进行，再在**同一条**里变成结果，并给一个去查看的按钮。例：「Uploading brand-guide.pdf」变成「Uploaded to Files, Open」 | 进行中是一只环；完成后是结果加一个按钮 | 工作还在跑就一直留着，完成后 **3 秒** |

指针停在 Toast 上，或焦点在它里面时，计时暂停。

状态 Toast 不放按钮。进度完成后，视频字幕只写了「结果 + 按钮」；Prompt 在描述原地替换时把按钮和关闭写在一起，但没有单独说进度结果上要不要关闭。拿不准就问，不要默认再加一颗关闭。

### Notification

适合用户刚刚没有亲手造成的消息：提及、回复、一笔付款、一个截止时间、安全提醒、一份做完的报告。它可以说得更多：标题、两三行正文、最多两个操作。

- 每一页都在，盖在内容之上。留到被打开、被处理或被关掉。
- 多条叠放。
- 还要有一个能再找到的地方：收件箱，或铃铛后面的列表，带未读数。铃铛上的数字同步到页面标题，后台的标签页也能看见。
- 需要做决定的（安全和付款）即使从屏幕上清掉，仍留在铃铛后面的列表里。

和弹窗的分别：用户需要看到，但不必立刻被打断。

### 都不用

| 情况 | 放哪 |
|---|---|
| 表单或单个字段的错误 | 字段旁边 |
| 用户必须修好的失败（上传失败、付款没过去） | 留在那一项上，附上修复办法，直到修好。几秒后消失的 Toast 不够 |
| 收不回来的操作 | 做之前问。触发器旁边能锚住的，用就地确认；影响面大的用弹窗。做完再补一条 Toast 已经晚了 |
| 只有用户自己去看时才值得出现的变化 | 角标、圆点，或原地改状态。不要再发一条消息 |

### 判断题

一条消息拿不准时，按这个顺序问。前一题已经能定，就不要再往后找理由把它升级成另一种。

1. **谁造成的？** 用户刚刚自己：Toast。另一个人、系统，或用户自己早先在别处做的：Notification。
2. **要不要人处理？** 不用：状态 Toast。只有一个可选的后续（撤销、查看）：动作 Toast。真正要做决定或要回复：Notification。必须修好：留在那一项上。
3. **还能撤销吗？** 能：立刻做，Toast 里给撤销，不要先问。不能：事先确认，不要事后补一条。
4. **有多少字？** 几个词：Toast。一两句，或带名字、金额、地点：Notification。
5. **漏看有没有后果？** 没有：Toast。有：Notification，留到处理完。
6. **人离开时工作还在跑吗？** 人还在这页上时结束，一条 Toast 就够。人已经去了别的标签页或别的应用，能碰到他的只有浏览器或系统通知（先要权限）；产品内部仍写进通知列表。
7. **同时来了好几条？** Toast 一次一条，新的换掉旧的，或排队。Notification 叠放。

两题指向不同容器时，不要替用户拍板。在表里标成问句。

### 外观是另一次选择

表交出去之后，把改外观单独问一次，等回答：

- A. 只改 Toast
- B. 只改 Notification
- C. 两个都改
- D. 都不改外观，只改种类和措辞

选中的每一个，先用这个产品自己的颜色和圆角，写一段短的改前 / 改后（数值和动效），再等一次确认才写代码。然后列出要动的文件，再等一次。一次改一件事。每个操作原本做什么，保持不变。

### Toast 的位置和形状

- **位置**：页面正中偏上，顶栏下 **12px**，盖在内容上。一次一条。没有顶栏时，原帖没说从哪量起，沿用页面已有的安全区，不要另造一个偏移。
- **条**：高 **40**，圆角 **16**（或圆角刻度上最近的一档）。白底，上到下的浅渐变 `#ffffff → #f7f9fc`。1px 边框 `rgba(214, 219, 227, 0.95)`。阴影 `0 16px 36px rgba(17, 24, 39, 0.08), 0 3px 10px rgba(17, 24, 39, 0.05)`。
- **内形同心**：图标内缩 **8**，尺寸 **24**，圆角 **8**。按钮或关闭内缩 **6**，高 **28**，圆角 **10**。也就是内圆角 = 外圆角 − 间距。
- **内容**：图标，然后一行 **14/20** 的字，然后按钮和关闭（有才放）。完成图标是强调色浅底上的对勾；进行中是一只环；撤销剩余时间是走空的环，秒数在环心。按钮是强调色文字，只有悬停才铺一层浅底。图标和文字之间 **8**。按钮和关闭挨在一起；这两项之间的间距原帖没给，沿用组件库已有的紧间距。
- **原帖没给的**：Toast 最大宽度、浅底的不透明度、环的线宽、扫光的带宽。这些沿用产品已有的值。没有现成值时问，不要自己造一个写进规则。

### Toast 的动效

- **进入**：**500ms**，`cubic-bezier(0.25, 0.1, 0.25, 1)`。整条从上方 **10px** 落下，从 **96%** 放大，从 **8px** 模糊里变清；完全不透明略早于结束（原帖没给百分比）。文字和控件作为一整条一起动，不要逐字进来。
- **离开**：**220ms**，ease-in。上移 **6px**，模糊 **6px**，淡出。
- **扫光**：一道亮带只扫一次，**900ms**，ease-out。只在「做成了」的时候扫：状态 Toast 进入时，进度 Toast 变成结果时，撤销之后那条。还在等用户选择的动作 Toast，进来时不扫。
- **原地换内容**（进度变成结果，或已归档变成已恢复）：旧的一行（字、按钮、关闭一起）**180ms** 内模糊并上移 **5px**。新的一行在 **100–400ms** 这段里从下方 **5px** 模糊着进来。条的宽度用 **380ms**、`cubic-bezier(0.32, 0.72, 0, 1)` 收到新宽度。只有图标单独动：旧的缩小并模糊掉，新的从 **-40°**、一半大小转进来。图标没有单独的时长，跟这一次替换走。
- **减弱动效**：状态照样变，再加大约 **150ms** 的淡化。位移、缩放、模糊都去掉。

### Notification 的位置和形状

- **位置**：右上，顶栏下 **12px**，每一页相同，盖在内容上。宽 **344**。挂载点要能跨页面留着；现在是每页各挂一个的，先写进诊断，合并之前问。
- **卡片**：圆角 **24**，内边距 **12**。头像或产品标志 **32**，按钮高 **32**、圆角 **12**，都和卡片同心。标题 **14/20**、半粗，时间在右侧。正文最多三行，**13/20**。按钮最多两个，放右下，主按钮在最后。
- **一叠**：最新的在最前。更旧的被裁到最前一张的高度，只露出边缘。第二张 **95%**，第三张 **90%**。最多露出三张。原帖说从最前一张底下露出 **12**；示例按一步 12 排，第二张偏移 12、第三张偏移 24。原帖没有单独写第三张的像素，若产品里已有自己的叠放偏移，沿用那个。
- **新的一条到来**：从上方 **56px** 滑到这一叠上，**560ms**，`cubic-bezier(0.32, 0.72, 0, 1)`，从一层轻模糊里变清（原帖没给这层模糊的像素）。其余的被往下、往后推一档。被盖住的那张在新的一条盖上来之前仍显示自己的内容，读起来是叠上，不是换掉。
- **悬停**：这一叠展开成列表，每张用自己的高度，间距 **8**，**380ms**，同一条曲线。列表里每张只留发丝边和一层很轻的阴影，避免阴影落在下一张上。叠着的时候，只有最前一张用深阴影。通知卡片的阴影色值原帖没给（上面那两层是 Toast 的），用产品已有的卡片阴影；没有就问。
- **关掉一张**，做法像 macOS：指针在某张上时，左上角出现一颗关闭（**24**，圆角 **10**）。用卡片自己的按钮处理完，也把它移走。移走的一张向右 **48px** 并淡出，**260ms**；下面的在 **400ms** 里补上空位。列表展开时，「全部清除」放在最后一张下面。
- **减弱动效**：同样只留状态变化和大约 **150ms** 的淡化。

### 两者共用

- **读屏**：Toast 用 `role="status"`，礼貌宣告。需要留意的通知在到达时宣告一次。原帖没有写通知的 `role` 或 `aria-live` 取值，沿用产品已有的通知宣告，不要另造 `assertive`。两条都不把焦点从用户正在做的事上拿走。按钮可以用键盘到达。Escape 关掉当前焦点所在的那一条。
- **触摸**：位置相同，滑动可关闭。没有悬停，点一下把这一叠展开。
- **暗色**：同一套规则落在产品自己的暗色表面上，扫光更淡。原帖没有给暗色色值，不要照着亮色反推一套。
- **平台**：Web 上 Toast 用 CSS transition 或 Web Animations API；叠放要量每张卡片的高度，transform 和高度一起过渡。React 的 Sonner 有相近的叠放，可以改现有的；新加这个依赖之前要问。iOS、Android、Flutter 用各平台自己的动画工具，数值相同。要在应用外面碰到人，用系统通知。

### 做完要核对这些

- 短句和长句、缺头像、一次来很多条通知、连续好几条 Toast。
- 撤销被点了、被忽略了、被关掉；进度中途失败；工作还在跑时离开页面。
- 手机和桌面、亮色和暗色、减弱动效、键盘和读屏。
- 已有测试仍然通过。静止时，没被点名的界面和改之前一样。

## 实现要点（示例实现，非原帖代码）

数值常量取自原帖。进入动画里「完全不透明」的关键帧比例、扫光的带宽、通知到达时的模糊像素、叠放第三张的偏移，原帖没有写成数字，下面标了「示例」。

```ts
export const FEEDBACK = {
  toast: {
    offset: 12,
    height: 40,
    radius: 16,
    icon: 24,
    iconInset: 8,
    iconRadius: 8,
    controlHeight: 28,
    controlInset: 6,
    controlRadius: 10,
    gap: 8,
    enterMs: 500,
    enterCurve: "cubic-bezier(0.25, 0.1, 0.25, 1)",
    leaveMs: 220,
    sheenMs: 900,
    lineOutMs: 180,
    lineInFromMs: 100,
    lineInToMs: 400,
    widthMs: 380,
    widthCurve: "cubic-bezier(0.32, 0.72, 0, 1)",
    statusMs: [3000, 5000] as const,
    actionMinMs: 5000,
    progressDoneMs: 3000,
    undoAckMs: 1500,
  },
  note: {
    offset: 12,
    width: 344,
    radius: 24,
    padding: 12,
    picture: 32,
    buttonHeight: 32,
    buttonRadius: 12,
    peek: 12,
    scales: [1, 0.95, 0.9] as const,
    visible: 3,
    arriveMs: 560,
    spreadMs: 380,
    spreadGap: 8,
    curve: "cubic-bezier(0.32, 0.72, 0, 1)",
    closeSize: 24,
    closeRadius: 10,
    dismissMs: 260,
    dismissX: 48,
    gapCloseMs: 400,
  },
  reducedFadeMs: 150,
} as const;

export type FeedbackKind =
  | "status-toast"
  | "action-toast"
  | "progress-toast"
  | "notification"
  | "inline"
  | "keep-on-item"
  | "confirm-first"
  | "badge-only"
  | "ask";

export type FeedbackCase = {
  causedBy: "self-just-now" | "other" | "system" | "self-earlier-elsewhere";
  mustFix: boolean;
  fieldError: boolean;
  glanceOnly: boolean;
  reversible: boolean;
  /** 操作还没发生 */
  notYetDone: boolean;
  /** 已经开始，还要等一会儿 */
  running: boolean;
  needsDecision: boolean;
  optionalFollowUp: boolean;
  words: "few" | "sentence";
  missMatters: boolean;
  /** 人已经离开这页或这个应用，工作还在跑 */
  leftThePage: boolean;
};

/** 判断题的示例。两题打架时返回 ask，让调用方去问，不要在这里拍板。 */
export function classifyFeedback(c: FeedbackCase): FeedbackKind {
  if (c.fieldError) return "inline";
  if (c.mustFix) return "keep-on-item";
  if (!c.reversible && c.notYetDone) return "confirm-first";
  if (c.glanceOnly) return "badge-only";

  if (c.causedBy !== "self-just-now") {
    if (c.words === "few" && !c.missMatters && !c.needsDecision) return "ask";
    return "notification";
  }
  if (c.needsDecision || c.leftThePage) return "notification";
  if (c.reversible && c.optionalFollowUp) return "action-toast";
  if (c.running) return c.missMatters ? "ask" : "progress-toast";
  if (c.words === "few" && c.missMatters) return "ask";
  if (c.words === "sentence" || c.missMatters) return "notification";
  return "status-toast";
}
```

`leftThePage` 为真时，产品内部进通知列表；浏览器或系统通知另要权限，这个函数只决定产品内的容器。人还在本页、工作即将结束时，用进度 Toast。

```css
.toaster,
.notes {
  position: fixed;
  top: 12px;
  /* 原帖没给层级。用产品已有的浮层 token，不要新造一个。 */
  z-index: var(--z-overlay);
  pointer-events: none;
}
.toaster { left: 50%; translate: -50% 0; }
.notes { right: 16px; width: 344px; } /* 右缘 16 是示例；原帖只给了顶栏下 12 和宽度 344 */

.toast {
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  height: 40px;
  /* 左 8：图标内缩。右 6：按钮 / 关闭内缩。垂直方向靠居中，24 的图标剩 8，28 的按钮剩 6。 */
  padding: 0 6px 0 8px;
  border-radius: 16px;
  border: 1px solid rgba(214, 219, 227, 0.95);
  background: linear-gradient(180deg, #ffffff, #f7f9fc);
  box-shadow:
    0 16px 36px rgba(17, 24, 39, 0.08),
    0 3px 10px rgba(17, 24, 39, 0.05);
  pointer-events: auto;
}
.toast .icon { width: 24px; height: 24px; border-radius: 8px; margin-inline-end: 8px; }
.toast .action,
.toast .close { height: 28px; border-radius: 10px; }
.toast .label { font: 14px/20px var(--font-sans); }

@keyframes toast-in {
  from { translate: 0 -10px; scale: 0.96; filter: blur(8px); opacity: 0; }
  /* 完全不透明略早于结束。80% 是本仓库示例，原帖没给百分比。 */
  80% { opacity: 1; }
  to { translate: 0 0; scale: 1; filter: blur(0); opacity: 1; }
}
@keyframes toast-out {
  to { translate: 0 -6px; filter: blur(6px); opacity: 0; }
}
.toast[data-phase="in"] { animation: toast-in 500ms cubic-bezier(0.25, 0.1, 0.25, 1); }
.toast[data-phase="out"] { animation: toast-out 220ms ease-in forwards; }

/* 只在做成了的时候扫一次。亮带宽度和 0.55 不透明度原帖没给，这是示例。 */
.toast[data-sheen="on"]::after {
  content: "";
  position: absolute;
  inset: 0 auto 0 0;
  width: 40%;
  pointer-events: none;
  background: linear-gradient(100deg, transparent, rgb(255 255 255 / .55), transparent);
  animation: toast-sheen 900ms ease-out 1;
}
@keyframes toast-sheen {
  from { translate: -100% 0; }
  to { translate: 350% 0; }
}

.toast[data-swap="out"] .line { animation: line-out 180ms ease forwards; }
.toast[data-swap="in"] .line { animation: line-in 300ms ease 100ms both; }
@keyframes line-out { to { translate: 0 -5px; filter: blur(4px); opacity: 0; } }
@keyframes line-in { from { translate: 0 5px; filter: blur(4px); opacity: 0; } }
/* 行模糊的 4px 是示例。原帖只说 blur，给了 5px 的位移，没给这次模糊的像素。 */
.toast { transition: width 380ms cubic-bezier(0.32, 0.72, 0, 1); }
.toast .icon[data-swap="in"] { animation: icon-in 300ms ease 100ms both; }
@keyframes icon-in {
  from { rotate: -40deg; scale: 0.5; filter: blur(4px); }
}

.note {
  width: 344px;
  border-radius: 24px;
  padding: 12px;
  pointer-events: auto;
}
.note .picture { width: 32px; height: 32px; border-radius: 12px; }
.note .btn { height: 32px; border-radius: 12px; }
.note .title { font: 600 14px/20px var(--font-sans); }
.note .body { font: 13px/20px var(--font-sans); }
.note .close { width: 24px; height: 24px; border-radius: 10px; }

/* 叠放：最新在前。第二、三张各露出 12，缩到 95% / 90%。第三张的 24px 是示例读法。 */
.notes[data-spread="false"] .note:nth-child(1) { transform: none; }
.notes[data-spread="false"] .note:nth-child(2) { transform: translateY(12px) scale(0.95); }
.notes[data-spread="false"] .note:nth-child(3) { transform: translateY(24px) scale(0.9); }
.notes[data-spread="false"] .note:nth-child(n + 4) { visibility: hidden; }
.notes[data-spread="true"] .note + .note { margin-top: 8px; }

.note { transition: transform 380ms cubic-bezier(0.32, 0.72, 0, 1); }
.note[data-phase="in"] { animation: note-in 560ms cubic-bezier(0.32, 0.72, 0, 1); }
@keyframes note-in {
  from { translate: 0 -56px; filter: blur(4px); } /* 4px 是示例，原帖只说 light blur */
}
.note[data-phase="out"] { animation: note-out 260ms ease forwards; }
@keyframes note-out { to { translate: 48px 0; opacity: 0; } }

@media (prefers-reduced-motion: reduce) {
  .toast[data-phase="in"],
  .note[data-phase="in"],
  .toast .line,
  .toast .icon {
    animation: feedback-fade 150ms linear !important;
  }
  .toast[data-phase="out"],
  .note[data-phase="out"] {
    animation: feedback-fade-out 150ms linear !important;
  }
}
@keyframes feedback-fade {
  from { opacity: 0; }
  to { opacity: 1; }
}
@keyframes feedback-fade-out {
  to { opacity: 0; }
}
```

计时在指针进入或焦点进入时暂停，离开再续上。状态 Toast 的停留在 3–5 秒里取一档，不要写成 `字数 × 常数`。撤销倒计时的环用剩余时间 / 至少 5 秒来画，秒数取整后放在环心。

暗色不要复制上面的 `#ffffff` 和那条边框。把渐变、边框、阴影换成产品的暗色 token，扫光的不透明度再降一档。

## 可直接用的 Prompt（精简版）

> Look through this product before editing. In a few lines, report every place that announces an event (toast, snackbar, alert, banner, notification list, inbox, badge, inline success or error, browser or system notification), what triggers it, what it says, its buttons, where it sits, how long it stays, whether it can be found again, which component it is, where that component is mounted, and how it talks to screen readers. Snackbars count as toasts. Then give a table and do not change code: where, what happens today, what it should be (status toast, action toast, progress toast, notification, or neither — and which instead), why in one line, and what would change. Mark rows that are already right. Mark rows that could go either way as a question. Then list anything missing: an undo that should exist, a failure that is only a toast, a background job that never says it is done, news with nowhere to find it again. Ask which rows to change and wait.
>
> A toast answers something the person just did here. It is short, it does not interrupt, and it leaves by itself. One at a time: the next one replaces it, or waits. Status: an icon and a few words, no buttons, 3 seconds in the short example, 3–5 seconds in general, longer when the sentence is longer. Action: the change is already done; one line, one button (usually Undo), and a close; a ring counts down at least 5 seconds (the demo uses 5) and pauses while the pointer or focus is inside; ignoring it keeps the change; Undo turns the toast into the restored result for about 1.5 seconds. Progress: the same toast shows the work, then becomes the result in place with a button to open it, stays while the work runs, then 3 seconds. A notification is news from someone else or from the system. It can hold a title, a few lines and up to two actions, sits on every page, stays until it is opened, handled or closed, stacks, and remains findable behind a bell or in an inbox, with the count also in the document title. Security and payments stay in that list even after they leave the screen. Field errors sit on the field. A failure that must be fixed stays on the item. An action that cannot be undone is confirmed before it runs. A change that only matters when someone looks is a badge or a dot, with no message. If the person has left the tab or the app while work is still running, a browser or system notification (with permission) is what can reach them, and the in-app list gets a copy too.
>
> After the table, offer restyling as a separate choice and wait: toast only, notifications only, both, or neither. For each one chosen, show a short before and after in this product's own colors and corner scale, then wait again. List the files and wait. Change one thing at a time. Toast: top center, 12 under the top bar, 40 tall, 16 radius, gradient #ffffff to #f7f9fc, 1px border rgba(214, 219, 227, 0.95), shadow 0 16px 36px rgba(17, 24, 39, 0.08), 0 3px 10px rgba(17, 24, 39, 0.05). Icon inset 8, 24 with radius 8; button or close inset 6, 28 tall, radius 10. One line at 14/20, gap 8. Enter 500ms on cubic-bezier(0.25, 0.1, 0.25, 1) from 10px above, 96% scale and 8px blur, as one piece. Leave 220ms ease-in, up 6px, blur 6px. A sheen crosses once in 900ms ease-out only when something is done, not while an action toast is still waiting. Swap the line in place: old line up 5px and out in 180ms, new line in from 5px below between 100 and 400ms, width 380ms on cubic-bezier(0.32, 0.72, 0, 1); the icon alone turns in from -40 degrees at half size. Notification: top right, 12 under the top bar, 344 wide, on every page. Card radius 24, padding 12; picture 32 and buttons 32 tall with radius 12; title 14/20 semibold with the time on the right; up to three lines at 13/20; up to two buttons, primary last. Newest in front; older cards clipped to that height, at 95% and 90%, peeking 12, at most three. A new card drops from 56px above in 560ms on the same curve while the covered card keeps its content until it is covered. Hover spreads them to their own heights, 8 apart, in 380ms; stacked, only the front card has the deep shadow. Close is a 24 control with radius 10 at the top-left on hover; handling a button removes the card too; it leaves 48px to the right in 260ms and the gap closes in 400ms. Clear all sits under the last card while spread. Reduced motion keeps the state change and about a 150ms fade, without travel, scale or blur. Toasts use role="status". A notification that needs attention is announced once. Neither takes focus. Escape closes the focused one. Touch swipes to dismiss and spreads the stack on tap. Dark mode uses this product's dark surfaces, with a fainter sheen. Do not add a dependency (including Sonner), a state library, or a new visual pattern without asking. Reuse motion and color tokens when they exist. Check short and long text, a missing picture, many notifications, several toasts in a row, undo used or ignored, a progress that fails, leaving while work is still running, both layouts, both color schemes, reduced motion, keyboard and screen reader. Existing tests stay passing.

上面是改写，不是原文。完整 Prompt 见[作者回复](https://x.com/Kubai087/status/2120685913441456371)。

## 常见错误

- 收不回来的操作做完了，再补一条 Toast。确认要放在做之前。
- 必须修好的失败（上传失败、付款没过去）只闪三秒，人来不及看，也没法回到那一项。
- 字段错误做成 Toast 或通知，而不是留在字段旁边。
- 「链接已复制」这种几个词的结果，做成一条要处理的通知。
- 别人的提及、截止时间、安全提醒做成会自己消失的 Toast，事后再也找不到。
- 安全和付款从屏幕上清掉之后，铃铛后面的列表里也没有了。
- 同时叠好几条 Toast，或通知一次只许一条、新的把旧的换掉。
- 进度结束时拆掉这条、再在别处新开一条结果。
- 不点撤销，改动却被撤回；或点了撤销，Toast 不告诉用户已经恢复。
- 动作 Toast 不到 5 秒就走，或倒计时在悬停和键盘焦点期间还在走。
- 还在等用户选撤销的 Toast，进来时就扫一道光。
- 文字逐字跳入，而不是整条一起动。
- 通知把焦点抢走，或打开之后不宣告、关掉也不把焦点还回去。
- 叠放超过三张还在露，或悬停（触摸上则是点按）不会展开。
- 被盖住的那张在新通知还没盖上来时内容就已经空了。
- `prefers-reduced-motion` 下仍在位移、缩放或模糊。
- 暗色照抄 `#ffffff → #f7f9fc` 和那条浅色边框。
- 表还没给人看，就开始改组件；或顺手装上 Sonner。
- 另做一页演示，而不是改产品里真实的那条提示。
- 原帖没给的最大宽度、环的线宽、扫光带宽、暗色色值，被写成了新的规定。

## 出处

- 原帖：https://x.com/Kubai087/status/2120682993563922586
- Prompt 回复：https://x.com/Kubai087/status/2120685913441456371
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例里的进入不透明关键帧（80%）、行模糊像素（4px）、扫光带宽（40%）和不透明度（0.55）、通知到达模糊（4px）、叠放第三张偏移（24px）、通知区右缘（16px）是本仓库取值，原帖没有这些数字。
