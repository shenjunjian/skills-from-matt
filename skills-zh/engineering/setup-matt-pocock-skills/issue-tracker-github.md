# Issue tracker：GitHub

本仓库的工单和 spec（规格说明）以 GitHub issue 的形式存放。所有操作都使用 `gh` CLI。

## 约定

- **创建工单**：`gh issue create --title "..." --body "..."`。多行正文用 heredoc。
- **读取工单**：`gh issue view <number> --json number,title,body,labels,comments`。
- **列出工单**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，并配合合适的 `--label` 和 `--state` 过滤器。
- **把工单设为父工单的子工单**：`gh issue create --parent <parent> ...`，或事后用 `gh issue edit <parent> --add-sub-issue <child>`（`gh` 2.94+）。旧版 `gh`：`gh api --method POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>`（database id，取法见下文**阻塞关系**）。没有 sub-issue 能力时，在子工单正文顶部写 `Part of #<parent>`。
- **在工单下评论**：`gh issue comment <number> --body "..."`
- **添加 / 移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **关闭**：`gh issue close <number> --comment "..."`

从 `git remote -v` 推断仓库；在克隆目录里运行时 `gh` 会自动完成这件事。

## 把 pull request 当作分诊面

**PRs as a request surface: no。** _（如果本仓库把外部 PR 当成功能请求，改为 `yes`；`/triage` 会读取这个开关。）_

设为 `yes` 时，PR 走与工单相同的标签和状态，改用 `gh pr` 的对应命令：

- **读取 PR**：`gh pr view <number> --comments`，diff 用 `gh pr diff <number>`。
- **列出待分诊的外部 PR**：`gh api --paginate 'repos/{owner}/{repo}/pulls?state=open' --jq '.[] | select(.author_association | IN("OWNER","MEMBER","COLLABORATOR") | not) | {number, title, author: .user.login, author_association, labels: [.labels[].name]}'`。
- **评论 / 打标签 / 关闭**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub 上工单和 PR 共用一套编号空间，所以光一个 `#42` 可能是其中任何一种：先用 `gh pr view 42` 判断，不行再回退到 `gh issue view 42`。

## 当 skill 说"发布到 issue tracker"

创建一个 GitHub issue。

## 当 skill 说"取回相关工单"

按上文**读取工单**的方式读取。

## Wayfinding 操作

由 `/wayfinder` 使用。**map** 是一个工单，它的**子**工单就是各个 ticket（工单）。

- **Map**：一个打了 `wayfinder:map` 标签的单独工单，正文包含 Notes / Decisions-so-far / Fog。`gh issue create --label wayfinder:map`。
- **子工单**：以 GitHub sub-issue 的形式关联到 map（见**把工单设为父工单的子工单**）。未启用 sub-issue 时，把子工单放进 map 正文的任务列表里，并在子工单正文顶部写 `Part of #<map>`。标签：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。工单被认领后，指派给当前推进它的开发者。
- **阻塞关系**：使用 GitHub 的**原生 issue dependencies**，这是规范的、界面上可见的表示方式。用 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 加一条边，其中 `<blocker-db-id>` 是阻塞方的数字 **database id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，_不是_ `#number`，也不是 `node_id`）。GitHub 会通过 `issue_dependencies_summary.blocked_by` 报告阻塞项（只报未关闭的，也就是实时闸门）。没有 dependencies 能力时，回退到在子工单正文顶部写一行 `Blocked by: #<n>, #<n>`。所有阻塞方都关闭后，该工单即为解除阻塞。
- **前沿查询**：列出 map 的未关闭子工单（`gh issue list --state open`，范围限定到 map 的 sub-issue / 任务列表），剔除有未关闭阻塞方的（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行里有未关闭工单）或已有指派人的工单；按 map 中顺序取第一个。
- **认领**：`gh issue edit <n> --add-assignee @me`，这是本次会话的第一次写入。
- **解决**：`gh issue comment <n> --body "<answer>"`，然后 `gh issue close <n>`，再往 map 的 Decisions-so-far 追加一个上下文指针（要点 + 链接）。
