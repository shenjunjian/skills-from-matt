## 它做什么

`setup-matt-pocock-skills` 回答关于某一个仓库的三个问题：工单住在哪、triage 标签叫什么、领域文档放在哪里。它把答案记录成 `docs/agents/` 下的 markdown 文件。

这些文件是仓库之间唯一不同的东西。Skill 本身在哪儿都一样。它们在运行时读取 `docs/agents/issue-tracker.md`，照它说的做。这就是这套 Skill 不绑定 GitHub 的原因，也是你永远不用为了指向另一个 tracker 而去编辑某个 Skill 文件的原因。用"把这些 Skill 接到一个自定义 issue tracker 上"这样的话来调用它，对任何你能以编程方式连上的东西都成立，且不需要改动 Skill。

它是一个由提示词驱动的 Skill，不是确定性脚本。它读取你的 `git remote`、`CLAUDE.md` 和 `GLOSSARY.md`，把它发现的内容提出来给你看，并在写入任何东西之前等你确认。

## 什么时候用它

你通过输入 `/setup-matt-pocock-skills` 来调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent)（智能体）不会自己去找它。它的元数据刻意标记为不可自动调用，所以没有别的 Skill 能替你触发它。

每个仓库运行一次，在任何其他工程类 Skill 首次使用之前。如果 [triage](https://aihero.dev/skills-triage)、[to-spec](https://aihero.dev/skills-to-spec)、[to-tickets](https://aihero.dev/skills-to-tickets) 或 [wayfinder](https://aihero.dev/skills-wayfinder) 开始猜你的工单该放哪，或者贴出你的 tracker 里并没有的标签，说明这个仓库还没设置好。你也可以在项目进行到一半的仓库里运行它。这个 Skill 读取已经存在的东西，所以之前的工作不会丢失。

## 前置条件

它把内容写进你运行它的那个仓库：

| 它写入 | 位置 |
| --- | --- |
| `issue-tracker.md` | `docs/agents/` |
| `domain.md` | `docs/agents/` |
| `triage-labels.md` | `docs/agents/`，仅在安装了 `triage` Skill 时 |
| 一个 `## Agent skills` 代码块 | `CLAUDE.md` 或 `AGENTS.md` 中已经存在的那一个 |

全部以 markdown 提交。没有用户级或全局模式。配置住在仓库里，所以每个仓库各有一份。

## 三个决策

每个小节都从推荐的答案开始，并跳过它的探查已经答过的问题。多数运行只需要两次确认。

| 决策 | 它提议什么 | 什么时候问 |
| --- | --- | --- |
| **Issue tracker** | 与你 `git remote` 相匹配的那个 | 总是，因为这是唯一一个真实的选择 |
| **Triage 标签** | 保留五个规范名称（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`） | 仅在安装了 `triage` Skill 时 |
| **领域文档** | 单一上下文：根目录一个 `GLOSSARY.md` 加 `docs/adr/` | 仅在它发现 monorepo 信号时，此时它会提供一个多上下文的 `GLOSSARY-MAP.md` |

tracker 的可选项：

| 选项 | 工单住在哪 | 需要什么 |
| --- | --- | --- |
| **GitHub** | 仓库的 GitHub Issues | `gh` CLI |
| **GitLab** | 仓库的 GitLab Issues | `glab` CLI |
| **Local markdown** | 本仓库 `.scratch/<feature>/` 下的文件 | 什么都不需要，连 remote 都不需要 |
| **Other** | 你说的任何地方 | 你用一段话描述这个工作流 |

前三个作为模板随 Skill 提供，开箱可用。Local markdown 是一个完整选项，不是退路。这个 Skill 支持没有 remote 的个人项目。一条注意：如果你用 GitHub，就不要用 local markdown。两者是互斥的，选一个。

"Other" 同样是完整选项。Jira、Linear、Azure DevOps 和 Beads 都是靠它工作的。你描述工作流，Skill 把你的散文记录进 `docs/agents/issue-tracker.md`，下游 Skill 照着这段散文执行。已经有人把这条路走通过：一个基于 [MCP](https://www.aihero.dev/ai-coding-dictionary/mcp) 的 Jira 变体、一个仿照 `gh` 形状做的 Gitea CLI、一个手工搭的本地看板。

## 常见问题

**我必须用 GitHub 吗？**

不必。GitHub、GitLab 和 `.scratch/` 下的 local markdown 都作为现成模板提供，其他任何方式走 "other" 这条路也能用。这是被问得最多的一句话，措辞大致是：*"是不是死锁在 GitHub 上"*、*"我能用 GitLab / Jira 吗"*、*"Azure DevOps 怎么办"*。答案始终一样：setup 选的是 tracker，不是 Skill。

**更新 Skill 之后我需要重新运行它吗？**

v1.1 之后直接的回答是需要。Skill 自己的收尾消息更柔和，它说只有换 tracker 或推倒重来时才重新运行。两种说法都站得住。模板种子随版本会变，所以旧版本留下的 `docs/agents/issue-tracker.md`，可能和如今读取它的那些 Skill 脱节。如果某个下游 Skill 做的事和文档描述不同，就重新运行 setup。它开销很低。

**它写了 `CLAUDE.md`，可我在用 Codex。**

这是一个已知的缺口，而且仍未关闭。选文件的规则是"若 `CLAUDE.md` 存在就编辑它，否则编辑 `AGENTS.md`"。它检查的是哪个文件存在，不是哪个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（运行框架）在跑。在一个留着 Claude Code 时代 `CLAUDE.md` 的仓库里，Skill 会把它的 `## Agent skills` 块写进一个 Codex 从不读取的文件。用户有两个绕法：手动把那个块搬到 `AGENTS.md`，或者以 `AGENTS.md` 为准、把 `CLAUDE.md` 做成指向它的一行。若两个文件都不存在，Skill 会问你要创建哪一个，而不是替你选。这一点让期待它自动决定的人犯过糊涂。

**它没有创建我的 triage 标签。**

它本来就不创建。`docs/agents/triage-labels.md` 是一份*映射*：它告诉 `/triage` 你 tracker 里的哪些字符串对应那五个规范角色。它不会去跑 `gh label create`。在一个全新的 GitHub 仓库里这些标签还不存在，而这件事被用户报成 bug 不止一次。两个后果：

- 如果你的 tracker 已经在用规范名称，这份映射就是一张恒等表，没有东西需要配置。这是预期的常见情形，不是漏掉的步骤。
- 本 Skill 也不创建 [wayfinder](https://aihero.dev/skills-wayfinder) 需要的 `wayfinder:map` 和 `wayfinder:<type>` 标签，而 `gh issue create --label <missing>` 会直接失败，不会顺手建出标签。在 GitHub 仓库上第一次运行 wayfinder 之前，手动创建它们。

**我能在这里配置其他 Skill 的行为吗（[grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)（拷问）的节奏、问题格式、语气）？**

不能。它配置三件事：tracker、标签、文档布局。有用户希望把它变成存放个人偏好的地方。答复是 Skill 保持有主张，不接受任何用户级配置。偏好属于 `CLAUDE.md`，写成普通指令即可，而每个 Skill 本来就会读那里。

**我能把配置放在 `~/.claude`，而不是提交进每个仓库吗？**

目前不能。有一位在多个仓库间使用这套 Skill 的用户提了这个需求，但没有用户级模式。每个仓库都带着自己的一份 `docs/agents/`。

**有一个 Skill 去配置其他 Skill，这难道不奇怪吗？**

一个存在已久的抱怨说不奇怪才怪，原话是：*"用一个 Skill 去设置另一个 Skill，让我觉得不对劲：那意味着 LLM 在配置它自己的 Skill。"* 这个取舍是真实的。没有 setup 这一步，每个碰工单的 Skill 就得各自保存一份 tracker 说明。产出是你能读、能手改的 markdown，这一点限制了风险。它写下的每个文件你都能读、都能手工改动。日常修改就那么做，而不是再跑一次。

## 起作用的表现

- `docs/agents/issue-tracker.md` 和 `docs/agents/domain.md` 存在，若装了 `triage`，还有 `triage-labels.md`。
- 你的 harness 会读取的那个指令文件里出现了 `## Agent skills` 小节，其中每个文件都有一行摘要指向它。
- 它提议的 tracker 与你实际使用的 remote 相符，标签字符串与你 tracker 里已存在的标签相符。
- 之后，`/to-tickets` 发布时不再问你工单住在哪，`/triage` 贴的是标签而不是自己编的。
- Skill 文件本身没有任何改动。如果 setup 编辑了某个 `SKILL.md`，那一定是哪里错了。

## 它在整体中的位置

`setup-matt-pocock-skills` 是这套工程流程的**一次性设置**，是其他一切所依赖的前提条件，而不是链条上的一步。它的邻居就是它的读取者：[triage](https://aihero.dev/skills-triage)，在此写下的标签词汇表由它应用；[to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets)，把内容发布到这里点名的 tracker；以及 [wayfinder](https://aihero.dev/skills-wayfinder)，它读取同一份 tracker 文件的 "Wayfinding operations" 小节，从中了解如何存放地图和子 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）。[domain-modeling](https://aihero.dev/skills-domain-modeling) 随后补全 setup 记录下来的领域文档布局。它只在你敲定一个术语或一项决策时才创建 `GLOSSARY.md` 和 ADR，所以 setup 跑完之后仓库还没有领域文档是正常状态。下一步该用哪个 Skill，[ask-matt](https://aihero.dev/skills-ask-matt) 在整套里为你指路。
