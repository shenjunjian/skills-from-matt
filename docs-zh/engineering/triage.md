## 它做什么

`triage` 逐个处理你项目跟踪器上的工单。它把每一个推过一个小的**分诊角色**状态机（一个类别角色加一个状态角色）。每个工单最后落到三种结果之一：一份 agent-ready 的简报、一个向报告人提的具体问题、或者一个带着记录在案的关闭理由被关闭的工单。

它只管**不是你创建的**工单。也就是原始的 bug 报告、陆续进来的功能请求、或者一份不请自来的外部 pull request：从外部进入跟踪器的工作，报告人留成什么形状就是什么形状。[to-tickets](https://aihero.dev/skills-to-tickets) 产出的 [工单](https://www.aihero.dev/ai-coding-dictionary/ticket) 天生就是 agent-ready 的，所以拿 `triage` 去过它们就是把这份工白费。规则很简单：`/triage` 只对外部进来的工单用，不对你自己创建的工单用。

它也跟手工打标签不一样，因为它先给建议然后等你表态。它把自己的类别判断和状态判断连理由一起给你，再加上它在代码库里查到的东西，在你叫它动手之前它什么都不改。

## 什么时候找它

要输入 `/triage` 才调用它，然后用平实的大白话描述你要什么。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己去找它。例子："给我看看有什么需要我处理的"、"我们看一下 #42"、"把 #42 挪到 ready-for-agent"。

| 你手上有什么 | 该去哪 |
| --- | --- |
| 一个装满了别人提交的原始报告的跟踪器 | `/triage` |
| 一个你自己的粗略想法，什么都没写 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 一场已经定下来的对话，要变成 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明） | [to-spec](https://aihero.dev/skills-to-spec) |
| 一份要切成 agent-ready 工单的 spec | [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一个确认存在的 bug，需要的是根因不是标签 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## 前置条件

`triage` 读写你的工单跟踪器，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须先配好那个跟踪器和它的标签词汇表。下面这些角色名是**规范的**。你跟踪器里的标签字符串可能不同，setup 提供映射。如果你的跟踪器本来就一字不差地用这些规范名，那你既没什么要映射，也没什么要配置。

跟踪器配置还决定外部 pull request 算不算一个需求入口，以及谁算外部。这个开关默认是关的，setup 也不再问这件事。要把 PR 纳入范围，就去 `docs/agents/issue-tracker.md` 里把它打开。

## 状态机

每个被分诊的条目最后都恰好带一个类别角色和一个状态角色。类别只有两种：`bug`（有东西坏了）和 `enhancement`（新功能或改进）。状态有五个：

| 状态 | 含义 |
| --- | --- |
| `needs-triage` | 你需要评估它。一个没打标签的工单通常先落在这里。 |
| `needs-info` | 在等报告人。他们回复之后回到 `needs-triage`。 |
| `ready-for-agent` | 完整规定好了，附着一份 agent 简报。[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)（人离开工位）agent 可以接走它。 |
| `ready-for-human` | 同样的简报，外加为什么 agent 干不了：需要判断、需要外部访问、需要手工测试。 |
| `wontfix` | 已关闭，理由记录在案。 |

词汇表就这些。每个条目只许有一个状态角色这条规则让查询保持简单。这些状态也是这个 [Skill](https://www.aihero.dev/ai-coding-dictionary/skill)（技能）被要求改动最多的地方。用户要求过：一个给"已经规定好但被另一个工单卡住"的工作用的状态，一个等待未来某个触发条件的工作用的 `deferred` 状态，一个终态 `implemented`。这些一个都没上线。见下面的问题。

`wontfix` 分三种情形。区别要紧，因为只有一种会往知识库里写东西：

| 你为什么关掉它 | 会发生什么 |
| --- | --- |
| 已经实现了 | 一条评论，指出这个功能已经在哪。`.out-of-scope/` 里什么都不放，因为它是已建成的功能，不是被拒绝的，放一个文件进去会破坏去重检查。 |
| 被拒绝的 bug | 客气的解释，然后关闭。 |
| 被拒绝的功能请求 | `.out-of-scope/` 里一个文件，从关闭评论里链过去，然后关闭。 |

`.out-of-scope/` 每个被拒绝的**概念**存一个 markdown 文件，不是每个工单一个。每个文件是一份短的设计文档，不是数据库里的一行。它写明被拒绝了什么、为什么，并列出所有提过这个要求的工单。`triage` 在评估任何东西之前先把整个目录读一遍。它按概念匹配，不按关键词匹配，所以 "night theme" 能匹配上 `dark-mode.md`。找到匹配时，它把旧决定给你看，问你现在是否还同意，而不是从头再争论一遍这个请求。

## 先验证，再写简报

在任何 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)（追问）之前，`triage` 先检查这个说法站不站得住。是 bug，它照报告人的步骤复现一遍。是 PR，它把分支签出来跑相关的测试。然后它报三种结果之一：

- 确认存在，附上代码路径。
- 复现失败。
- 细节不够，没法尝试。这是 `needs-info` 最强的一条信号。

同一趟里，它还对着代码库再跑两项检查。**冗余**检查问这个功能是不是已经实现了，并且按领域概念搜索，不按报告人的措辞。**先前拒绝**检查问 `.out-of-scope/` 是不是已经说过不了。两项检查都很便宜，任何一项命中就产出一个 `wontfix`。

所有这些活都是为了把一件产物做好：**agent 简报**。这是工单转到 `ready-for-agent` 时 `triage` 发的那条结构化评论。`triage` 发完简报，简报就是合同，原始报告只剩下背景。简报求的是**耐用**而不是精确，因为一个工单可能在 `ready-for-agent` 上待几个星期，其间代码一直在变。所以简报写类型、签名和行为约定，绝不写文件路径和行号。一次确认过的复现能让简报比任何猜测强得多。

## PR 就是带着代码的工单

如果跟踪器把外部 pull request 当作一个需求入口，它们走同一台状态机，同样的类别、状态和转移。这些状态作用在 diff 上。`ready-for-agent` 意味着简报已附上，agent 该对代码走下一步。`ready-for-human` 意味着人可以合并它。PR 上的简报描述的是这份已有 diff 还差什么，不是怎么从零把东西建出来。

发现阶段只列出*外部*的 PR，因为合作者正在做的分支不是分诊工作。这个过滤只作用于发现阶段。你要是明确点名某个 PR，`triage` 就会处理它，不管是谁写的。

## 常见问题

**我跑了 `/to-spec` 和 `/to-tickets`，那些工单现在没被分诊就摊在那里。我要拿 `/triage` 过一遍吗？**
不用。它们已经是 agent-ready 的。`to-tickets` 发布时就打了 `ready-for-agent` 标签，所以 AFK 执行器不用再过一遍就能把它们接走。碰到这事的用户走过 spec 流程，在输出上看到了 `needs-triage`，而他的 AFK 执行器把所有东西都忽略了。`triage` 是外部工作的上匝道，spec 流程是你自己启动的工作走的那条道。两条道在 `ready-for-agent` 汇合，不是在那之前。

**现在有了 `to-spec` → `to-tickets` → `implement` 这条流程，`triage` 还有意义吗？**
只有当有外部工作进来才有。`triage` 比那条流程更早，干的也是另一件事：它处理别人提上来的报告。如果你跟踪器里的一切都出自你自己的规划，你很少会用到它。只要你维护任何公开的东西，或者你的团队往你身上派 bug，这里就是那类工作的起点。主要用途是接收外部贡献者提工单的开源仓库。

**agent 想打 `ready-for-agent`，`gh` 说这个标签不存在。**
这是已知未修 bug（[#616](https://github.com/mattpocock/skills/issues/616)）。`setup-matt-pocock-skills` 把标签词汇表写进 `docs/agents/triage-labels.md`，但不在你的跟踪器里创建标签。用 `gh label create` 或跟踪器的界面，自己把五个状态标签和两个类别标签建一次，报错就没了。那个 issue 链接了一个社区修复分支，还没合并。

**五个状态不够。被阻塞的、推迟的、已经实现的怎么办？**
这是这个 Skill 上被提得最多的缺口。它来三种形态：

- 一个已经完整规定好、但要等另一个工单关闭的工单（[#139](https://github.com/mattpocock/skills/issues/139)）。提报人说这种情况下 `ready-for-agent` "技术上没错"但有误导性，于是 agent 把它接走然后卡住。
- 有意为之但等某个触发条件的未来工作，所以现在还不可执行（[#297](https://github.com/mattpocock/skills/issues/297)）。
- 一个"已实现，等待验证"的终态。没有它，AFK 执行器会把已经完成的工单再排一遍队。

被阻塞那种情形被认可为真实存在，但名字还没定（`blocked` 还是 `paused`）。这些都没上线。作为绕法，人们会在类别旁边加一个仓库自己的额外标签。这样状态那一格填的是准确的值，但 Skill 并不认识这个额外标签。有个社区分支走得更远，加了 `needs-slicing`、`tracking` 和工作量标签。这能用，但它属于那个分支，不属于这个 Skill。

**这跟 `/diagnosing-bugs` 有什么区别？**
这里的验证步骤是故意做浅的。它回答"这是真的吗，大概在哪"，不去找根因。如果一个 bug 照报告人的步骤几分钟内复现不出来，就用 `needs-info`；想现在就去查就用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。两个 Skill 的文本都还没提到对方。有用户报了这个缺口，还开着。

**我能把它指向我整个待办堆让它一路跑吗？**
你可以这么要求，但要留神它读了什么。"给我看需要我处理的"那一趟是一次便宜的列举，用来*挑选*。你挑一个工单，然后 `triage` 才去为那个工单收集完整的[上下文](https://www.aihero.dev/ai-coding-dictionary/context)。如果你一次跑二十个工单，agent 可能把这份便宜的列举当作全部证据，还不告诉你。列举返回工单正文，不返回评论。有个用户正好踩了这个。三个工单本来就有一条写着"已修复，建议关闭"的评论，结果三个都拿到了新的 agent 简报。要批量跑，就明确说它必须读每个工单的评论。

**它能配 Linear 用吗，或者 GitHub Issues 之外的东西？**
能。跟踪器是配置，不是硬编码的假设。人们拿它跑 Linear（通过 `linear` CLI）、GitLab，以及 `.scratch/` 下的纯 markdown 文件。一种常见的分工是 Linear 放工单和规划，GitHub 放代码和 PR。说"工单跟踪器"的 Skill 就映射到 Linear，说"PR"的映射到 GitHub。本地 markdown 跟踪器有一个模板 bug 还开着：生成的文件里验收条件可能出现两遍，一遍在顶层，一遍在 agent 简报里（[#200](https://github.com/mattpocock/skills/issues/200)）。

## 有效的话应该是这样

- 它经手的每个条目最后都恰好带一个类别角色和一个状态角色，不会是零个，也不会是两个互相冲突的状态。
- 它给你一个带理由的建议然后停下。它不会把工单重新打上标签就往下走。
- 在任何东西到 `ready-for-agent` 之前，它复现了 bug，或者签出并跑了那个 PR。
- 它写的简报点出类型和行为，里面没有文件路径也没有行号。
- 一个你半年前拒绝过的请求重新出现时，它告诉你并引用旧理由，而不是再分诊一遍。
- 它发的每条评论都以 `> *This was generated by AI during triage.*` 开头

## 它在整体中的位置

`triage` 是一个**上匝道**，不是主链上的一步。主流程从你自己的想法起步（grill、spec、工单、implement、review）。`triage` 是别人送来的工作并行走的那条道。两条道终点相同：一个打上 `ready-for-agent` 标签、上面有简报的工单。[implement](https://aihero.dev/skills-implement) 接它的方式和接 [to-tickets](https://aihero.dev/skills-to-tickets) 的工单一样。当一个请求需要更多细节、`triage` 才能为它写简报时，`triage` 会把 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 一起跑，一次一轮提问，这样它就在你做决定的同时把决定记进 `GLOSSARY.md` 和 ADR。拿不准自己走的是哪条道时，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
