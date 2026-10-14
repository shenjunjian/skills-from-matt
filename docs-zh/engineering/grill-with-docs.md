## 它做什么

`grill-with-docs` 就一个计划或设计向你提问，一直问到你和 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（代理）对它形成同一个理解为止，并且在提问的同时把词汇和那些难以撤销的决定写进你的仓库。它跑的就是 [grill-me](https://aihero.dev/skills-grill-me) 那套提问（一轮问题，然后等你回答，再来下一轮），只不过对准的是一个代码库。

它是 **[stateful](https://www.aihero.dev/ai-coding-dictionary/stateful)**（有状态的）。其余每一份 grilling Skill 都把 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）留在你脑子里，这一份会在磁盘上留下文件。一个术语一旦定下来，这个 Skill 立刻把它写进 `GLOSSARY.md`，不是到最后一次性补上。一个决定过了三道闸门，这个 Skill 就把它写成一份 ADR。差别全在这里，而人们用它时遇到的大部分麻烦也出在这里。产出物是真实仓库里的真实文件，所以它们可能在你以为该有的时候缺席，也可能在多个人同时书写时跑偏。

## 什么时候找它

要输入 `/grill-with-docs` 才能调用它，agent 不会自己去找它。

在一次改动刚开始的时候用它，前提是你在一仓库里，计划还模糊，这件事的叫法也还没定下来。它是单次 session 的工具。你要哪一份 grilling Skill，取决于你面前摆着什么：

| 你手上有什么 | 该找谁 |
| --- | --- |
| 根本不在工作目录里干活 | [grill-me](https://aihero.dev/skills-grill-me) |
| 一个仓库，外加一次能在单场 session 里定下来的改动 | `grill-with-docs` |
| 一桩大到装不进单场 session 的工作（全新项目、大功能） | [wayfinder](https://aihero.dev/skills-wayfinder) |
| 一个完全没有领域文档的仓库，也没有特定要做的功能 | `grill-with-docs`，对准仓库而不是对准某次改动 |
| 一个决定卡在只有某个人脑子里才有的知识上 | [to-questionnaire](https://aihero.dev/skills-to-questionnaire) |

跟 wayfinder 的分界归结到 session 数量：单场 session 的规划用 `/grill-with-docs`，跨多场的规划用 `/wayfinder`。

## 前置条件

这个 Skill 往你的仓库里写东西，所以你得处在一个可以安全写入的位置。定下来的术语进根目录的 `GLOSSARY.md` 词汇表；如果根目录的 `GLOSSARY-MAP.md` 把这个仓库标记成多 context，那就进对应那个 context 的 `GLOSSARY.md`。决定进 `docs/adr/`。这两处 Skill 都只在需要时才创建。第一个术语或第一个决定定下来之前什么都不存在，所以你不必预先搭建任何东西。

它还需要另外两份 Skill 就位，因为它自己的 `SKILL.md` 只有一行，把活儿委托给那两份。[grilling](https://aihero.dev/skills-grilling) 提供提问，[domain-modeling](https://aihero.dev/skills-domain-modeling) 提供书写。只装 `grill-with-docs`，你得到的是一份跑不起来的 Skill。

## 留下的书面痕迹

一场 session 出来三样东西，它们不平级。

| 定下来的是什么 | 落在哪里 |
| --- | --- |
| 一个术语：项目自己对某样东西的叫法 | `GLOSSARY.md`，就在它定下的那一刻写进去 |
| 一个难以撤销、缺少上下文就令人意外、且确实构成取舍的决定 | `docs/adr/` 下的一份 ADR |
| 你们定下的其余一切 | 只在那场对话里，别处没有 |

第三行最让人措手不及。`GLOSSARY.md` 只是一份词汇表。它不放实现细节，不放 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明），也不放草稿笔记。一份 ADR 要同时满足那三个条件，所以多数决定不够格，多数 session 一份 ADR 也不产出。一场 session 让词汇表更锋利、ADR 数量为零，这符合设计意图，但这也意味着你们谈定的大部分内容只存在于你们谈定它的那个 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)（上下文窗口）里。把同一场对话交给 [to-spec](https://aihero.dev/skills-to-spec)，而不是先把它 [clearing](https://www.aihero.dev/ai-coding-dictionary/clearing)（清空）。

词汇表是主要的产出。这个 Skill 构建领域语言：项目自己的那些词，一次谈定，于是你、agent 和你的同事不必再重新推导一遍。不是所有人都认同这能提升 agent 的表现。最有力的反对意见是：一个术语和它的英文展开说明在 [model](https://www.aihero.dev/ai-coding-dictionary/model)（模型）那里得到同样的结果，词汇表主要是让共享这套词的人之间说话更短。按这个看法，词汇表仍然有价值，但价值落在人身上。

## 常见问题

**我该用这个还是 `/wayfinder`？**
范围说了算。能在单场 session 里定下来的都用这个；工作大到一场装不下时再用 [wayfinder](https://aihero.dev/skills-wayfinder)，它先把工作画成一张由决策 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）组成的地图。wayfinder 更慢也更密，在一个范围清晰的功能上去找它是最常见的误用。它不替代这个 Skill，而且它可以为地图上合适的那几部分起一场 grilling 会话。

**它跑完了，但既没有 `GLOSSARY.md` 也没有 ADR。**
有两个已知原因。第一个是没有任何内容够格：ADR 要过三道闸门，而一场讲的是某次改动、又没有新词出现的 session 本来就没东西可写。第二个是真 bug。当这个 Skill 跑在另一层编排里（一套 spec-driven-development 封装、一个多 agent 框架、一条把它当作别人流水线中某一步来调用的规则），用户报告写文件那一半悄悄没有发生，提问却照常进行。这个 bug 已登记，仍未修。你要是处在这样的配置里，先检查一遍工作目录，再决定信不信这场 session 的产出。

**它一口气问完所有问题，没有建议，也一次都没提到 `GLOSSARY.md`。**
那是这个 Skill 没能加载它的两个依赖。因为 `SKILL.md` 只有一行委托，一个没有加载到 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 的 agent 只能猜 grilling 是什么意思，于是你拿到的问题是全部一起涌来、毫无结构。部分加载更让人困惑：`grilling` 加载了、`domain-modeling` 没有，你就得到一场不错的提问和一片空白的书面痕迹。这事多久发生一次取决于 model 和 [effort](https://www.aihero.dev/ai-coding-dictionary/effort)（推理投入）档位，而它是这份 Skill 被报告最多的问题。怀疑遇上了，就直接问 agent 它加载了哪几份 Skill。

**我其余那些决定都去哪了？**
只留在对话里。这是关于这个 Skill 最严重、仍未解决的意见。词汇表不是 spec，多数回答换不来一份 ADR，也没有任何记录把每个定下的回答同一条 spec、一张工单、一个测试连起来。后面的步骤会把精确的回答（顺序保证、否定式需求、数值默认项）磨成更弱的散文，成品看起来完整，缺的正是你定下的那件事。眼下，留着这场 session，直接把它喂给 [to-spec](https://aihero.dev/skills-to-spec)，然后拿你自己的回答逐条重读那份 spec，别假定它全都收进去了。

**能把它指到一个完全没有文档的现有仓库上吗？**
能。代码库没有 ADR、没有领域语言、没有设计原则，这个 Skill 就是为这种情况准备的：调用它，说"帮我把仓库记录下来"。用户常把它和 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 搭配，用来建立或修复 `GLOSSARY.md`。准备好自己来掌方向。它读代码，就它读到的东西向你提问，而代码库里已有的哪些词才是对的，由你来定。

**session 结束后我该做什么？**
这个 Skill 的结束语常常没有落点，这是已知问题。在主流程里，答案是同一场对话里接着用 [to-spec](https://aihero.dev/skills-to-spec)。如果改动小到可以立刻动手构建，就直接走 [implement](https://aihero.dev/skills-implement)。

**为什么叫这个名字？**
没人对这个名字满意。有一个未关闭的建议把它改名为 `grill-domain-model`，那个名字对行为的描述更准确。目前还没有进展。改名一旦落地，文档页跟着挪，URL 也会变。

## 有效的话应该是这样

- `GLOSSARY.md` 在 session *进行中*就在变，一个术语一个术语地加，而不是到结尾整块冒出来。
- 词汇表读起来是纯粹的词汇（你项目的词，配紧凑的定义），里面没有实现细节，也没有像 spec 的散文。
- 代码库能回答的问题，靠读代码库得到答案，不会来问你。
- 你拿到很少甚至没有 ADR，而拿到的那些都是你一旦要重新争论就会恼火的决定。
- 它会质疑你用过的某个词，因为你已有的词汇表给它的定义不一样。

## 它在整体中的位置

`grill-with-docs` 是主构建链的开头：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

它排在任何东西被写成 spec 之前。它产出共享的理解和定下来的词汇，随后由 [to-spec](https://aihero.dev/skills-to-spec) 把它们合成，而不会再重新问你一遍。跟它最近的两个邻居是 [grill-me](https://aihero.dev/skills-grill-me)，也就是同一套提问，只是没有仓库也没有文件；以及 [domain-modeling](https://aihero.dev/skills-domain-modeling)，也就是它所驱动的那套词汇表加 ADR 的纪律。这两份都用 [grilling](https://aihero.dev/skills-grilling) 这个原语来做提问。在它的上游，[wayfinder](https://aihero.dev/skills-wayfinder) 给大到单场 session 装不下的工作画图，也能把地图的一部分往下交回给它。拿不准哪份 Skill 或哪条 flow 合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
