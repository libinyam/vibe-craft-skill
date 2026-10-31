# vibe-craft-skill

> 让 Vibe Coding 出来的产品更有质感 —— 一个给 Claude Code / Cursor 等 AI 编程助手用的前端细节技能（Agent Skill）。

内容整理自 **[@Kubai087（枯白）](https://x.com/Kubai087)** 的 X 栏目 **《999 个小技巧，让你 Vibe 的产品更有质感》**。作者每天更新一条，本仓库跟随更新。

> ⚠️ **致谢与声明**：所有技巧的创意、原始方案、演示视频与原版 Prompt 均归原作者 [@Kubai087](https://x.com/Kubai087) 所有。本仓库内容是**用自己的话改写的规则摘要**，并补充了示例代码，**不是原文**。想看完整原帖和演示效果，请点击下表链接并**关注作者**。

## 这是什么

AI 写出来的界面往往"能用但没质感"。这个技能把一条条具体、可量化的设计小技巧整理成 AI 能直接执行的规则：每条都有原理、数值、示例代码、精简 Prompt 和常见错误。结构参考了 [impeccable](https://github.com/pbakaus/impeccable) 这类前端设计技能：

```
vibe-craft-skill/
├── SKILL.md              # 技能入口：触发条件、核心原则、技巧索引
├── reference/            # 每条技巧一个文件，按分类存放
│   ├── components/       # 组件
│   ├── interaction/      # 交互
│   ├── effects/          # 质感效果
│   ├── shape/            # 形状与圆角
│   └── typography/       # 排版
├── tips.json             # 机器可读索引
├── CONTRIBUTING.md
└── LICENSE
```

当你让 AI 做按钮、菜单、卡片、表单、登录页、视觉效果、文字排版，或者说"让这个页面更有质感"时，它会先读对应的 reference 再动手。

## 安装

**Claude Code（全局）**

```bash
git clone https://github.com/libinyam/vibe-craft-skill.git
mkdir -p ~/.claude/skills
cp -r vibe-craft-skill ~/.claude/skills/vibe-craft
```

**Claude Code（单个项目）**

```bash
mkdir -p .claude/skills
cp -r vibe-craft-skill .claude/skills/vibe-craft
```

**Cursor**

- 支持 Agent Skills 的版本：同样把文件夹放进项目的 `.cursor/skills/vibe-craft/`（或 `.claude/skills/`）。
- 或者把 `SKILL.md` 与 `reference/` 放进项目，在 `.cursor/rules/vibe-craft.mdc` 里写一条规则："做 UI 组件或视觉效果前，先读 `SKILL.md` 并按索引读取对应 reference 文件"。
- 临时使用：在对话里 `@SKILL.md` 或 `@reference/...` 直接引用。

**其他 Agent**：把 `SKILL.md` 当作系统提示 / 规则文件，`reference/` 当作按需读取的知识库即可。

## 技巧目录

| # | 技巧 | 分类 | 本仓库整理 | 原帖 |
|---|---|---|---|---|
| 001 | 富有点击手感的拟物按钮 | 组件 | [reference](reference/components/001-tactile-skeuomorphic-button.md) | [中文](https://x.com/Kubai087/status/2114378773655326873) · [EN](https://x.com/Kubai087/status/2115247182853120484) |
| 002 | 级联菜单安全三角区 + 延时 | 交互 | [reference](reference/interaction/002-cascading-menu-safe-triangle.md) | [原帖](https://x.com/Kubai087/status/2114729985143677258) |
| 003 | Dither 与 ASCII 参数化效果 | 质感效果 | [reference](reference/effects/003-dither-ascii-effect.md) | [原帖](https://x.com/Kubai087/status/2115091203465290087) |
| 004 | 递进式圆角 + 连续曲率圆角（超椭圆） | 形状与圆角 | [reference](reference/shape/004-concentric-radius-superellipse.md) | [原帖](https://x.com/Kubai087/status/2115469767650021796) |
| 005 | 更灵动的拖拽排序：插入线 + 由近及远避让 | 交互 | [reference](reference/interaction/005-drag-reorder-ripple.md) | [原帖](https://x.com/Kubai087/status/2115823717024735708) |
| 006 | 像书一样翻开的卡片 | 组件 | [reference](reference/components/006-book-open-card.md) | [原帖](https://x.com/Kubai087/status/2116198373040759107) |
| 007 | 让你的文字排版更顺眼（以中英文为例） | 排版 | [reference](reference/typography/007-text-wrap-cjk-typography.md) | [原帖](https://x.com/Kubai087/status/2116555064227811412) · [Prompt](https://x.com/Kubai087/status/2116555520274493582) |

持续更新中，新技巧追加方式见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 许可

本仓库自己的代码与改写文字采用 [MIT](LICENSE)。技巧创意归原作者 [@Kubai087](https://x.com/Kubai087) 所有。

---

## English

**vibe-craft-skill** is an Agent Skill (for Claude Code, Cursor and similar coding agents) that teaches the agent concrete, measurable UI craft details — tactile buttons, safe-triangle cascading menus, drag-to-reorder, book-opening cards, dither/ASCII texture effects, concentric radii with superellipse corners, Chinese/English text wrapping, and more.

It is based on **[@Kubai087](https://x.com/Kubai087)**'s X series *"999 Tips to Make Your Vibe-Coded Product Feel Crafted"* (《999 个小技巧，让你 Vibe 的产品更有质感》), one tip per day. **All tip ideas belong to the original author.** The files here are rewritten summaries with example code, not the original posts — follow [@Kubai087](https://x.com/Kubai087) for the originals, demo videos and full prompts.

**Install:** copy this folder to `~/.claude/skills/vibe-craft` (global) or `.claude/skills/vibe-craft` in your project. In Cursor, place it under `.cursor/skills/` or reference `SKILL.md` from a rule in `.cursor/rules/`.

License: MIT for this repo's own code and text; see [LICENSE](LICENSE) for the notice about original authorship.
