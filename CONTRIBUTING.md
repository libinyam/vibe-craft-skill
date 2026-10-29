# Contributing / 新增技巧

本仓库跟随 [@Kubai087](https://x.com/Kubai087) 的栏目《999 个小技巧，让你 Vibe 的产品更有质感》持续更新。新增一条技巧时：

1. **一条技巧一个文件**：`reference/<category>/NNN-<kebab-slug>.md`，`NNN` 为原帖编号（三位数）。
   分类目录：`components`（组件）、`interaction`（交互）、`effects`（质感效果）、`shape`（形状与圆角）；确有需要再新增分类。
2. **文件结构**固定为：标题（中 / 英）→ 出处说明块 → 原理 → 规则 / 数值 → 实现要点（示例代码）→ 可直接用的 Prompt（精简版）→ 常见错误 → 出处。
3. **用自己的话改写**，不要整段粘贴原文或原版 Prompt；数值可以引用，短句 Prompt 可以引用。只写原帖里有的内容，原帖没说清或只在图片 / 视频里的细节要明确标注，不要猜。自己补充的通用实现知识要标明"示例实现 / 本仓库整理"。
4. **同步更新三处索引**：
   - `tips.json`：追加一项（`id`、`title_zh`、`title_en`、`category`、`post_url`、`date` 按 America/Los_Angeles 日期、`file`）；
   - `README.md` 的技巧目录表；
   - `SKILL.md` 的技巧索引（含"什么时候读"）。
5. 每个文件末尾必须链接原帖并致谢作者。

PR 标题建议：`tip: NNN <中文标题>`。
