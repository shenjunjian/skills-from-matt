## 它做什么

`ask-matt` 是本仓库 Skill 之上的路由。你描述自己处在哪种情况里（一个不知道怎么开工的想法、一堆陆续涌进来的 bug 报告、一场跑了很久的 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）），它说出合适的那个 Skill 或那串 Skill，以及这串步骤里哪些决定要由人来拍板。

它给完建议就停。它不会 grill、不会写 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明）、不会打开文件、也不会触发它刚点名的那个 Skill；你拿到的结果是接下来该输入什么，输入这件事由你自己动手。它同时是一份手写的本仓库 Skill 地图，不是对你已安装内容的扫描，所以它不会路由到你自己写的 Skill 或别的作者写的 Skill。

## 什么时候找它

要输入 `/ask-matt` 才能调用它；agent 不会自己去找它。

| 你的情况 | 路由给回的结果 |
| --- | --- |
| 有个想法，完全不知道从哪起步 | 主流程的起点，以及这次构建是否小到可以跳过 spec |
| 别人不断送来的 bug 和需求 | [triage](https://aihero.dev/skills-triage) 入口，以及你自己生成的 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）为什么不该走这个入口 |
| 两个 Skill 看着可以互换 | 它们之间的界线，而且通常是一条具体的判据，不是口味问题。选 [grill-me](https://aihero.dev/skills-grill-me) 还是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，取决于你在不在工作目录里；选 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 还是 [wayfinder](https://aihero.dev/skills-wayfinder)，取决于这桩工作能不能在一次 session 里做完 |
| session 已经很长，还要就 [context](https://www.aihero.dev/ai-coding-dictionary/context)（上下文）做个决定 | 阶段边界上那五个选项的有序决策树 |
| 你已经选好一个 Skill | 没有有用的答案。直接调用那个 Skill。 |

## 前置条件

路由只点名 Skill，不安装它们。它指向的东西必须都已装好，这条推荐才落得了地，而它只认识本仓库里已转正的那批 Skill。

依赖跟踪器的那几条路线（triage、`to-spec`、`to-tickets`、`implement`）预设 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 已经在仓库里配置好了工单跟踪器。哪怕这一步还没做，路由照样会推荐它们。

## 是 flow，不是 Skill

这个 Skill 打头的词是 **flow**，一条*穿过*各个 Skill 的路径，而不是单个 Skill。你报出自己的情况，路由就把你放到某条 flow 的某一步上。这跟"这是一个匹配你关键词的 Skill"是两种不同的回答。路线一共五种，Skill 本身把它们完整描述了一遍：

- **主流程**，从想法到发布。grill、spec、工单、implement（一次一张工单，或者用 [implement-spec](https://aihero.dev/skills-implement-spec) 把整张任务图并行跑完）、review，然后是 [retro](https://aihero.dev/skills-retro)，它把这次构建学到的东西喂回 agent 的环境。这条流程有两个分支。一个是 prototype 绕行，用在某个问题要靠能跑的代码才能定下来的时候。另一个是 spec 与工单的拆分，只有当构建跨越多场 session，这份开销才划得来。
- **入口路线**，用在某种会自己产生工作、随后汇入主流程的情况：陆续进来的 bug 报告、某个坏掉的东西、或者一团大到看不清、装不进一场 session 的工作。
- **代码库健康**，是保养，不是新功能。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 巡查代码，寻找 deepening（加深）的机会，每找到一处就作为一个想法重新进入主流程。
- **独立项**，不挂在任何 flow 上，你单独用它们：prototype、那份问卷、research run。
- **垫在下面的一层词汇**，两份 reference，别的 Skill 在问题出在措辞而不是流程时把它们拉进来。

## 阶段边界

这个 Skill 另一个关键概念是 **phase boundary**（阶段边界）。一个 phase 是 session 内部的一块工作（[grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)、实现、QA），两个 phase 之间的边界是"这块上下文我该怎么处理"这个问题唯一成立的场所。phase 进行中没有要决定的事：要么继续，要么把剩下的拆给 [subagents](https://www.aihero.dev/ai-coding-dictionary/subagent)。

| 选项 | 什么时候选它 |
| --- | --- |
| **Continue** | 下一个 phase 需要这一个的原样内容，或者你还剩 [smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)（清醒的思考余量）。它是唯一让 session 继续充当 [primary source](https://www.aihero.dev/ai-coding-dictionary/primary-source)（一手来源）的做法，所以先判断能不能把它排除掉 |
| **`/clear`** | 身后的一切都可以丢掉。最省钱的选项，判断错了也撤不回来 |
| **[handoff](https://aihero.dev/skills-handoff)** | 有东西必须带走：换一个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（运行环境）、换一个目录、交给同事、或者在 phase 中间岔出去的一件小事 |
| **Subagent** | 任务边界收得足够紧，可以在你 [away from the keyboard](https://www.aihero.dev/ai-coding-dictionary/afk)（人离开工位）时由它跑 |
| **`/compact`** | 上面都不合适。这是默认选项，决策树常常收在这里 |

人常常在这几个里选错两个，这就是为什么路由给的是顺序，而不只是清单。`/handoff` 看上去像在上下文窗口之间搬东西的通用办法，其实不是。它只给你可移植性。`/compact` 是决策树的最后一项，不是第一项，因为排在它上面那四个选项要么更便宜，要么更精确。

## 常见问题

**不就是一份按正确顺序排好的 Skill 清单吗？**

一直有人要求在 README 里给一份。这个 Skill 就是那份清单。静态表格会写 `wayfinder → to-spec → to-tickets → implement → code-review → retro`，而且对大多数情况都是错的。要紧的是那些分支：有没有代码库、构建跨不跨 session、光靠谈能不能把这个问题定下来。代价是路由靠人手工维护，所以它落在仓库后面。`/grilling` 早就上线了，路由却很晚才提到它。

**它告诉我一半的 Skill 都没装。**

这是已知 bug，还没修。路由带你走的那批 Skill，大多设了 `disable-model-invocation: true`，也就是说 harness 会把它们从注入 agent 上下文的那份 skill 列表里省掉。agent 把那份列表当成完整清单，于是报告它们缺失。有一份报告里，它声称整条 spec 加工单的 flow 都不存在，改走光秃秃的 `/grilling` 和 `/tdd`。插件的二十七份 Skill 里有十六份带这个标记，所以这是常态，不是边缘情况。它们都装了。照样把 slash command 输进去，或者去查 `.claude-plugin/plugin.json`，那里才是当前有什么的权威依据。

**它描述了某个 Skill 的行为，那个 Skill 其实不做这事。**

这也是个真 bug，也还没修。路由是按它自己给每份 Skill 写的一句话摘要作答，不是按那份 Skill。一份详细的报告记录了同一场 session 里的三处，其中包括仅凭"把这轮对话变成一份 spec"这句摘要就建议跳过 [to-spec](https://aihero.dev/skills-to-spec)。路由一次都没有打开 `to-spec/SKILL.md`。每一处都是用户反驳之后它才去核对，从没有主动核对过。那次跳过 `to-spec` 让人漏掉了一次真正的 seam 检查，产出的工单也因此低估了工作量。当路由对别的 Skill 说出你会照着做的结论时，让它先把那份 `SKILL.md` 打开。地图完全没覆盖的问题也一样，比如该不该用 [plan mode](https://www.aihero.dev/ai-coding-dictionary/agent-mode)：那个答案是 [model](https://www.aihero.dev/ai-coding-dictionary/model)（模型）的推断，不是这里写下的内容。

**为什么是散文而不是一份编号清单？**

这个抱怨合理。有一个未关闭的工单主张，路由里大部分判断是确定性的，叙述式的写法让它很难扫读。没什么拦着你要压缩过的形式："直接把顺序给我"就能拿到顺序。散文承载的是条件部分：那些分支、哪里期待人来做决定、步骤之间在哪里该 `/clear`、哪里该 `/compact`。平铺的清单恰好丢掉的就是这些。

**它能在我自己的 Skill 或其他作者的 Skill 之间路由吗？**

不能。已经有三份各自独立的提案，要求做一个能读取你本地 `skills/` 目录、从已安装内容里给建议的路由。`ask-matt` 不是那个东西。它是一张单一集合的手写地图，靠人维护，对你自己写的或者从别处装的 Skill 一无所知。

**它让我去改一份 SKILL.md。**

这个建议往往没错，但那次改动很少活得久。有人问怎么让 [implement](https://aihero.dev/skills-implement) 关闭工单，被告知在 Skill 里加一行，他立刻就看到了问题所在。`npx skills update` 会覆盖那个文件，而插件的安装是只读的。把长期生效的行为放进你自己的 `CLAUDE.md` 或 `AGENTS.md`，或者在调用时说出来。在提示里做的改动熬得过更新。人们就是这样把 flow 指到 Linear 而不是 GitHub，或者问它哪几张未关闭的工单可以并行跑。

**它点名了一个我没有的 Skill，或者漏掉了一个我有的。**

别一上来就当它消失了，先查 changelog 里的改名。`writing-great-skills` 变成了 [writing-for-agents](https://aihero.dev/skills-writing-for-agents)，没留别名；`to-prd` 变成了 [to-spec](https://aihero.dev/skills-to-spec)；`pathfinder` 变成了 [wayfinder](https://aihero.dev/skills-wayfinder)。四份 Skill 直接退役，交给了吸收它们的后继：`ubiquitous-language`、`design-an-interface`、`qa` 和 `request-refactor-plan`。如果它漏掉了你手上某份 Skill，那就是上面说过的路由自身滞后。

## 有效的话应该是这样

- 它最后说出该输入什么就停在那里，而不是自己开工。
- 它给回的路线提到了在哪里该清空或压缩上下文、哪里期待你来审查，而不只是一串 Skill 名字。
- 两个 Skill 接近时，它说是哪一个，也说清你的情况为什么另一个不对。
- 它每句关于别的 Skill 行为的断言，在 trace 里都能看到它读了那份 Skill 的 `SKILL.md`。
- 你在它交回的东西里认出自己的处境，而不是最接近的那个通用场景。

## 它在整体中的位置

`ask-matt` 是一个 **standalone 路由**，罩在整个集合之上。它从来不是一条链里的一步。它指向每一条链，其余文档页都链回它，这样谁都不用重画那张图。从这里出发，你最常落到 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，也就是主流程的起点，或者 [triage](https://aihero.dev/skills-triage)，那个为"不是你先起头、而是送上门的工作"准备的入口。

在它自己描述的那些 Skill 之上，它是一份 [secondary source](https://www.aihero.dev/ai-coding-dictionary/secondary-source)（二手来源）。路由和某份 `SKILL.md` 说法冲突时，以 `SKILL.md` 为准。
