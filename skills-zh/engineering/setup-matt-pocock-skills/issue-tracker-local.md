# Issue tracker：本地 Markdown

本仓库的工单和 spec（规格说明）以 markdown 文件形式存放在 `.scratch/` 里。

## 约定

- 每个功能一个目录：`.scratch/<feature-slug>/`
- spec 是 `.scratch/<feature-slug>/spec.md`
- 实现类工单每个 ticket（工单）一个文件，位于 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，从 `01` 开始编号，绝不写成单个合并的工单文件
- 分诊状态记在每个工单文件顶部附近的 `Status:` 行里（角色字符串见 `triage-labels.md`）
- 评论和对话历史追加在文件底部 `## Comments` 标题之下

## 当 skill 说"发布到 issue tracker"

在 `.scratch/<feature-slug>/` 下新建一个文件（目录不存在就创建）。

## 当 skill 说"取回相关工单"

读取被引用路径的那个文件。用户通常会直接给出路径或工单编号。

## Wayfinding 操作

由 `/wayfinder` 使用。**map** 是一个文件，每个工单对应一个**子**文件。

- **Map**：`.scratch/<effort>/map.md`（正文包含 Notes / Decisions-so-far / Fog）。
- **子工单**：`.scratch/<effort>/issues/NN-<slug>.md`，从 `01` 开始编号，正文里写下问题。`Type:` 行记录工单类型（`research`/`prototype`/`grilling`/`task`）；`Status:` 行记录 `claimed`/`resolved`。
- **阻塞关系**：顶部附近写一行 `Blocked by: NN, NN`。它列出的每个文件都是 `resolved` 时，该工单即为解除阻塞。
- **前沿**：扫描 `.scratch/<effort>/issues/`，找出未关闭、无阻塞、未被认领的文件；编号最小的优先。
- **认领**：在任何工作开始之前，设为 `Status: claimed` 并保存。
- **解决**：在 `## Answer` 标题下追加答案，设为 `Status: resolved`，再往 `map.md` 里 map 的 Decisions-so-far 追加一个上下文指针（要点 + 链接）。
