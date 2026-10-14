## 它做什么

`retro` 回顾一次编码 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话），并对 agent 的**[环境](https://www.aihero.dev/ai-coding-dictionary/environment)**提出改进建议，好让下一次运行更顺。它读取会话自身的记录（默认是当前那次，也可以是你在会话日志里指定的某一次）。它找出 agent 吃力的时刻，给你一份候选修复清单，最严重的排在前面。

它改动的是环境，不是代码。拿 agent 交付的那个 bug、它花了二十次[工具调用](https://www.aihero.dev/ai-coding-dictionary/tool-call)才找到的那个文件、或者 reviewer 漏掉的那条规则来说：`retro` 不直接修它们中的任何一个。它问的是仓库里有什么让这些事得以发生，并提出下次能阻止它们的那项检查、那个指针或那条标准。而且它只提建议。你不选定某个候选项，任何东西都不改动。

## 什么时候用它

你通过输入 `/retro` 来调用它，agent 不会自己去找它。

在一次比本该更容易的会话结束时用它。举例来说：agent 为某样东西搜索了太久、犯了一个工具本可以拦住的错误、或者需要它拿不到的信息。顺利的会话教不了什么，发现都来自艰难的那些。如果你想对会话产出的代码要一个结论，改用 [code-review](https://aihero.dev/skills-code-review)。

## 发现落在哪里

每个候选项属于一个类别，而类别决定修复去处：

| 会话里出了什么错 | 用什么修 |
| --- | --- |
| agent 花了很长时间才找到一个文件或事实 | 从它已经在读的文件出发，加一个**导航指针** |
| 它犯了一个工具本可拦住的错误 | 一项**[自动化检查](https://www.aihero.dev/ai-coding-dictionary/automated-check)**：lint 规则、类型、测试、pre-commit hook、CI job |
| reviewer 漏掉一个需要判断力的错误 | `CODING_STANDARDS.md` 里给 reviewer agent 的一条规则 |
| `AGENTS.md` 或 `CLAUDE.md` 太大 | 把其中的引导内容搬出去，放进标准或检查 |
| 一次工具调用相对它的产出过于昂贵 | 精简这个工具，或者换掉它 |
| 某个引导文件里塞满不起作用的行 | 删掉这些 **no-op** |
| agent 需要它够不着的信息 | 扩大它的访问范围：把 dev server 日志同时写入文件，给出对某个服务的只读访问 |

核心主张是标准属于 **reviewer**，不属于实现者。实现 agent 承受的上下文压力最大，因为它要探索、写代码、调试失败。review 的 agent 只拿到一个 diff，别无其他。所以新规则要放在有余地执行它的地方，也就是 review。它绝不放进 [AGENTS.md](https://www.aihero.dev/ai-coding-dictionary/agents-md)，那会加载进每个会话的[上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)，不管相关与否。

在写下任何规则之前，`retro` 先给违规分类。**机械性**违规（被禁用的 API、某种 import 写法、某条文件位置规则）配一项确定性检查，因为检查能失败，而标准文件里的一句话不能。只有真正需要判断力的情形，也就是任何 linter 都无法强制的，才写成散文。如果仓库根本没有护栏（没有 pre-commit hook，也没有跑 lint、typecheck 和测试的 CI job），`retro` 会把这件事作为它自己的一条发现报出来。

## 常见问题

**它自己写 lint 规则，还是等一个"是"？我能把它接到每次会话之后自动跑吗？**

它等。`retro` 只提建议。你不选定候选项，任何东西都不改动，所以它不手动编辑任何文件，也不自动安装 hook。这是刻意的设计。有一位用户在"被那些挡掉好改动的自动 hook 伤过"之后，正是要求这一点。决定什么值得配一项永久检查需要判断力，所以这个 Skill 保持 [human-in-the-loop](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)（人在环中）并由用户调用。确实有用户在每次实现之后都跑它。但顺利的会话教不了什么，所以每次会话之后都跑，大多产出的是没人需要的规则。没有 dry-run 模式。一项被提议的检查就是普通代码，所以在让它去挡 merge 之前，先拿仓库试试它。

**这样不就无止境地堆 lint 规则吗？它会建议删掉某条吗？**

部分会，这是它最弱的一环。它能删散文：引导文件里的 no-op，以及 `AGENTS.md` 或 `CLAUDE.md` 中本该属于标准或某项检查的引导内容。文件过大时它把这些标记出来建议删除。它只拿自己读到的那一次会话来做判断，所以每一项都当成删除测试的候选，而不是定论。它不审计自己上个月提过的 lint 规则、hook 或 CI job。它只看一次会话，所以无法告诉你某条规则现在触发得太频繁，或者它背后那个 bug 已经消失。检查仍然得你自己修剪。一条规则在好代码上不断触发，就是该移除的信号。

**它会不会为了填满各类别而编造通用建议？**

这是对它最有力的批评。有一位用户发现"活儿干完之后，AI 倾向于忘记会话中途的挣扎，转而编造通用建议来凑满 retro 的类别。"每个候选项都必须出自会话自身的记录，所以建议是针对那一次会话的。这既有好处也有代价。它很少编造不相干的东西，但可能给这一次会话恰好涉及的事过分重的分量。任何你追不到具体时刻的候选项，丢掉。严重性排序也只当第一稿，因为一个安静而昂贵的错误，可能排在一次喧闹而廉价的错误之后。

**我的会话很长。现在跑，还是重开一次？**

默认它回顾当前会话。那是最好的情形，因为那些挣扎还在上下文窗口里。如果会话内容已经移出 [smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)（高效区），先[清理](https://www.aihero.dev/ai-coding-dictionary/clearing)它，然后让一次全新的 `/retro` 指向会话日志里的上一次会话。

**agent 一直在犯同一个错误。我该往 `CLAUDE.md` 里加一行吗？**

通常不该，而 `retro` 反驳这一点多于反驳其他任何事。`CLAUDE.md` 里的一行会加载进每个会话，削弱那文件里其他一切的权重，并且随代码改动而过期。如果错误是机械性的，修复是一项会失败的检查。如果是判断问题，它进 reviewer 阅读的那份编码标准。`AGENTS.md` 和 `CLAUDE.md` 用来放导航指针，别的不放。出于同样的理由，`retro` 不是一个[记忆系统](https://www.aihero.dev/ai-coding-dictionary/memory-system)。它不存储发生过什么。它改变环境，让那个错误无法再次发生。

**我的配置里提到 `CODING_STANDARDS.md`，可我没有这个文件。它从哪来？**

没有任何 Skill 随包提供这个文件。当某次会话第一次为 reviewer 找出一条需要判断的规则时，`retro` 会提议创建它。你接受之后，[code-review](https://aihero.dev/skills-code-review) 就会读它。你已经在维护的其他任何标准文档，比如 `CONTRIBUTING.md`，用法相同。

**它和 `improve-codebase-architecture` 有什么区别？**

输入不同。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 只需要代码，在其中寻找结构上的改进。`retro` 需要一份会话历史，改进的是 agent 工作所处的环境，不是代码。两个都用，谁也不替代谁。

## 起作用的表现

- 每个候选项都指回会话里的某个具体时刻，而不是一条通用最佳实践。
- 重复出现的错误变成会失败的检查，而你的 `AGENTS.md` 随时间变短，不是变长。
- 如果某项检查本来就已存在、只是没接上，那条发现是"把它接上"，而不是"再建一个"的提议。
- 下一次做同类任务的会话能更快找到它需要的东西。

## 它在整体中的位置

`retro` 是主链条的最后一步，在这里你回看这条链走得如何：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

在一次值得从中学习的构建之后运行它，可以在同一个会话里，也可以指向那次会话的日志。顺利的一次构建之后可以跳过它。

- [code-review](https://aihero.dev/skills-code-review) 是 `retro` 最常调校的 reviewer agent。新的编码标准写到它的 Standards 轴能读到的地方。
- [writing-for-agents](https://aihero.dev/skills-writing-for-agents) 为 `retro` 提议的每一份引导文件和 Skill 定下写作风格，`retro` 在开始前会加载它。

拿不准当前情形需要哪个 Skill 时，[ask-matt](https://aihero.dev/skills-ask-matt) 在整套之间为你指路。
