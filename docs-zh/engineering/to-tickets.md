## 它做什么

`to-tickets` 拿一份计划、一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明），或者你正在进行的这场对话，把它拆成你的工单跟踪器上的若干 **[tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)**（工单）。每张工单声明自己的**阻塞边**：必须先完成、它才能开工的其他工单。

每张工单都是一颗 **tracer bullet**（曳光弹）：一条穿过这次改动每一层（schema、API、UI、测试）的窄而完整的路径，落地之后单独就能演示。这就是这个 Skill 与那种显而易见的拆法之间的区别，后者一层一层切，最后才集成。它还会把每张工单定到能塞进一个全新的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)（上下文窗口）的大小，因为接过这张工单的 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）从没见过你的 spec。

## 什么时候找它

要输入 `/to-tickets` 才能调用它。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己去找它。

| 你的位置 | 该跑什么 |
| --- | --- |
| 你有一个 spec 工单，构建跨好几场 session | `/to-tickets`，或者 `/to-tickets #<spec_issue>` |
| 计划只在对话里，从没写下来 | `/to-tickets` 直接读这段对话，不需要 spec |
| 整个改动塞得进一个上下文窗口 | [implement](https://aihero.dev/skills-implement)，跳过工单 |
| 还什么都没定 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，然后 [to-spec](https://aihero.dev/skills-to-spec) |
| 一张 [wayfinder](https://aihero.dev/skills-wayfinder) 地图已经走完 | 先走 [to-spec](https://aihero.dev/skills-to-spec) 把地图收拢，再 `/to-tickets` |

`to-tickets` 产出的工单天生就是 agent-ready 的。别拿 [triage](https://aihero.dev/skills-triage) 去过它们。triage 处理的是别人送来的工作。

## 前置条件

`to-tickets` 要往跟踪器里发布内容，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须已经为这个仓库配好一个跟踪器，以及 triage 标签词汇表。两种都行：GitHub 或 Linear 这样的真实跟踪器，或者 `.scratch/` 下的本地 markdown 文件，后者不需要任何额外配置。

## Tracer bullet，不是按层切

**水平**切片交付这次改动的一层。在所有层都落地之前什么都跑不起来，于是每张工单的验收条件都得伸进另一张工单负责的工作里去。**垂直**切片（也就是 tracer bullet）一次穿过所有层，铺出一条薄路，所以它单独就能验证，也把自己要评分的东西全都握在手里。

这是最常被违反的那条规则。有个团队按层切了一个 26 张工单的堆（corpus、producer、aggregator、selector），每关掉一张工单大概要跑二十轮 agent，其中四分之三左右是返工。他们自己的复盘把每一类失败都追回到水平切分上，而不是追回到具体实现上。

在任何东西发布之前有两件事会发生。`to-tickets` 会找 prefactoring（那条原则："先让改动变容易，再做那个容易的改动"），并把这部分工作排在最前。然后它把拆分结果作为一份编号列表给你看，并就此考你：粒度对不对，阻塞边是不是真的，有没有什么该合并或该拆开。在你批准之前，跟踪器上什么都不会落地，而这场考问就是提出异议的地方。

## 阻塞边

这些边才是这件产物的意义所在。它们按跟踪器不同而有两种起作用的方式：

| 跟踪器 | 边存在哪里 | 你怎么利用它们 |
| --- | --- | --- |
| 本地 markdown | `.scratch/<feature>/issues/<NN>-<slug>.md` 下每张工单一个文件里的文字，按阻塞者优先编号 | 从上到下，手动 |
| 真实跟踪器（GitHub、Linear） | 原生阻塞链接，或者跟踪器支持的话用子工单 | 阻塞项全部完成的工单就在**前沿**上，可以被拿走 |

不管哪种情况，边都存在工单里。跟踪器只决定有没有什么东西能并行地据它们行动。`to-tickets` 产出这件产物；怎么跑它（一次一场 session，还是一整队）是你的事，不是这个 Skill 的事。

## 大范围重构这个例外

有一种形状会打破 tracer bullet 规则。**大范围重构**是一处机械改动（重命名一个列、给一个共享符号换类型），它的**波及范围**覆盖整个代码库。一处编辑打坏上千个调用点，所以任何垂直切片都没法以绿色落地。

`to-tickets` 会把它改排成 **expand-contract**（先扩张、后收缩）：

- **Expand**：把新形式加在旧形式旁边，什么都不破坏。
- **Migrate**：按波及范围分批把调用点迁过去（按包、按目录），每批一张工单，每张都被 expand 阻塞。CI 保持绿色，因为旧形式还在。
- **Contract**：没有一个调用者剩下时，删掉旧形式，这张工单被每一个 migrate 批次阻塞。

连分批都没法单独保持绿色的地方，它们共用一条集成分支，并且全部阻塞最后一张 integrate-and-verify 工单。CI 只需要在那张工单上是绿的。

## 常见问题

**一个三行的改动它给了十二张工单。**
过度拆分是这个 Skill 被报告最多的问题，很多用户都碰到过。[model](https://www.aihero.dev/ai-coding-dictionary/model)（模型）默认输出原子单位，把能让它们有意义的那个分组丢掉了。考问那一步就是你来修这件事的地方。让它合并工单，它会合。也有一条下界。整个改动塞得进一个上下文窗口，你压根不需要这个 Skill。直接去 [implement](https://aihero.dev/skills-implement)。

**工单按层出来了：schema 全在一张，API 全在另一张。**
这正是垂直切片这条规则要防的那种失败，而 Skill 有时还是会产出它。在考问那一步用一个针对每张工单的问题抓出来：这张做完我能演示什么？答不上来的工单就是水平切片。有些人因此给每张工单加一行 "demo path"，并报告说这会把模型推向垂直切片。

**在 GitHub 上，工单没有被建成 spec 工单的子工单。**
这在上十次运行、好几个模型上都被报告过，[最完整的是 issue #554](https://github.com/mattpocock/skills/issues/554)。已修：`to-tickets` 现在把每张工单做成它那个来源工单的子工单，GitHub 跟踪器模板也给出了命令（`gh issue create --parent <n>`，`gh` 2.94+）。重跑一次 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills)，把旧的 `docs/agents/issue-tracker.md` 刷新。

**"Blocked by" 被写进工单正文，而不是真正的阻塞链接。**
同一类问题，[在 issue #513 里被报告](https://github.com/mattpocock/skills/issues/513)，那里 agent 甚至声称 GitHub 根本没有原生阻塞关系。有的：`gh issue create --blocked-by 12,15`。因为 Skill 先发布阻塞项，创建时它们的编号总是可用。正文那种写法是给没有原生边的跟踪器准备的退路，不是默认做法。工单模板现在在边是原生的时候会省掉 `## Blocked by`。

**本地工单放在哪？v1.1 的说明写的是根目录下的 `tickets.md`。**
确实这么写过，那是个 bug。一个共享文件还会在并行 agent 同时写入时造成竞态。本地模式现在在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下每张工单写一个文件，按依赖顺序，跟本地跟踪器模板本来描述的那套布局一致。`NN` 前缀是真正的工单 ID，所以 `/implement 03` 能用，不用再重打一遍长标题。

**它读我的 spec 时一直在截断。**
一份很大的 spec 可能超出跟踪器工单能干净返回的量。没有本地副本可退，于是 agent 把 [tool calls](https://www.aihero.dev/ai-coding-dictionary/tool-call)（工具调用）花在反复取分块上，永远读不到结尾。别在 `/to-spec` 和 `/to-tickets` 之间 [clear](https://www.aihero.dev/ai-coding-dictionary/clearing)（清空）或 [compact](https://www.aihero.dev/ai-coding-dictionary/compaction)（压缩）。把它们放在同一个上下文窗口里跑，agent 就不用再把 spec 取回来。

**验收条件什么都没评：有些在任何活干之前就已经通过了。**
模板要求写条件，但对它们能不能失败一字未提，所以这事会发生。三种形状反复出现：一个在基线提交上已经成立的条件，一个只有别的工单里的工作才能满足的条件，还有一个只是复述需求、没有从产物里推出来。垂直切片能挡住大部分，因为交付新行为的切片按构造在基线提交上就是失败的。这件事仍然值得手工查一遍。对每个条件，说出哪个观察能证明它为假，并确认它在实现者起步的那个提交上是假的。

**工单发布完了。我到底怎么把它们跑起来？**
Skill 停在产物这一步，没有自动派发模式。派发是手工的：看看板，数一数没有未完成阻塞项的工单有几张，就开那么多场 agent session。给每张工单一个全新的上下文，彼此之间清空。[implement](https://aihero.dev/skills-implement) 结束时不会可靠地关闭或勾掉工单，无论在 GitHub 还是本地 markdown 上，所以工单状态要你自己更新。

## 有效的话应该是这样

- 每张工单对"这张做完我能演示什么？"都有答案，答案是行为，不是一层。
- 在任何东西发布之前，那份列表带编号回到你手上，每张上面有一行 "Blocked by"。
- 排在最上面那张工单没有阻塞项，可以立刻开工。
- 工单正文里没有文件路径也没有行号，只有 prototype 产出的代码片段除外。
- 每张工单读起来都像是一场全新的 session 在你不在场的情况下能干完的东西。
- prefactoring，只要它找到了，排在顺序最前面，而不是混在功能工单里。

## 它在整体中的位置

`to-tickets` 是主构建链上的一步：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

上游是 [to-spec](https://aihero.dev/skills-to-spec)，交给它一份已经定下来的 spec 用来切。把两者放在同一个上下文窗口里，中间不要 `/clear`。下游是 [implement](https://aihero.dev/skills-implement)，每场全新的 session 构建一张工单，靠 [tdd](https://aihero.dev/skills-tdd) 驱动测试，收尾用 [code-review](https://aihero.dev/skills-code-review)。[implement-spec](https://aihero.dev/skills-implement-spec) 是另一条下去的路。它把同一批阻塞边当作任务图来读，在一条集成分支上并行构建所有就绪的工单。拿不准哪个 Skill 或哪条 flow 合适的时候，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
