---
name: setup-matt-pocock-skills
description: "为本仓库配置工程类 Skills：设置它的 issue tracker（工单跟踪器）、triage 标签词汇表和领域文档布局。在其他工程类 Skills 首次使用前运行一次。"
disable-model-invocation: true
---

# 配置 Matt Pocock 的 Skills

搭好工程类 Skills 所依赖的每仓库配置：

- **Issue tracker**：工单放在哪里（默认用 GitHub；也开箱支持本地 markdown）
- **Triage labels**：五个标准分诊角色所使用的标签字符串
- **Domain docs**：`GLOSSARY.md` 和 ADR 放在哪里，以及读取它们的消费规则

这是一个由提示词驱动的 skill，不是确定性脚本。先探索，展示你发现的内容，跟用户确认，然后再写入。

## 流程

### 1. 探索

看当前仓库，了解它的起始状态。有什么就读什么，不要靠猜：

- `git remote -v` 和 `.git/config`：这是 GitHub 仓库吗？是哪一个？
- 仓库根目录的 `AGENTS.md` 和 `CLAUDE.md`：是否存在任意一个？其中是否已有 `## Agent skills` 段落？
- 仓库根目录的 `GLOSSARY.md` 和 `GLOSSARY-MAP.md`
- `docs/adr/` 以及任何 `src/*/docs/adr/` 目录
- `docs/agents/`：这个 skill 之前的产出是否已经存在？
- `.scratch/`：说明本地 markdown 的工单跟踪约定已经在使用中
- `triage` skill 装了吗？（有这个 skill 同级的 `triage` skill 目录，或者你的可用 skills 列表里有 `triage`。）这决定 B 节要不要运行。
- monorepo 信号：`pnpm-workspace.yaml`、`package.json` 里的 `workspaces` 字段，或者有自己的 `src/` 且已填充内容的 `packages/*`。只有在真正大型的多包仓库里才会有这些；没有就意味着单上下文，而几乎所有仓库都是单上下文。

### 2. 呈现发现并提问

总结已有什么、缺什么。然后按顺序逐节进行。一节一个问题，答完再进下一节。

每节先给出推荐答案，让用户一句话就能接受。只有当选择确实会分叉时才给一行解释；探索已经定论的节直接跳过（`triage` 没装就跳过 B 节，没有 monorepo 就跳过 C 节）。

**A 节：Issue tracker。**

> 解释："issue tracker"是这个仓库存放工单的地方。`to-tickets`、`triage`、`to-spec` 这类 skill 会从它那里读取并写入。它们需要知道该调用 `gh issue create`、在 `.scratch/` 下写一个 markdown 文件，还是遵循你描述的其他流程。选你在这个仓库里实际用来跟踪工作的地方。

默认倾向：这些 skill 是为 GitHub 设计的。如果 `git remote` 指向 GitHub，就提议 GitHub。如果 `git remote` 指向 GitLab（`gitlab.com` 或自建主机），就提议 GitLab。否则（或者用户另有偏好），提供：

- **GitHub**：工单放在仓库的 GitHub Issues 里（使用 `gh` CLI）
- **GitLab**：工单放在仓库的 GitLab Issues 里（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
- **Local markdown**：工单以文件形式放在本仓库的 `.scratch/<feature>/` 下（适合个人项目或没有远程的仓库）
- **其他**（Jira、Linear 等）：请用户用一段话描述流程；skill 会把它记成自由格式的散文

把选择记录到 `docs/agents/issue-tracker.md`。GitHub 和 GitLab 模板带有一个 "PRs as a request surface" 开关，默认**关闭**。保持关闭，也不要提起它：想要把外部 pull request 纳入分诊队列的用户，之后自己到文件里把这个开关打开。

**B 节：Triage 标签词汇表。** 如果 `triage` skill 没装（探索已经告诉你了），整节跳过，没装的 skill 不需要标签。

如果装了，只问一个问题：

> 要不要保留默认的 triage 标签？（推荐：**保留**）

默认值就是那五个标准角色，每个标签字符串与其名称相同：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。回答**保留**时，原样写入。只有用户说不保留才收集覆盖值，通常是因为他们的跟踪器已经在用别的名字（例如用 `bug:triage` 对应 `needs-triage`），这样 `triage` 就会套用现有标签，而不是造出重复项。

**C 节：领域文档。** 默认用**单上下文**（仓库根目录一个 `GLOSSARY.md` 加 `docs/adr/`）。这适合几乎每个仓库；直接写入，不用问。

只有在探索发现 monorepo 信号时，才提议**多上下文**（根目录一个 `GLOSSARY-MAP.md` 指向各上下文的 `GLOSSARY.md` 文件）。这时确认他们要哪种布局。

### 3. 确认并修改

把下面这些的草稿展示给用户：

- 要加进 `CLAUDE.md` / `AGENTS.md` 中被编辑那一份的 `## Agent skills` 块（选择规则见第 4 步）
- `docs/agents/issue-tracker.md`、`docs/agents/domain.md`、`docs/agents/triage-labels.md` 的内容（最后一个仅在 `triage` 已安装时才写）

让他们在写入前修改。

### 4. 写入

**选定要编辑的文件：**

- `CLAUDE.md` 存在，就编辑它。
- 否则 `AGENTS.md` 存在，就编辑它。
- 两个都不存在，就问用户要创建哪一个；不要替他们选。

`CLAUDE.md` 已存在时绝不创建 `AGENTS.md`（反过来也一样）；永远编辑已经存在的那一份。

如果选定的文件里已经有 `## Agent skills` 块，就地更新它的内容，而不是再追加一份重复的。不要覆盖用户对周边段落的编辑。

该块：

```markdown
## Agent skills

### Issue tracker

[one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.

### Triage labels

[one-line summary of the label vocabulary]. See `docs/agents/triage-labels.md`.

### Domain docs

[one-line summary of layout: "single-context" or "multi-context"]. See `docs/agents/domain.md`.
```

只有在 `triage` 已安装且 B 节运行过时，才包含 `### Triage labels` 子块，才写 `docs/agents/triage-labels.md`。没装时两者都省略。

B 节在 GitHub 或 GitLab 上运行时，为跟踪器逐个创建它缺少的已配置标签（`gh label create` / `glab label create`）。

然后用本 skill 目录里的种子模板作为起点来写这些文档文件：

- [issue-tracker-github.md](./issue-tracker-github.md)：GitHub 工单跟踪器
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md)：GitLab 工单跟踪器
- [issue-tracker-local.md](./issue-tracker-local.md)：本地 markdown 工单跟踪器
- [triage-labels.md](./triage-labels.md)：标签映射（仅在 `triage` 已安装时）
- [domain.md](./domain.md)：领域文档消费规则 + 布局

对于"其他"工单跟踪器，用用户的描述从零写出 `docs/agents/issue-tracker.md`。

### 5. 完成

告诉用户配置已完成，以及现在哪些工程类 skill 会读取这些文件。说明他们之后可以直接编辑 `docs/agents/*.md`；只有在想更换工单跟踪器或从头重来时，才需要重新运行这个 skill。
