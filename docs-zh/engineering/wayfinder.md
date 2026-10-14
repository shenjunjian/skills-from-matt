## 它做什么

`wayfinder` 接手的是一桩大到装不进一场 agent [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）的工作。这个想法的**目的地**你说得出来，但通往它的路还看不见。wayfinder 把它绘制成你的工单跟踪器上一张共享的**地图**，由若干 **Decision ticket**（决策工单）组成，然后一张张把它们解决掉，直到路径清晰。

它只做规划，不做构建。每张工单问一个问题，答案是一个决定，不是构建的一块。当任何东西都不需要在有人动手构建之前再被决定时，这张地图就算走完了。这条规则把一张 wayfinder 工单同普通的实现 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）区分开，也是 agent 最常违反的规则。地图走完时，wayfinder 交棒出去，不继续往代码里走。

## 什么时候找它

要输入 `/wayfinder` 才能调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己去找它。

它是整套 Skill 里最重的一条流程，所以触发条件很窄。这桩工作必须大过一场 agent session 能装的量，而且通往目的地的路必须还不清楚。分界是 session 的数量：单场 session 的规划用 `/grill-with-docs`，跨多场 session 的规划用 `/wayfinder`。

| 你面前是什么 | 该跑什么 |
| --- | --- |
| 一个范围清楚、一次坐下去就能定下来的功能 | [grill-me](https://aihero.dev/skills-grill-me)，有代码库时用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 一个从零起步的项目，或者跨很多场 session 的构建，路还看不清 | `/wayfinder` |
| 一场决定已经做完的对话 | [to-spec](https://aihero.dev/skills-to-spec)：直接跳过地图 |
| 一张走完的 wayfinder 地图 | [to-spec](https://aihero.dev/skills-to-spec)，然后 [to-tickets](https://aihero.dev/skills-to-tickets) 和 [implement](https://aihero.dev/skills-implement) |
| 一场已经长得太大的现存 session | 说"交接给 `/wayfinder`"（[handoff](https://aihero.dev/skills-handoff) 既能进一张地图，也能从一张地图出来） |

从零起步不是必要条件。人们在遗留代码和半成品代码库上照样用 wayfinder，在那里可能更有用，因为一大片迷雾是"这里已经有什么是事实"，而不是"我们该做什么"。

## 前置条件

地图和它的工单都住在仓库的工单跟踪器上，所以 wayfinder 需要 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 提供的跟踪器配置。那一步会写一个 "Wayfinding operations" 一节，说明怎么在 GitHub、GitLab 或本地 markdown 上表达这张地图、它的子工单、阻塞边和前沿查询。Wayfinder 通过你 `CLAUDE.md` / `AGENTS.md` 里的那个指针找到这份文档，不是在固定路径上找。没有配置跟踪器时，它退回到本地 markdown 文件。

跟踪器干的是真活。它的原生阻塞链接在跟踪器自己的界面里就把前沿显示出来。在一个没有原生依赖链接的跟踪器上（比如自托管的 Gitea），wayfinder 从地图文字里推断阻塞关系。这能用，但你得盯得更紧。

## 地图、迷雾和前沿

**地图**是一个打上 `wayfinder:map` 标签的单一工单，它的工单是这个工单的子工单。它是**一份索引，不是存储**。一个决定只活在一个地方，就是它的那张工单，而地图只给一行摘要加一个链接。一场 session 以低分辨率加载地图，需要时才打开具体工单。这让一张地图可以一直长，而不用每场 session 都把全部历史读进来。

地图有四节：

- **Destination（目的地）。** 这张地图走完是什么样子。你最先说出它，在任何一张工单存在之前，因为目的地固定了你用来衡量每张工单的范围。
- **Decisions so far（目前已定）。** 每个已关闭的工单占一行，每行都带一个链接，指出细节在哪。
- **Not yet specified（尚无法规定）。** 这就是**战争迷雾**：你能看出会来、但现在还说不精确的那些决定。区分迷雾和工单的测试，是你*现在*能不能把问题精确地说出来，不是你能不能回答它。你解决一张工单，它前面的迷雾就散开，任何你现在能规定清楚的东西就毕业成一张新工单。
- **Out of scope（范围之外）。** 目的地之外的工作。迷雾只朝目的地*方向*聚集，所以范围外的工作保持关闭，永远不会毕业。

**前沿**是那些未完成、未被阻塞、未被认领的工单组成的集合（已知世界的边缘）。一场 session 在动手做任何事之前，先把工单指派给自己来认领它。受指派人*就是*认领本身，所以并发的 session 会跳过那张工单。Session 用名字指代工单，绝不光用 `#42`，因为一片工单编号在叙述里很难读。

## 四种决策工单类型

每张工单带一个 `wayfinder:<type>` 标签。每张工单要么是 **[HITL](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)**（由一个为自己发言的人来一起做），要么是 **[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)**（完全由 agent 驱动）。HITL 工单只能通过实时的往来对话来解决。一个自问自答它的 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)（追问）问题的 agent，就是把它破坏了。

| 类型 | 模式 | 什么时候找它 | 由什么解决 |
| --- | --- | --- | --- |
| `grilling` | HITL | 默认。这个问题能靠谈出来定。 | [grilling](https://aihero.dev/skills-grilling) 加上 [domain-modeling](https://aihero.dev/skills-domain-modeling)，在一场全新的 session 里 |
| `prototype` | HITL | "这里该长什么样"或"这里该有什么行为"：谈不出结果的问题。 | [prototype](https://aihero.dev/skills-prototype)，从工单到那个建出来的东西留一条链接 |
| `research` | AFK | 工作目录之外的一条事实卡住了一个决定。 | 一个 [research](https://aihero.dev/skills-research) [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent)（子代理），在你绘制地图时启动，在 `research/<name>` 分支上并行跑 |
| `task` | 两者皆可 | 没有要决定的东西，但有些手工活卡着一个决定，比如开通访问权限、注册一个服务、或者搬一些数据好让你看清它的形状。 | agent 能自己做的就自己做，否则给人一份精确的清单 |

`task` 是唯一一种*动手做*而不是做决定的类型。它属于地图只因为它解开一个决定的阻塞，绝不因为它交付了目的地的一部分。实践中出错最多的就是这个类型。agent 把它读成一个实现步骤，然后开始在地图里写产品代码。

research 是*一场 session 一张工单*唯一的例外。

## 常见问题

**这跟 `/grill-with-docs` 有什么区别？我该从哪个起步？**
是 session 数量，不是项目规模。`/grill-with-docs` 是单场 session 的规划，wayfinder 是跨多场 session 的规划。如果整件事你能装进一场对话里，grilling 更便宜也更好，而 wayfinder 在这种情况下更慢更密。社区里的简说是：工作装不进单场 session，wayfinder 才讲得通。这远是被问得最多的 wayfinder 问题。人们不停地问，因为 Skill 的描述没告诉你你自己的任务在那条线上的哪一处。session 数量得你自己判断。

**它要的"目的地"，是指这场 session 的结尾，还是整件事的结尾？**
整张地图的结尾，不只是第一场 session。这个问题的措辞有歧义，但 wayfinder 按定义就是个跨多场 session 的工具，所以一场 session 范围的回答永远讲不通。典型的目的地是：一份要交出去的 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明）、一个在规划开始前锁定的决定、一个概念验证，或者一处就地改动，比如一次数据迁移。

**地图已经走完了。wayfinder 不是已经写好 spec、建好工单了吗？我为什么还要 `/to-spec` 和 `/to-tickets`？**
不是。Wayfinder 的工单是决策工单，到地图关闭的时候它们也全都关闭了。剩下的是满满一张带链接的决定的地图，那不是一份构建计划。[to-spec](https://aihero.dev/skills-to-spec) 把那些带链接的决定收拢成一份 spec（`/to-spec #<map_issue>`），[to-tickets](https://aihero.dev/skills-to-tickets) 再把那份 spec 切成 tracer-bullet（曳光弹）实现工单。要是你把地图直接接进 [implement](https://aihero.dev/skills-implement)，你就跳过了收拢，丢掉了那些带链接的细节。只有当这桩工作原来很小，才直接进实现。有些人跑更短的这条链，报告说管用。多出来的两步给你的是一份能让审查者或同事读出来的明确 spec，当你不是独自工作时，这件事更要紧。

**我的 agent 在一次 wayfinder session 中途开始写生产代码。**
这是这个 Skill 被报告最多的失败，而 Skill 里一个真实的缺口导致了它。你可以在地图的 **Notes** 里覆盖 wayfinder "只规划、不动手"的默认设置。但 Notes 是 agent 写的，于是这条约束和它的豁免都住在被约束的那个 agent 自己拥有的文件里。有个用户看着一个 agent 往自己的 Notes 里写了"这张地图带有执行"。在后面的 session 里，agent 把那一行读成许可，在一台在线服务器上开始构建。Skill 没有针对"我要的是默认设置"的硬停止。在有之前，任何一张不是你自己绘制的地图，先去读它的 Notes，把实现留在单独的 session 里，并把任何看起来像构建切片的 `wayfinder:task` 当作类型标错了。

**我绘了 27 张工单，画到第十三张，剩下的就说不通了。**
这个问题是一个用户报告的原文，别人也报告同样的结果。默认情况下 wayfinder 做详尽的规划。当后面的工单依赖的假设被前面的工单推翻时，地图就掉进批评者指控这个 Skill 的那种瀑布陷阱。两件事有帮助。第一，把地图的范围定在一个有界的目的地，不是整个产品。用户报告说，范围定在一个明确 epic 的地图比一张 sprawling 的 "implement V1" 表现更好。目标是小步交付，不是规划一件极大的东西。第二，[prototype](https://www.aihero.dev/ai-coding-dictionary/prototyping)（做原型）要用得激进。路径能保持最新，是因为便宜的具体产物会在实现依赖某个不确定之处之前先把它暴露出来。Wayfinder 是"原型最大化"，不是"规划最大化"。

**我能并行做几张工单吗？**
前沿告诉你哪些工单可以拿，阻塞边让并行工作在纸面上是安全的。实际上，一次一张是更稳妥的默认。如果你同时做两张 grilling 工单，一场 session 可能问你一个你刚在另一场里答过的问题，因为两场 session 之间不共享任何[上下文](https://www.aihero.dev/ai-coding-dictionary/context)。prototype 工单也有一个已知的缺口。一个用户报告说，一个 agent 建了三个 UI 变体，自己选了一个，然后把工单关闭了。这个选择是你的，而 Skill 还没把这点说得足够清楚。你要是真的并行跑工单，自己先把依赖图审一遍。

**我必须用 GitHub Issues 吗？**
不用。任何工单跟踪器都行。GitHub 支持最好，因为它的原生子工单和阻塞关系让前沿不用打开地图就能看见。人们也用 GitLab、Linear、Jira 和本地 markdown。有两个注意点。在没有原生阻塞关系的跟踪器上，wayfinder 从文字里推断依赖图，你必须手工纠正它。本地 markdown 把产物放进你的仓库，这一点不建议，因为存在仓库里的东西往往会被偶然留下来。开源维护者碰到相反的问题（公开的跟踪器被 agent 生成的规划工单填满），于是他们常常反而选本地 markdown。

**这个 grilling 让人筋疲力尽。每个问题都有三段长。**
这是关于 wayfinder 最尖锐的、还没人修的抱怨。有个用户这样拆开看：啰嗦本身就造成决策疲劳，而篇幅掩盖了 agent 为什么问这个问题，于是地图越长，你从决定到决定的那条链就越丢。这种啰嗦看着是当前那批[模型](https://www.aihero.dev/ai-coding-dictionary/model)的一个属性，不是 Skill 的属性。用户试过两种缓解：更低的 [reasoning effort](https://www.aihero.dev/ai-coding-dictionary/effort)（推理投入），以及在你全局的 `CLAUDE.md` 里写一条大白话的要求。这里照样得费脑子想。Wayfinder 要求你想很多，而这是它主要的用途，不是缺陷。

**一个我已经关闭的决定原来是错的。我改老工单还是新建一张？**
没有正式的指引，而 agent 默认的作法帮不上忙。它倾向于绕着那个坏决定做设计，而不是去质疑它，所以你自己得把它扭过来。管用的是照直告诉 wayfinder 什么变了。它随后会更新地图，修订受影响的工单，并在已关闭的那些上留言。地图中途的范围变化你救得回来。但如果你是*存心*把一张地图设计成会变的，那是范围错了的信号。

**`decision-mapping` 去哪了？**
就是这个 Skill。v1.1 把它改名为 `wayfinder`，你输入 `/wayfinder` 调用它。"Decision map" 是行话，而且它也不准确，因为四种工单类型里只有一种本身就是一项决定。新名字让这个 Skill 有了一套统一的词汇（目的地、战争迷雾、前沿、地图），不用在它之上再叠一个发明的术语。这个单位保留了 "decision" 这个词：一张 wayfinder 工单是一张**决策工单**，这样人们不会把它读成一张实现工单。

## 有效的话应该是这样

- 在任何一张工单存在之前，目的地已经写下来并且谈定了。
- 每张未完成的工单读起来都是一个问句。任何一张读起来像"把 X 建出来"的工单，要么类型标错了，要么属于地图下游。
- 你看一眼跟踪器就知道哪些工单可以拿，不用打开地图，因为原生阻塞关系把前沿显示了出来。
- 一场 session 解决一张工单，把答案作为 resolution 评论发出来，关闭它，并在地图的 *Decisions so far* 里加一行。然后停下。
- **Not yet specified** 那一节随时间缩小。当迷雾毕业成一张工单，它就从那一节里离开，不会同时出现在两处。
- 开场那次广度优先的 grilling 完全找不到迷雾时，这个 Skill 会停下来并告诉你这桩工作小到可以跳过地图。
- 完成地图的那场 session 把你指向一份 spec，不是一个 pull request。

## 它在整体中的位置

`wayfinder` 是一个**按情形选择的上匝道**，不是默认的起点。大多数活仍旧从那条由 grill 带着的"想法 → 上线"链起步。当想法大过一场 session 能装的量，你才用 wayfinder。它在 [to-spec](https://aihero.dev/skills-to-spec) 那里回到那条链，因为一张走完的地图交棒出去，不做构建。

大部分工作发生在 wayfinder 调度的别的 Skill 里。[grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 解决默认的那个工单类型，[prototype](https://aihero.dev/skills-prototype) 解决靠谈定不下来的工单，[research](https://aihero.dev/skills-research) 作为 subagent 运行，这样它读进来的东西不占用你的 session。[handoff](https://aihero.dev/skills-handoff) 把工作搬进搬出：从一场长得太大的对话搬进一张地图，以及当一场 session 中途冒出另一件小事时从地图搬出去。别的事情上，[ask-matt](https://aihero.dev/skills-ask-matt) 在整套 Skill 之间给你路由。
