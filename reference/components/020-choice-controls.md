# 020 · 开关、复选框、单选和下拉，怎么选？
**Choosing a switch, checkbox, radio, segment or dropdown**

> 分类：组件 Components · 原帖：[2026-11-18](https://x.com/Kubai087/status/2123217190052827633) · Prompt：[作者回复](https://x.com/Kubai087/status/2123219668165140511) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对介绍帖、作者站点同一条技巧和附带 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。2–5 与「特别多」来自介绍帖；6 个以上、每项一行说明、结果要马上看得见、无障碍，来自作者站点同一条和官方 Prompt；尺寸和时长来自作者站点这条技巧的参数表。介绍帖没写、只在作者站点和 Prompt 里写明的几点，下面会标明。只在演示视频里的文案不当成新规则。介绍帖时间为 2026-11-19 00:50:00 UTC，Prompt 回复为 2026-11-19 00:59:51 UTC，按本仓库约定把介绍帖记为 America/Los_Angeles 的 2026-11-18。没有英文原帖。原帖附约 62 秒演示视频。X 接口只返回了 Prompt 回复的前 280 字，完整 Prompt 以作者站点同一条为准：[选择控件的诊断与改造](https://vibedesign.tips/zh/020-choice-controls/)（英文页题为 Diagnosing and fixing choice controls）。本文中文标题用介绍帖，英文标题用 Choosing a switch, checkbox, radio, segment or dropdown。「实现要点」代码为本仓库示例。

## 原理

开关、复选框、单选、分段选择、下拉选择和下拉菜单外观相近，经常被混用。作者站点举的例子是：设置页上拨了开关，下面却还有一颗保存，人离开之后设置并没有写上。Prompt 要求先诊断，列成表，等确认再改。

Prompt 把选择收成四组，每组只差一件事：

- **何时生效。** 开关拨下去就生效。复选框随表单一起提交。
- **装的是值还是动作。** 下拉选择装数值、状态或选项。下拉菜单装动作，点了就执行。
- **选项有多少、文案会不会变。** 少而短，摊开。多、长，或以后还会变，收进下拉选择。
- **能选几个。** 恰好一个，用单选。任意多个，用复选框。

这是改产品里已经有的控件，不是做一页演示。在现有代码、技术栈、组件、token 和文案口气里改。没有问过，不加依赖，也不新造一套视觉样式。缺分段选择或带搜索的下拉时，先说缺，并从组件库或平台控件里提一个方案，再决定要不要做。

## 规则 / 数值

先判断，再建议。点头的那些才改。

### 先看产品，再动手

用几行把发现说清楚，再出表。

找出每一处让人选一个值、打开或关掉一件事、或从列表里执行动作的地方：设置和偏好、表单和对话框、注册和引导问题、筛选、工具栏、页头、行尾菜单。

每一处记下：

- 它设置什么，或它做什么。
- 选项有几个，标签有多长，每一项要不要一行说明。
- 能选几个。一个都不选，算不算合理答案。
- 何时生效：立刻，还是等保存或提交。
- 放在哪：设置列表、表单、问卷、工具栏或页头。
- 选完之后屏幕上出现什么。

再找到背后的共用组件：开关、复选框、单选组、分段选择、标签页、下拉选择、带搜索的组合框、菜单。

### 开关还是复选框：何时生效

- **开关**：像灯的开关，拨动的一刻就生效。用在自己会保存的设置和偏好里。旁边不放保存按钮。效果要马上看得见，或很容易看到。Prompt 的例子是通知被打开、一块面板出现。作者站点补了一句：打开桌面通知后，马上出现一条通知。
- **复选框**：表单里的一个答案，提交时才生效。介绍帖说要搭配保存按钮。作者站点写成提交按钮。Prompt 的例子是新建项目对话框里的「设为私有」，以及发送前的「给我也发一份」。
- 一件事的开和关用一个开关。不要做成「开 / 关」两个单选。人得把两项都读完才知道现在是哪一态，看起来也像一个比实际更大的选择。
- 检验：反面如果只是「不是它」，这就是一件事的开或关。两边各自有名字（浅色 / 深色、按月 / 按年），那是多中选一，按下面的选项数量来选。

做错的样子：表单里放开关，旁边还有保存。人拨完就离开，什么都没存上。自己会保存的列表里放复选框，人会去找保存按钮。

### 下拉选择还是下拉菜单：值还是动作

- **下拉选择**：装一个值。选完，框里显示新值。设置类的选择立刻生效。Prompt 的例子是选了语言，页面就换成那种语言。列表里，选中项后面打勾。不要画单选圆圈。
- **下拉菜单**：装动作。例子是导出数据、复制、删除。点了就执行。按钮保持自己的名字（更多、一个「…」图标、导出）。结果出现在别处：一条 Toast、一个新页面，或一次下载。作者站点写明：执行「导出数据」之后，用 Toast 报告结果。菜单放在页头、工具栏和行尾，不放进表单的字段中间。介绍帖说它常用来收纳工具栏或页面顶部的按钮。
- **主按钮加旁边的菜单**：一个主动作旁边放一个图标按钮，打开相关动作。介绍帖的例子是「发送」旁边的「定时发送」「存为草稿」。Prompt 把这叫做 split button，并补了「明天 9:00 发送」「发一封测试给我」。每一项点了就执行，主按钮的名字不变。一个动作比旁边那些常用得多时才这样排。
- **标签页和分段选择**也按同一件事分开。标签页切到另一页内容，本身不写入一个值。分段选择写入一个值，或切换同一份内容的看法。Prompt 的例子是列表 / 看板 / 日历。

做错的样子：菜单按钮变成刚刚点的那一项。下拉选择里的选项是动词，或停在「动作」上、还留着刚执行过的那条。当前值不打开列表就看不见。

### 选项有多少：分段、单选，还是下拉

都是多中选一。

| 选项 | 用 |
|---|---|
| 2–5 个，标签短 | 分段选择。点一下就选中，每一项都留在视野里 |
| 2–5 个，每一项需要一行说明 | 单选，上下排列。说明放在该标签下面。整组放在设置名称下面，和名称同一条左缘，不要挤进控件那一列。这是作者站点和 Prompt 补上的，介绍帖只写了「短」和「更长」 |
| 6 个或更多 | 下拉选择 |
| 标签或条数还会变：人名、数量、日期、人会添加的条目 | 下拉选择。今天只有几条也一样。更长的标签或新的一条，不该把版面顶开。作者站点写了这条 |
| 几百个：时区、国家；Prompt 还写了人 | 下拉选择，列表顶部一个搜索框。输入会收窄列表。没有匹配时要说出来，不要变成空白。Prompt 的例句是没有时区匹配 “xyz” |

介绍帖说：选项更多、文字较长，或选项的值并不固定，就改用下拉选择，免得内容一变界面就跳。选项特别多时，在列表顶部加搜索，并做好没有结果时的空状态。

做错的样子：三个短选项藏在下拉里。得点两次，视野里一次只有一项。一长串选项排成一行，超出尽头或折行。

### 单选还是复选框：能选几个

- **单选**：恰好一个，每个选项都露在外面。用在必须先读完再答的地方：问卷、引导问题、表单里的方案，以及上面那种需要一行说明的选项。选中之后不能取消。当「都不」是合理答案时，把它做成一个选项（暂不确定、都不是）。Prompt 也允许改成复选框列表。两条都写了，表里要问用哪条，不要替人定。
- **复选框**：任意多个，包括一个都不选。上下排列。
- **下拉里面一样**。单选：选中项后面打勾，不画单选圆圈。多选：每一项前面放复选框。框里把已选用芯片显示出来；很多时显示个数。

### 每个控件都要满足

这些来自 Prompt，介绍帖没有逐条写。

- 控件用自己的标签说明它设置什么，不只靠选项上的字。
- 当前值不打开任何东西就能看见。
- 立刻生效的控件把效果显示出来。表单在提交之前，不要表现得像已经发生了。
- 默认值是安全、常见的那一个。不要预先勾上会替人报名的项。
- 控件不可用时，说出原因，不要只把它变灰。
- 键盘和读屏：开关是 `role="switch"`，带 `aria-checked`。单选是一个 `radiogroup`，用方向键移动。下拉选择用 Enter 或 Space 打开，用方向键移动。菜单是 `role="menu"`，Escape 关闭。触控目标至少 44×44。这是 Prompt 的无障碍要求，不是参数表里的控件图形尺寸。

### 尺寸和时长

下表是作者站点这条技巧的参数。Prompt 要求沿用产品自己的组件和 token。产品里已经有尺寸或时长，用产品的。还没有时，用这组作起点。参数表没有给颜色，下面不补。

| 控件 | 参数 |
|---|---|
| 选项数量 | 2–5 个短选项：分段选择。2–5 个、每项需要说明：单选。6 个以上：下拉选择。几百个：带搜索的下拉选择 |
| 开关 | 40×24px，圆角 8px；滑块 18px；切换 220ms，`cubic-bezier(.32, .72, 0, 1)` |
| 分段选择 | 高 36px，内边距 4px；滑块高 28px，圆角 8px；切换 280ms。英文页把这块滑块叫 thumb。没有另给轨道圆角，也没有给这条切换的曲线 |
| 选项列表 | 每项高 36px；圆角 14px；展开 180ms，收起 120ms。没有给曲线 |
| 单选与复选框 | 18px。单选为圆形。复选框圆角 5px |

### 先出表，等确认再改

先给表，先不要改代码。一行一个控件：

| 栏 | 写什么 |
|---|---|
| 哪里 | 哪个屏幕，哪一项设置或问题 |
| 它设置或做什么 | 选项、能选几个、何时生效，写成一句 |
| 现在 | 今天用的控件 |
| 该是 | 该用的控件，或「已经对」 |
| 为什么 | 指向上面哪一组对照，一行 |
| 改动 | 控件之外还要动什么：加上或去掉保存、补上「都不」选项、把动作从表单挪到菜单 |

按上面四组对照分组。已经对的标出来。两边都说得通的，写明卡在哪，然后问。凡是会改变行为的，单独点出：开关改成复选框之后要等保存；动作离开表单、挪到页头。然后问改哪几行，等回答。

### 改点头的那些

- 先列出要动的文件，再等一次。
- 一次改一个控件。它设置什么、做什么，保持不变，除非另外说好。人已经存下的值留下。
- 产品里有的组件就用产品的。没有时用平台自己的：
  - Web：原生 `select`，或带搜索的 listbox（combobox）。开关用 `role="switch"`。单选用 radiogroup。菜单用 menu button。
  - iOS：Toggle；Picker 的 `.segmented` 或 `.menu`；Menu；长列表用可搜索的列表。
  - Android：Switch、Checkbox、RadioButton、SegmentedButton。值用 ExposedDropdownMenu，动作用 DropdownMenu。
  - Flutter：Switch、Checkbox、Radio、SegmentedButton。值用带搜索的 DropdownMenu，动作用 MenuAnchor。

### 做完核对

- 每个开关都是立刻生效。每个复选框都只在提交时生效。
- 每个下拉选择在选中后把值显示出来。每个菜单按钮在执行后仍是自己的名字。
- 没有一对单选在表示开和关。允许「都不」的单选问题，有办法说出来。
- 长列表能搜，没有匹配时会说出来。短的摊开。
- 键盘、读屏、手机和桌面、亮色和暗色。已有测试仍然通过。

### 演示视频里的例子

约 62 秒，虚构应用 Kite。画面上的句子只是例子。下面这些不当成新的规定，也不要拿去改选项个数的门槛。

- 设置里的桌面通知：开关拨动后立刻打开。视频右上角的 Toast 文案是 “Desktop notifications are on.” 作者站点只说「马上出现一条通知」，没有规定这句文案，也没有规定必须是 Toast。
- 新建项目的对话框：复选框「设为私有」「固定到侧栏」。悬停提示 “Not saved yet. So it's one click away.” 提交「创建项目」后，Toast 是 “Website redesign created”。悬停那句是视频文案。
- 语言从 English 换成简体中文，框里是新值，界面立刻换语言。按钮「更多」打开导出所有项目、复制设置链接、恢复默认设置。点导出之后，Toast 是「导出任务已开始，完成后将发送至你的邮箱」，按钮仍叫「更多」。
- 发送按钮旁边的箭头打开「明天 9:00 发送」等项。选完之后，视频的提示是 “The button didn't change”，按钮旁出现 “Scheduled for tomorrow at 9:00”。状态那一行是视频里的做法。
- 默认视图是分段的列表 / 看板 / 日历。谁能看见是上下排列的单选，每项有一行小字。时区下拉带搜索，输入 “Sing” 得到 “(GMT+8) Singapore”。
- 视频里的日期格式是一个下拉，里面有 5 种格式，从 “18 Nov 2026” 选到 “2026-11-18”。作者站点和 Prompt 的规则仍是：6 个或更多，或者值会变，才用下拉选择。不要从这一幕改成「5 个也可以」。
- 问卷「团队有多大」用单选。再次点已选的 “2-10” 取消不了，提示 “Can't be cleared”。点 “Not sure yet” 的提示是 “The way out”。“你会用 Kite 做什么”是复选框，可以同时选。角色是单选下拉，选中项后打勾，没有单选圆圈。合作团队是多选下拉，每项前有复选框。

视频底部的规则句，和上面四组对照是同一套，没有多出一条。

## 实现要点（示例实现，非原帖代码）

只收录参数表里的尺寸和时长，以及 Prompt 写明的 `role`。没有颜色，这里不补。触控 44×44 是 Prompt 的无障碍要求，不写进这组图形尺寸，避免和 18px 的单选、复选框混成一条新尺寸。产品已有 token 时，用产品的，不要为了套这组数新做一套控件。

```css
:root {
  --choice-switch-width: 40px;
  --choice-switch-height: 24px;
  --choice-switch-radius: 8px;
  --choice-switch-thumb: 18px;
  --choice-switch-duration: 220ms;
  --choice-switch-ease: cubic-bezier(.32, .72, 0, 1);

  --choice-segment-height: 36px;
  --choice-segment-padding: 4px;
  --choice-segment-slider-height: 28px;
  --choice-segment-slider-radius: 8px;
  --choice-segment-duration: 280ms;

  --choice-list-item-height: 36px;
  --choice-list-radius: 14px;
  --choice-list-open-duration: 180ms;
  --choice-list-close-duration: 120ms;

  --choice-mark-size: 18px;
  --choice-checkbox-radius: 5px;
}

.choice-switch {
  width: var(--choice-switch-width);
  height: var(--choice-switch-height);
  border-radius: var(--choice-switch-radius);
  /* 参数表只给了时长和曲线，没写过渡的是哪个属性。 */
  transition-duration: var(--choice-switch-duration);
  transition-timing-function: var(--choice-switch-ease);
}
.choice-switch .thumb {
  width: var(--choice-switch-thumb);
  height: var(--choice-switch-thumb);
}

.choice-segment {
  height: var(--choice-segment-height);
  padding: var(--choice-segment-padding);
  /* 280ms 没有配套曲线，这里不补。 */
  transition-duration: var(--choice-segment-duration);
}
.choice-segment .slider {
  height: var(--choice-segment-slider-height);
  border-radius: var(--choice-segment-slider-radius);
}

.choice-list {
  border-radius: var(--choice-list-radius);
}
.choice-list .item {
  height: var(--choice-list-item-height);
}
.choice-list[data-state="open"] { animation-duration: var(--choice-list-open-duration); }
.choice-list[data-state="closed"] { animation-duration: var(--choice-list-close-duration); }

.choice-radio,
.choice-checkbox {
  width: var(--choice-mark-size);
  height: var(--choice-mark-size);
}
.choice-radio { border-radius: 50%; } /* 参数表说圆形。50% 是示例写法。 */
.choice-checkbox { border-radius: var(--choice-checkbox-radius); }
```

列表项的 class 只是示例钩子。参数表说的是选项列表的高度和圆角，没有把这组数写到动作菜单上，所以上面不套 `menuitem`。Prompt 点名的是 `role="switch"`、radiogroup 和 `role="menu"`。

```html
<button type="button" role="switch" aria-checked="false" class="choice-switch">
  <span class="thumb"></span>
</button>
<div role="radiogroup"></div>
<div role="menu"></div>
```

下面这个函数只编码已经写明的对照。对不上，或 Prompt 允许两种做法时，返回 `"ask"`，让调用方去问。它不决定文案、颜色，也不代替上面那张诊断表。

```ts
/** 本仓库示例，非原帖代码。 */
export type ChoiceAdvice =
  | "switch"
  | "checkbox"
  | "radio"
  | "segmented"
  | "select"
  | "select-with-search"
  | "menu"
  | "split-button"
  | "tabs"
  | "ask";

export type ChoiceFacts = {
  /** page：切到另一页，本身不写入值。 */
  holds: "value" | "action" | "page";
  effect: "immediate" | "on-submit";
  /** 反面只是「不是」。浅色 / 深色这种两边都有名字，则为 false。 */
  oppositeIsJustNot: boolean;
  optionCount: number;
  /** 作者写的是「几百」，没有整数门槛。由调用方判断。 */
  hundreds: boolean;
  labels: "short" | "needs-line";
  /** 人名、数量、日期、人会添加的条目。写在「多中选一」那一组。 */
  optionsCanChange: boolean;
  pick: "one" | "many";
  noneIsValid: boolean;
  /** 问卷、引导、方案：必须先读完选项。 */
  mustReadAll: boolean;
  /** 一个动作比旁边的常用得多。 */
  oneActionDominates: boolean;
};

export function adviseChoice(facts: ChoiceFacts): ChoiceAdvice {
  if (facts.holds === "page") return "tabs";
  if (facts.holds === "action") {
    return facts.oneActionDominates ? "split-button" : "menu";
  }

  if (facts.oppositeIsJustNot) {
    if (facts.pick !== "one" || facts.optionCount > 2) return "ask";
    return facts.effect === "immediate" ? "switch" : "checkbox";
  }

  if (facts.pick === "many") {
    // 自己会保存的复选框列表是 Prompt 里的错法，但没有指定换成哪一种。
    if (facts.effect === "immediate") return "ask";
    if (facts.optionCount < 2) return "ask";
    // 「几百」用带搜索的下拉。多项时，Prompt 说每一项前仍是复选框。
    if (facts.hundreds) return "select-with-search";
    // 「条数会变就用下拉」写在多中选一里，不替多项的复选框做决定。
    if (facts.optionsCanChange) return "ask";
    return "checkbox";
  }

  if (facts.hundreds) return "select-with-search";
  if (facts.optionCount >= 6) return "select";

  const few = facts.optionCount >= 2 && facts.optionCount <= 5;
  if (!few) return "ask";

  // 条数会变 → 下拉。2–5 且需要一行说明，或必须先读完 → 单选。两条都碰上时问。
  if (facts.optionsCanChange && (facts.labels === "needs-line" || facts.mustReadAll)) {
    return "ask";
  }
  if (facts.optionsCanChange) return "select";

  if (facts.labels === "needs-line" || facts.mustReadAll) {
    // 单选不能取消。Prompt 允许加「都不」选项，或改成复选框列表。
    return facts.noneIsValid ? "ask" : "radio";
  }
  // 分段选择能不能空选，原文没写。
  return facts.noneIsValid ? "ask" : "segmented";
}
```

`optionCount >= 6` 只走在「多中选一」这条分支里。不要用它把多项复选框改成下拉。日期格式那一幕虽然在下拉里放了 5 项，这里仍按 6 个以上才返回 `"select"`。

## 可直接用的 Prompt（精简版）

> This is a change to the existing product, not a demo. Before editing, find every place a person picks a value, turns something on or off, or runs an action from a list: settings, forms and dialogs, sign-up and onboarding, filters, toolbars, page headers, row menus. For each, note what it sets or does, how many options there are, how long the labels are, whether each needs a line, how many can be picked, whether none is a fair answer, when it takes effect, where it sits, and what shows after a pick. Name the shared components. Say what you found in a few lines.
>
> Then give a table and do not change code. One row per control: where; what it sets or does; what it is now; what it should be, or that it is already right; why, in one line; what else changes, such as a Save button added or removed, a "none" option, or an action moved out of a form. Group the rows by the pairs below. If a case could go either way, say what decides it and ask. Call out every change that alters behaviour, then ask which rows to change and wait. List the files and wait again. Change one approved control at a time. Keep what it sets, and keep values people already saved. Use this product's components. If a segmented control or a select with search is missing, say so and propose one from the component library or the platform before building it. Do not add a dependency or a new visual pattern without asking.
>
> Switch or checkbox, by when it takes effect. A switch acts the moment it is flipped, in settings that save on their own, with no Save button beside it, and the effect is visible at once. A checkbox is one answer in a form and applies on submit. On and off for one thing is a single switch, never two radios. If the opposite is only "not", it is on or off. If both sides have names of their own, it is a choice of one.
>
> Select or menu, by value or action. A select holds a value, shows it in the field, and marks the picked row with a tick, not a radio circle. A menu holds actions, runs the one picked, and the button keeps its own name. The result shows elsewhere, such as a toast. Menus sit in page headers, toolbars and at the end of rows, not among a form's fields. When one action is used far more often than the others, keep it as the main button and put the rest in a menu opened by an icon beside it. Tabs switch to another page and set nothing. A segmented control sets a value, or switches the view of the same content.
>
> How many options, when the person picks one. Two to five short options: a segmented control, all of them visible. Two to five that each need a line: radios stacked under the label, on the same left edge. Six or more: a select. If the labels or the count will change, a select even when only a few exist today, so the layout does not jump. Hundreds, such as time zones, countries and people: a select with a search field at the top of the list, and a line when nothing matches, rather than a blank list.
>
> How many can be picked. Radios: exactly one, every option visible, where the person has to read them all, as in a survey, onboarding, or a plan. A radio cannot be cleared, so when none is a fair answer, add that as an option or use a checkbox list, and ask which. Checkboxes: any number, including none, stacked. Inside a select, one value gets a tick; several values get a checkbox on each row, and the field shows chips, or a count when there are many.
>
> Each control names what it sets. The current value is visible without opening anything. An immediate control shows its effect. A form shows nothing until it is submitted. Defaults are the safe, common choice, and nothing starts ticked that signs the person up. If a control is unavailable, say why. A switch is role="switch" with aria-checked. Radios are one radiogroup, moved with the arrow keys. A select opens with Enter or Space and moves with the arrows. A menu is role="menu" and closes with Escape. Touch targets are at least 44 by 44. On the web, use a native select or a listbox with search, role="switch", a radiogroup, and a menu button. On iOS, use Toggle, a Picker in the segmented or menu style, Menu, and a searchable list. On Android, use Switch, Checkbox, RadioButton, SegmentedButton, ExposedDropdownMenu for values and DropdownMenu for actions. In Flutter, use Switch, Checkbox, Radio, SegmentedButton, DropdownMenu with search, and MenuAnchor. When the product has no size or timing token yet, start from: switch 40×24, radius 8, thumb 18, 220ms on cubic-bezier(.32, .72, 0, 1); segmented control 36 tall with 4 padding, slider 28 tall and radius 8, 280ms; list items 36 tall, radius 14, open 180ms, close 120ms; radio and checkbox 18, radio circular, checkbox radius 5. If tokens already exist, use those. Check that every switch applies at once, every checkbox only on submit, every select shows its value, every menu button keeps its name, no radio pair means on and off, a radio question that allows none can say so, long lists can be searched and say when nothing matches, and short ones stay visible. Keyboard, screen reader, phone, desktop, light and dark. Existing tests still pass.

上面是改写，不是原文。完整 Prompt 在[作者回复](https://x.com/Kubai087/status/2123219668165140511)里。X 接口只返回了这条回复的前 280 字，全文以作者站点同一条为准：https://vibedesign.tips/zh/020-choice-controls/

## 常见错误

- 设置页的开关旁边还有保存。人拨完就离开，设置没有写上。
- 自己会保存的列表里用复选框，人去找一颗并不存在的保存。
- 一件事的开和关做成两个单选。
- 两边都有自己的名字（浅色 / 深色），却做成一个开关。
- 菜单按钮变成刚执行的那一项。下拉选择的选项是动词，或停在刚跑过的动作上。
- 当前值要打开列表才看得见。
- 两到五个短选项藏在下拉里。一长串选项排成一行，超出或折行。
- 每一项都需要一行说明，却塞进分段选择。
- 人名、日期、人会添加的条目今天只有几条，就做成会把版面顶开的分段选择。
- 单选问题允许「都不」，却没有这个选项，也没有改成可以不选的复选框。
- 单选的下拉里画了单选圆圈。多选的下拉里没有复选框，框里也看不出选了什么。
- 几百条选项没有搜索。没有匹配时列表是空白。
- 立刻生效的控件看不到结果。表单还没提交，界面却像已经完成。
- 默认勾上了会替人报名的项。控件不可用时只变灰，不说原因。
- 表还没给人看就开始改。缺分段选择或带搜索的下拉时，不问就新做一套，或加上一个依赖。
- 一次改掉控件原本设置的东西，或丢掉人已经存下的值。
- 把视频里 5 种日期格式读成新门槛。发布的规则仍是 6 个以上，或值会变，才用下拉选择。
- 把 Kite 的 Toast 句子、悬停提示写成必须出现的文案。
- 给参数表没写的颜色、曲线或 44px 图形尺寸补上一套。44×44 是触控目标，不是开关或单选的外形。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2123217190052827633 （2026-11-19 00:50:00 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-18）
- Prompt 回复：https://x.com/Kubai087/status/2123219668165140511 （2026-11-19 00:59:51 UTC。X 接口只返回前 280 字）
- 作者站点同一条（完整 Prompt 和参数表）：https://vibedesign.tips/zh/020-choice-controls/
- 英文页同一条：https://vibedesign.tips/020-choice-controls/ （标题 Diagnosing and fixing choice controls。分段选择的滑块在这里叫 thumb）
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 没有英文原帖。原帖附约 62 秒演示视频。视频里的产品名、Toast 和提示文案，以及日期格式下拉里的 5 个选项，没有写成新规则。
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例里的 CSS 变量、`50%` 圆角写法、列表项的 role 钩子和 `adviseChoice` 是本仓库的。
