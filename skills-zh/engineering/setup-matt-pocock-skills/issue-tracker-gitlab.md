# Issue tracker：GitLab

本仓库的工单和 spec（规格说明）以 GitLab issue 的形式存放。所有操作都使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI。

## 约定

- **创建工单**：`glab issue create --title "..." --description "..."`。多行描述用 heredoc。传 `--description -` 可打开编辑器。
- **读取工单**：`glab issue view <number> --comments`。要机器可读的输出就用 `-F json`。
- **列出工单**：`glab issue list -O json`，并配合合适的 `--label` 过滤器。
- **在工单下评论**：`glab issue note <number> --message "..."`。GitLab 把评论叫作 "notes"。
- **添加 / 移除标签**：`glab issue update <number> --label "..."` / `--unlabel "..."`。多个标签可以用逗号分隔，也可以重复该参数。
- **关闭**：`glab issue close <number>`。`glab issue close` 不接受关闭备注，所以先用 `glab issue note <number> --message "..."` 发出说明，再关闭。
- **Merge request**：GitLab 把 PR 叫作 "merge request"。使用 `glab mr create`、`glab mr view`、`glab mr note` 等，形状与 `gh pr ...` 相同，只是用 `mr` 代替 `pr`，用 `note`/`--message` 代替 `comment`/`--body`。

从 `git remote -v` 推断仓库；在克隆目录里运行时 `glab` 会自动完成这件事。

## 把 merge request 当作分诊面

**MRs as a request surface: no。** _（如果本仓库把外部 merge request 当成功能请求，改为 `yes`；`/triage` 会读取这个开关。）_

设为 `yes` 时，MR 走与工单相同的标签和状态，改用 `glab mr` 的对应命令：

- **读取 MR**：`glab mr view <number> --comments`，diff 用 `glab mr diff <number>`。
- **列出待分诊的外部 MR**：`glab mr list -F json`，然后只保留作者不是项目成员/所有者的 MR（贡献者的 MR，不是维护者正在进行的工作）。
- **评论 / 打标签 / 关闭**：`glab mr note`、`glab mr update --label`/`--unlabel`、`glab mr close`。

与 GitHub 不同，GitLab 给工单和 MR 分别编号，所以只要知道维护者指的是哪一面，`#42` 就不会有歧义。

## 当 skill 说"发布到 issue tracker"

创建一个 GitLab issue。

## 当 skill 说"取回相关工单"

运行 `glab issue view <number> --comments`。

## Wayfinding 操作

由 `/wayfinder` 使用。**map** 是一个工单，它的**子**工单就是各个 ticket（工单）。

- **Map**：一个打了 `wayfinder:map` 标签的单独工单，正文包含 Notes / Decisions-so-far / Fog。`glab issue create --label wayfinder:map`。（在支持原生 epic 的 GitLab 版本上，也可以用 epic 承载 map；带标签的工单在任何版本都能用。）
- **子工单**：描述顶部带 `Part of #<map>` 的工单，标签为 `wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。工单被认领后，指派给当前推进它的开发者。
- **阻塞关系**：使用 GitLab 的**原生 blocking link**，这是规范的、界面上可见的表示方式。通过 `/blocked_by #<n>` 快速动作添加，以 note 形式发出（`glab issue note <child> --message "/blocked_by #<blocker>"`）。原生 blocking link 是 Premium/Ultimate 功能；在免费版（或不可用的环境）回退到在描述顶部写一行 `Blocked by: #<n>, #<n>`。所有阻塞方都关闭后，该工单即为解除阻塞。
- **前沿查询**：`glab issue list -O json`，范围限定到 map 的子工单，剔除有未关闭阻塞方的：指向未关闭工单的原生 `blocked_by` link（`glab api projects/:id/issues/<child-iid>/links`），或 `Blocked by` 行里有未关闭工单，或已有指派人的工单；按 map 中顺序取第一个。
- **认领**：`glab issue update <n> --assignee @me`，这是本次会话的第一次写入。
- **解决**：`glab issue note <n> --message "<answer>"`，然后 `glab issue close <n>`，再往 map 的 Decisions-so-far 追加一个上下文指针（要点 + 链接）。
