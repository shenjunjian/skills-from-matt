## 它做什么

`to-spec` 把你刚进行的这场对话变成一份 **[spec](https://www.aihero.dev/ai-coding-dictionary/spec)**（规格说明），并把它作为单个工单发布到你的工单跟踪器上。

它不会对你做访谈。你找到它的时候，决定已经做完了。所以它把已知的东西（来自对话、代码库、你的 `GLOSSARY.md` 和 ADR）综合起来，不再开启新一轮提问。这份 spec 记录的是你已经做出的决定，不是拿来新做决定的地方。

## 什么时候找它

要输入 `/to-spec` 才能调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己去找它。

这桩构建大到一个 agent 的一场 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）装不下、必须拆到好几场里去做，这时候就找它。触发条件就这一条：

| 你的位置 | 该跑什么 |
| --- | --- |
| 还什么都没定 | 先走 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 已经定了，而且这桩工作塞得进一个 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)（上下文窗口） | [implement](https://aihero.dev/skills-implement)：跳过 spec |
| 已经定了，而且这桩工作跨好几场 session | `/to-spec`，然后 [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一张 [wayfinder](https://aihero.dev/skills-wayfinder) 地图已经走完 | `/to-spec #<map_issue>` |

## 前置条件

`to-spec` 会把 spec 作为工单发布，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须先为这个仓库配好跟踪器和 triage 标签词汇表。两种跟踪器都行：GitHub 这样的真实跟踪器，或者 `.scratch/` 下的本地 markdown 文件，后者不需要任何额外配置。

## 这份 spec 是一份决策记录

spec 之所以存在，是因为上下文窗口会结束。你在 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)（追问）的过程中定下了很多东西：方案长什么样、你反复争论过的那些选择、还有你故意拒绝掉的东西。这一切都在一场你马上就要清掉的对话里。spec 把它留下来。

所以 spec 不做验证，也不做任何决定。它用你项目自己的词汇记下你定下的东西，让一场全新的 session 能接手这份工作，不用再听你解释一遍。如果 spec 里写了某件你从没说过的事，那是缺陷。

## 先定 seam，再写文字

在写任何内容之前，`to-spec` 会先画出这个功能将在哪里被测试的 **seam**（接缝），并跟你一起确认。它宁可用已有的 seam 也不新建，并且挑它能挑到的最高那个 seam。一次改动理想的 seam 数量是一个。

别的 Skill 后面会用到这些谈定的 seam。[tdd](https://aihero.dev/skills-tdd) 只在你事先谈好的 seam 上工作。[code-review](https://aihero.dev/skills-code-review) 对照 spec 审查 diff，所以某个没人同意过的 seam 会作为一条审查意见冒出来。这两条连接都穿过这份文档。这就是为什么你要在这里认真谈 seam，而不是把这件事留给实现阶段。

## 常见问题

**`/to-prd` 去哪了？**
就是这个 Skill，在 v1.1 改了名。"Spec" 现在是全文统一用的那一个词，旧的 `to-prd` slug 已经不能用了，所以请按新名字重装。旧的词汇由 *spec* 和 *tickets* 这一对取代。spec 是目的地，也是把它固定下来的那些决定。[tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）是走到那里的步骤。要是你改了方向，删掉未完成的工单，留下 spec。

**spec 为什么被打上 `ready-for-agent` 标签？我不想让 agent 照着它去实现。**
这个标签的意思是"不需要再分诊"：这份文档已经完整到 agent 可以据它工作。它标记的是一个输入，不是一张工单。但那些轮询 `ready-for-agent` 的 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk)（人离开工位）agent 看不出这个区别。它们会试图在一轮里把整份 spec 全建出来，而不是一张张接过工单切片。这是这个 Skill 被报告最多的问题。在它改掉之前，在你的 AFK agent 的提示里明确排除父 spec，或者在 `/to-tickets` 跑完之后把标签去掉。

**为什么不从 grilling 直接到 `/to-tickets`，把 spec 跳过？**
很多时候你确实该这么做。只有在跨多场 session 的工作上，spec 才值它那一步。它的价值在于工单可以扔掉而 spec 不行。每张工单按一个全新的上下文窗口来定量，做完就被删除或关闭，而 spec 一直留在那里，作为记录它们背后推理的那一处。在单场 session 的改动上，这些都给不了你，反而多付一次综合步骤，而 [model](https://www.aihero.dev/ai-coding-dictionary/model)（模型）会在里面跑偏。从 grilling 直接去 `/implement`。

**我刚跑完一张 wayfinder 地图。我喂给它什么？**
给它主地图工单，`/to-spec #<map_issue>`，不是那一张张决策工单。[wayfinder](https://aihero.dev/skills-wayfinder) 产出的决定散落在整张地图上，不是可交付物。`to-spec` 把它们收拢成一份你可以照着构建的文档。要是你把地图直接接进 `/implement`，就丢掉了这一步。

**这份 spec 是给我审的，还是只给 agent 看的？**
主要给 agent，读起来也是这个样子：完整、密、满是引用。读 seam 那部分和范围外那部分。就在这两处，一个错的决定现在抓最便宜，以后发现最贵。确实有人抱怨要读全文，而它没有摘要模式。但如果 spec 让你意外，那是 grilling 太浅；不是 spec 太长。

**工单一旦开工，我把 spec 冻住，还是让 agent 改写它？**
没有任何东西让它保持同步。实际上它是你当时所知情况的快照，实现第一次教你新东西的时候它就开始过期。这桩活交付之后就把它当成可丢弃的。你的 `GLOSSARY.md` 和 ADR 才是打算长期留存的文件。要是你在实现中学到了该长留的东西，写进那里，别写进一份被改过的 spec。

**我的活是重构或者模块边界，不是新功能。模板合身吗？**
不太合身，这是已知局限。模板严重依赖用户故事，而用户故事套不进架构性的工作。你最后写出来的是一些没人要的故事，围着那些其实关于接口和不变式的决定。改用 implementation-decisions 和 testing-decisions 这两节。把长期有效的架构决定通过 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 记成 ADR，别记在 spec 里。

**它会去跟踪器上查相关的工作，或者引用它所遵循的 ADR 吗？**
两件都不会。它会读并遵循它触及那片领域的 ADR，但不做链接。它也不会在动笔前去跟踪器上搜重叠的工单，所以一份 spec 可能重复别人已经提过的工作，而且没有任何提示。这片领域要是很忙，你自己先去搜一遍跟踪器。

**`/to-tickets` 读不了我的 spec：一直在截断。**
一个跟踪器工单可能不会把一份很大的 spec 完整返回，而本地也没有副本可以替代。要解决这个问题，就别在 `/to-spec` 和 `/to-tickets` 之间 [clear](https://www.aihero.dev/ai-coding-dictionary/clearing)（清空）或 [compact](https://www.aihero.dev/ai-coding-dictionary/compaction)（压缩）。把它们放在同一个窗口里跑，`/to-tickets` 就再也不用去取 spec。

## 有效的话应该是这样

- 它直接动笔写，而不是问你新一轮问题。
- 它在写之前把 seam 给你看，并且尽可能少地提。
- 它用的是你项目的名词，不是通用产品管理的那套套话。
- 你记得里面每个决定都是自己做的。它没有为了填某一节而凭空造东西。
- 范围外那一节列的是真实的东西。你拒绝掉的那些，通常是整页上最有用的几行。

## 它在整体中的位置

`to-spec` 是主构建链上的一步，但只在其中的跨多场 session 那条分支上：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

上游，[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 做出决定，这个 Skill 只负责记录；一张走完的 [wayfinder](https://aihero.dev/skills-wayfinder) 地图在这里接入链条。下游，[to-tickets](https://aihero.dev/skills-to-tickets) 把 spec 切成 tracer-bullet（曳光弹）工单，交给 [implement](https://aihero.dev/skills-implement) 去构建。拿不准哪个 Skill 或哪条 flow 合适的时候，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
