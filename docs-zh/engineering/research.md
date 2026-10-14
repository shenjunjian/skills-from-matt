## 它做什么

`research` 靠阅读那些真正拥有答案的资料来源来回答问题，然后在仓库里留下一个带引用的 Markdown 文件。它只依据 **[一手资料](https://www.aihero.dev/ai-coding-dictionary/primary-source)** 工作：官方文档、源码、spec（规格说明）、第一方 API。它把每一条论断追溯到拥有它的来源，所以只要 API 自己的文档可达，它就不会重复某篇博客对它的说法。

它不在对话里回答你。产出是一个文件，写在仓库已有的这类笔记所在的位置，每条论断都带一个链接。你拿到的是一份可以回应、可以交给另一个 agent、也可以直接丢掉的文档，而不是一个 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）一结束就消失的答案。

## 什么时候用它

输入 `/research`，或者当任务变成大量阅读杂活时 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（智能体）会自动用它。

当下一步是*从工作目录之外查清某件事*（某个第三方 API 的行为、某个 spec 的说法、某个版本声明是否成立），而你又不想让自己的线程停下来做阅读时，就用它。你需要的东西决定该用哪个 Skill：

| 你需要什么 | 用什么 |
| --- | --- |
| 一个决策正等着的外部事实 | `research` |
| 一个*和你一起*通过访谈做出的决策 | [grilling](https://aihero.dev/skills-grilling) |
| 一条持久的架构决策，写进 `GLOSSARY.md` 和 ADR | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 查清某个做法在你的代码库里是否行得通 | [prototype](https://aihero.dev/skills-prototype) |
| 一个大到一个会话装不下的计划 | [wayfinder](https://aihero.dev/skills-wayfinder) |

`research` 和 `grill-with-docs` 之间的界线是**产出物的保质期**。research 产出的事实寿命很短，比如这个库的鉴权机制本周是怎么做的。ADR 记录的则是你要长期保留的决策。如果你正在产出的是决策而不是事实，你在做的是 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)（拷问），不是研究。

## 委派出去的阅读杂活

阅读工作由一个**后台 agent** 执行。你继续干活，它把每条论断追到一手来源、写出一个 Markdown 文件、然后回报。research 是把杂活委派出去，不是把思考外包出去。你拿到的是一份可以用来拷问、规划或设计的文档，决策仍然由你做。

没有任何东西阻止这个后台 agent 再生出自己的后台 agent。这是本 Skill 记录得最清楚的问题。

文件放在哪里由仓库决定，不由 Skill 决定。它遵循笔记已有的任何约定。若没有约定，它挑一个合理的位置并告诉你它放在哪。每次运行写一个文件。

## 常见问题

**它又派生了第二个 research agent。这是应有的吗？**

不是。这是一个未修复的 bug，[issue #530](https://github.com/mattpocock/skills/issues/530)。Skill 告诉它的调用方起一个后台 agent，却没有限定 agent 类型。于是调用方起了一个 `general-purpose` agent，而这个 agent 手上有 `Agent` 工具、拿着同一份指令，就又照做一遍。有一位报告者测到单个研究任务在三次重叠运行中花掉约 450k [tokens](https://www.aihero.dev/ai-coding-dictionary/token)，重复的那次半小时后才结束，而用户看不见它。这在 Claude Code 之外也会发生。用户确认在 Codex 配 GPT-5.6-sol 上有同样的嵌套。没有随版本发布的修复。有人给自己安装的那份副本打了一行补丁，告诉已经是一个 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) 的 agent 自己动手完成工作，这有帮助，但它只是一条指令，不是结构性修复。调用 Skill 之后，盯着你的后台任务列表，把重复的那个停掉。

相反的失败也会发生。如果你自己的全局指令禁止 agent 再次委派工作，后台 agent 就会拒接任务，而 Skill 什么都不做，也不告诉你。

**文件该放哪，我该提交它吗？**

Skill 把文件放在仓库已有的笔记位置，对此没有更多主张。社区观点大体一致：保留 ADR，不保留研究文件。有一个 Discord 讨论串把这个问题说得最清楚："ADR 要。其他的一切做完就归档或删掉。否则它就成了工作的残渣，如果你已经偏离了 spec／研究，它会污染之后对仓库的读取。"研究文件记录的是它写就那天为真的东西，所以一份过期的比没有更糟。总体看，这些文件不该进 git，也没有标准的安放位置。人们改用 Obsidian、单独的知识仓库，或者工单跟踪器。

**什么算"高信任"的一手来源，由谁判定？**

由[模型](https://www.aihero.dev/ai-coding-dictionary/model)判定。Skill 点名了合格来源的*种类*（官方文档、源码、spec、第一方 API），没有白名单、没有域名关卡、没有核验环节。这正是该 Skill 首次提出时最响的反对意见，而且没有公开回应过："五个 research subagent 指向垃圾，只会让你更快拿到五个自信的错答案。你怎么把关什么算高信任来源？"你唯一的保护是每条论断上的引用。随机跟着走两三个。如果它们落在某物的概述而不是某物本身，这次运行就失败了。

**后面的会话会复用早先一次运行的发现吗？**

不会。没有任何东西会自动加载过去的研究文件。文件留在仓库里，直到某个人或某个 Skill 指向它。这是对设计最早也最有力的质疑："价值在于 markdown 成为 agent 之后重读的上下文，不在于抓取本身。一个写一次就死的文件只是个花样搜索。"已发布的 Skill 没有解决这个问题。实践中，只有你自己把它喂给下一步时它才有用：附到 spec 里、引到 grilling 会话里、让一个 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）指向它。

**为什么不直接让 agent 去读文档？**

你可以这么做，而那句精确表达这个意思的两行提示词，正是本 Skill 取代的做法。Skill 提供两样提示词给不了的东西。它在后台运行，所以你会话的 [context](https://www.aihero.dev/ai-coding-dictionary/context)（上下文）保持干净。而一手资料约束和带引用的文件输出每次都一样，不取决于你怎么措辞提示词。和 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（运行框架）自带的深度研究模式相比，差别在于那个文件和那条一手资料规则，不在于搜索。如果一个小问题用两行提示词就能拿到你要的东西，就用两行提示词。

**它什么时候停止阅读？**

Skill 没有停止条件。这表现为两种看似相反、成因相同的抱怨：agent 钻得过深，以及 agent 把一个主题铺得很广却漏掉唯一要紧的那个细节。有一位实践者这样描述："深度研究类 Skill 有时钻得过了头。而让 agent 去做研究，通常会漏掉关键的细节。"范围得由你来定。一个收窄、可回答的问题（一个 API、一种行为、一条版本声明）带回的结果远好于"研究 X"。

**`/wayfinder` 建了 research 工单。我要自己去解决它们吗？**

不用，现在它替你发出去了。在 v1.1 之后尚未发布的改动里，制图会话会为每张 research 工单起一个 `/research` subagent，并行运行。每个 subagent 把它的发现记录在一次性 `research/<name>` 分支上，工单上留一个 [context pointer](https://www.aihero.dev/ai-coding-dictionary/context-pointer)（上下文指针）。research 工单是 wayfinder"每会话一张工单"规则的唯一例外，因为它们是 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk)（无需你在场）：没有任何东西等着你。一个已知问题：之后删除分支会破坏工单里的上下文指针。

## 起作用的表现

- 你自己的会话照常继续。如果你坐着看它读，说明委派没有发生。
- 只出现一个新增后台任务。第二个名字几乎相同的，就是嵌套 bug。
- 出现一个新增 Markdown 文件，位于仓库已经在用于笔记的目录，而且 agent 告诉你路径。
- 里面每条论断都带链接，随机跟两个，你会落在官方文档、spec 或源文件本身上，而不是落在别人对它的转述上。
- 只凭这份文件你就能做出原先卡住的那个决定，不必自己回头查来源。

## 它在整体中的位置

`research` 是随手可用的独立 Skill。它为思考类 Skill 供料，不是构建链上的一环。你把它产出的文件*带进*流程。[grilling](https://aihero.dev/skills-grilling) 和 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 在已经掌握事实时能问出更尖锐的问题，[to-spec](https://aihero.dev/skills-to-spec) 可以据它做综合。[wayfinder](https://aihero.dev/skills-wayfinder) 是唯一直接调用它的 Skill，它用 `/research` subagent 解决地图上每张 research 工单。整张地图见 [ask-matt](https://aihero.dev/skills-ask-matt)。
