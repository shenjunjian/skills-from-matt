## 它做什么

`implement-spec` 拿来一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明）和它的 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单），在一次运行里把整件事落地。负责编排的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（代理）把每张工单交给一个实现用的 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent)（子代理），每个 subagent 在自己独立的 git worktree 里工作；它把每条完成的分支合进一条**集成分支**，在合并结果上跑 [code-review](https://aihero.dev/skills-code-review)，然后把工单逐一结掉。

它把工单当成一张**任务图**来读，不是一份清单。阻塞边决定什么可以开始，所以任何时刻都有一批**前沿**工单，它们的阻塞项都已经落地，前沿上的每张工单同时开跑。这就是它和一张张工单依次做的区别。决定节奏的是图的形状，不是工单在跟踪器上的排列顺序。

## 什么时候找它

要输入 `/implement-spec` 才能调用它，agent 不会自己去找它。

| 你的情况 | 该找谁 |
| --- | --- |
| 一份 spec，已拆成带阻塞边的工单，你想在一次运行里全部落地 | `/implement-spec` |
| 一次只做一张工单，在你自己的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)（上下文窗口）里，工单之间做 [clearing](https://www.aihero.dev/ai-coding-dictionary/clearing)（清空） | [implement](https://aihero.dev/skills-implement) |
| 一份 spec，还没拆成工单 | 先走 [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一小块活儿，谈不上什么图 | 直接用 [implement](https://aihero.dev/skills-implement) |

## 前置条件

- **一个工单跟踪器。** 这个 Skill 从 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 配置好的跟踪器上读工单，也在上面结掉工单。如果一个都没配置，它会停下来叫你先去跑那一步，而不是自己猜。
- **带阻塞边的工单**，也就是 [to-tickets](https://aihero.dev/skills-to-tickets) 写出来的那种。没有边的话图是平的，每张工单同时开跑。
- **一个能在后台跑 subagent、并给每个 subagent 一个 git worktree 的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)**（运行环境）。这个 Skill 存在的意义就是同时跑多张工单，所以在一次只能跑一个 subagent 的 harness 上，它只是一个更慢的 `implement`。

## 集成分支

一切都落在同一条分支上。每个实现者：

1. 开工前先确认自己的 worktree 基于集成分支，
2. 用 [tdd](https://aihero.dev/skills-tdd) 构建它的工单，一片一片地 red-green（红-绿），
3. 汇报完成之前先把集成分支的最新提交合进自己的分支，这样合回去就是一次 fast-forward。

要不要有 pull request，由跟踪器说了算。如果你的跟踪器靠 PR 结活儿，或者你主动要一个，这个 Skill 会在第一次合并之后开一个标记为草稿的 PR，并在结束时标记为可评审。否则这次运行就停在那条集成分支上，每张工单按你的跟踪器结活儿的方式结掉；对着本地 markdown 跟踪器也能完全离线跑通。

实现者通过 [context pointers](https://www.aihero.dev/ai-coding-dictionary/context-pointer)（上下文指针：spec、工单、共享的探索笔记、早先的提交）跟编排者沟通，而不是粘贴摘要。这让每个 subagent 的提示词保持小，也给编排者的上下文窗口留出装下整张图的余量。

## 常见问题

**跟我自己挨张工单跑 `/implement` 有什么区别？**

这就是这个 Skill 存在的意义。在它发布之前，人们一直在自己搭类似的版本，有一位用户把需求说得很清楚：他想要"subagents 来实现工单"，而不是"新建一个 session、一次一张地叫它们去做工单，毕竟一份 spec 可能有五张以上的工单"。用 `implement` 时，调度员是你：一张工单一个 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话），中间清空上下文，哪几张工单解除了阻塞也由你自己盯着。`implement-spec` 把这活儿交给一个编排 session。代价是你不再能边落地边逐张读每张工单的工作内容，你是在最后评审那条集成分支。要起一次运行，先清空上下文，然后输入 `/implement-spec`，并给一个指向 spec 的引用（一个 issue 号或一个文件路径）。改动小、没有真正的图，就别用它，直接用 `implement`。

**它需要 GitHub 吗？我想让它停在分支上。**

不需要了。有一位用户很喜欢开发中的那个版本，他的抱怨正是这一点："它最后会建一个 PR，这就得有一个像 GitHub 这样的在线仓库。我希望它能在离线状态下做同样的活，停在所有工作都合并好的那条分支上。"现在运行结束在集成分支上。只有当配置的跟踪器靠 PR 结活儿，或者你主动要一个，才会开 PR。所以在本地 markdown 跟踪器上，运行结束时每张工单都已结掉，工作都合并在那条分支上。

**它的评审与修复循环跑了好几个小时，或者一直去"修"那些还没构建的工单。**

只要 `code-review` 跑在这个 Skill 给它的唯一那个时点之外，这两种情况都会发生。它拿代码跟整份 spec 比对，所以只有在每张工单都落地之后它才有意义。如果在运行中途跑，每张还没构建的工单都会被读成失败。于是 agent 就去构建那张工单，而这又触发一次评审。按设计，这个 Skill 在结束时跑一次 `code-review`，把每条发现交给一个修复用的 subagent，但它还没说这次修复之后什么时候该停。有一位用户报告一个五张工单的功能"评审与修复循环花了大约四个小时"。如果你看到第二次大范围评审起步，就叫它对已修复的那几条发现跑定向检查，然后停下。第一次评审会找到真问题，这是预期。这次运行的产出是一份草稿，靠评审补完，不是能直接发布的东西。

**它像 implement 那样驱动 tdd 吗？**

现在会了，起初不会。用过开发版本的用户注意到"实现者的 subagents 不继承 /tdd 指令"，所以他们从一张工单扩到整份 spec 的那一刻，red-green 就停了。现在每个实现者都用 `tdd` 来构建自己的工单。仍然没有一步让你交互式地商定 seam（接缝），而 `implement` 的 session 里有；想让 seam 钉死，就在 spec 或工单里把 seam 写出来。

**两个并行跑的实现者在同一个文件上撞了，或者给同一个东西取了不同的名字。**

worktree 不消除冲突，只把冲突推迟到合并的时候。根据工单文字写出的阻塞边，是在猜每张工单会碰哪些文件，而两张号称"代码库不同部位"的工单仍然共用一份消息目录、一个配置注册表或一个类型。每个实现者只看得到自己的工单和共享笔记，看不到另一个正在进行的工作，于是一位用户的 web 和 mobile 两张工单把同一个字符串分别写成了 `blockedSince` 和 `blockedOn`。当前沿上两张工单碰同一个共享文件时，要么在它们之间加一条阻塞边让它们依次跑，要么让探索笔记把每张工单要加的确切名字钉死。

**被阻塞的工单从来不开始，即使阻塞它的那张已经合并了。**

这是 GitHub 上的已知问题。跟踪器的 blocked-by 计数只在阻塞项*关闭*时才下降，而工单通常在 PR 合并时关闭，那已经是运行的末尾。跟踪器作为起步时那张图的来源是对的，在运行中途它是过期的。叫编排者自己跟踪哪些工单已经合进了集成分支，并从这份记录算出前沿。

**它能替代 Sandcastle 或者一个 AFK 脚本吗？**

不能。人们这么问是因为现在这些 Skill 已经伸进实现环节了："Sandcastle 还有意义吗？你们的 Skill 现在看起来也能处理实现了。" `implement-spec` 是把一个 agent 放到一次 harness session 内部的编排位上，这不需要任何基础设施，而且你可以看着、可以掌方向。真正 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk)（人离开工位）的工作，用一个确定性循环（[Sandcastle](https://github.com/mattpocock/sandcastle)、一个 shell 脚本、一个 CI 任务）更快、更便宜、更可靠，因为没有 agent 在做编排决策。

**某张工单的关键测试在它的 worktree 里被跳过了，而它报告说是绿的。**

worktree 里只有 git 跟踪的东西。读被 gitignore 排除的测试夹具、本地数据库或凭据的测试，在那里会悄悄跳过。对于验证依赖未跟踪内容的工单，叫编排者改在主检出目录里跑它。

## 有效的话应该是这样

- 只要图允许，就有几个实现者同时在跑，不是一个接一个。
- 一张工单在它的最后一个阻塞项落到集成分支的那一刻就开跑，而不是等整次运行结束。
- 每张工单的 trace 里都能看到 `tdd` 在跑，代码之前有一个失败的测试。
- 合进集成分支都是 fast-forward，不是解决冲突。
- 运行结束在一条分支上，每张工单都已结掉，只有你的跟踪器要 PR 时才有 PR。

## 它在整体中的位置

`implement-spec` 是主链里的构建环节，是每张工单跑一次 [implement](https://aihero.dev/skills-implement) 之外的并行替代：

```txt
grill-with-docs → to-spec → to-tickets → implement-spec → retro
```

它的邻居是 [to-tickets](https://aihero.dev/skills-to-tickets)，由它声明这个 Skill 当作任务图来读的那些阻塞边；以及 [code-review](https://aihero.dev/skills-code-review)，收尾前它会在集成分支上跑一次。拿不准自己走的是哪条 flow 时，[ask-matt](https://aihero.dev/skills-ask-matt) 是罩在整个集合之上的路由。
