## 它做什么

`grilling` 是那套面试式的循环，在任何人对一个计划、一个决定或一个想法采取行动之前，先把它压一压。它把对象画成一棵**设计树**：每个决定都分叉出挂在它下面的那些决定，然后它一个分支一个分支地问你，直到没有一个假设还停留在没说出口的状态。

它不是一次只问一个问题，也不是一次把所有问题问完。每一**轮**问的是完整的 **frontier（前沿）**：所有前置决定已经定下来的问题，其余一概不问。两个问题只要互相依赖，就绝不会被放进同一轮；一个悬在一个还没回答的问题上的问题，属于后面的某一轮。你的回答定下决定，frontier 向外推进，下一轮问的就是这腾出来的问题。十三个问题通常落在大约三轮里，而不是十三轮。

## 什么时候找它

输入 `/grilling`，或者当任务合适时 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（代理）会自己去用它。它是 grilling 这一族里唯一一份由模型自动调用的 [skill](https://www.aihero.dev/ai-coding-dictionary/skill)（技能），所以你很少需要自己输入它。通常是你*确实*输入的那份 Skill 替你把它跑起来了。

直接输入 `/grilling` 得到的就是纯粹的追问，别无任何东西。当你想要的比这更多时：

| 你手上有什么 | 该找谁 |
| --- | --- |
| 根本不在工作目录里干活 | [grill-me](https://aihero.dev/skills-grill-me)：同一场 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话），只是换一个 agent 绝不会自己触发的名字 |
| 你在一个工作目录里 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：同一场 session，而且它一边问一边写 `GLOSSARY.md` 和 ADR |
| 一桩大到装不进单场 session 的工作 | [wayfinder](https://aihero.dev/skills-wayfinder)：它画一张地图，并在决策工单里跑 grilling |
| 一个靠谈定不了的问题：某样东西该长什么样、该有什么感觉 | [prototype](https://aihero.dev/skills-prototype)：先把一次性版本做出来，然后再回来 |
| 你自己的 Skill 需要一套追问 | 在它里面调用 `/grilling`，别再另写一套追问 |

## 轮次、frontier，以及谁说了算

这份 Skill 站在三个概念上。

**设计树**是这个对象的模型：挂着决定的决定。**frontier** 是那些前置全部已定的决定构成的集合：agent 目前唯一能问的问题。**一轮**就是一个完整的 frontier，全问出来，全答回去。

一轮之内，每个问题都有固定格式：一个 `❓` 后面跟着编号和标题，然后是正文，然后 agent 的建议回答单独占一行，起头是 `➡️`。这个格式让你可以用编号回答整轮（"1 同意，2 选第二个，3 不同意，理由是"），而不用把问题原文抄回去。这个格式有一个已知的问题。建议有时是在*反对*问题的原始措辞，于是同意这条建议就等于对这个问题回答"不同意"。碰上这种情况，就照建议回答，并说明你在这么做。

设计的另一半是事实与决定之间的分工。事实是这份 Skill 自己的活儿。当前沿里的一个问题需要 [environment](https://www.aihero.dev/ai-coding-dictionary/environment)（环境）能确定的东西时，agent 会派出一个 [sub-agent](https://www.aihero.dev/ai-coding-dictionary/subagent)（子代理）去查，而不是来问你。这一轮不等这份调研结果；只有依赖它的那些问题才等。决定归你，agent 必须等。一个跑 `grilling` 的 agent 要是替自己回答了决定，这份 Skill 就坏了。frontier 为空时 session 结束。随后 agent 会等你确认双方理解一致，才着手去做你们商定下来的东西。

frontier 有一个限制。它由 agent 凭判断挑出来，不是从一张图上算出来的。所以它可能把两个问题放进同一轮，之后才发现其中一个的回答本该改掉另一个。唯一的补救就是告诉它。这会在下一轮重新打开受影响的那个分支。

## 哪些东西住在这里，哪些住在封装里

这一页讲机制。封装类 Skill 的页面回答人们最常问的问题。

| 问题 | 在哪一页回答 |
| --- | --- |
| 树、frontier、轮次、问题格式、事实与决定的分工 | 这里 |
| 一场 session 该跑多久、谈不出答案的问题怎么办、怎么避免一路点头 | [grill-me](https://aihero.dev/skills-grill-me) |
| 什么东西会写进 `GLOSSARY.md`、什么会变成一份 ADR | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |

## 常见问题

**我能回到一次只问一个问题的方式吗？**
可以，而且很大一部分用户就是这么做的。把下面这段加到你的全局 `CLAUDE.md`：

```
When grilling, ask one question at a time.
```

并非所有人都喜欢按轮提问的默认设置。读得慢的人、用第二语言工作的人、靠逐个提问的格式帮助自己集中注意力的人，都反馈说一次一个问题更适合他们。这个退出口是一个受支持的设置，不是一个绕道办法。

**`/batch-grill-me` 去哪了？**
进了这份 Skill。按轮提问曾经短暂地作为一份独立 Skill 发布，后来并进了 `grilling` 本身，于是所有建立在这个原语之上的东西（`grill-me`、`grill-with-docs`、`triage`、`wayfinder`）一次性都拿到了它。没有 `batch-grill-me` 可装，也没有独立的逐个提问的 Skill；上面那行 `CLAUDE.md` 就是回到一次一个问题的路径。

**一次问完整轮，肯定会丢掉我前面的回答本该引出的问题。不是吗？**
这是对轮次最常见的反对意见，而 frontier 回答了它。一轮里只包含互不依赖的问题，所以一轮里的任何回答都不可能让同一轮里的另一个问题失效。你的回答仍然会改变后面的内容，因为 agent 每一轮都从零重新算出下一轮。你确实会丢掉一些东西，但比"所有问题一次问完"所暗示的要少。见上面 frontier 的那个限制。

**它问完了就开始动手构建。**
确认闸门就是为此存在的。这份 Skill 完成的时刻是你说双方理解一致，不是 frontier 变空的时候。较弱较快的 [model](https://www.aihero.dev/ai-coding-dictionary/model)（模型）仍然会跳过这个闸门。用户报告这件事最多发生在较低 effort（推理投入）档或非一线的 model 上，它们把"一直问到双方理解一致"塌缩成两三个问题加一份提纲。如果你的会这样，可靠的解决办法是在你自己的 `AGENTS.md` 或 `CLAUDE.md` 里加一行，告诉 agent 未经允许不得实现。

**它没问我，自己把问题回答了。**
那是这一趟运行里的 bug，也正是这份 Skill 的文本要区分事实与决定的原因。最经常发生在另一份 Skill 调用 `grilling` 去解决一个工单时：agent 把那个任务当成了可以继续往下走的许可。同一条约束也是这里没有异步模式的原因。有人提过要做这样一个变体：读一个 GitHub issue，然后发出一份合并后的决定备忘。那是一份不同的 Skill，因为一场没人回答的 grilling session 产出的是 agent 的意见，不是你的。

**我能限制问题的数量吗？**
不能，而且不设上限是刻意的。有些计划需要三个问题，有些需要五十个；一个固定的天花板要么在最难的案例上把追问截断，要么在最简单的案例上显得武断。设计中的控制手段就是直说：告诉它收尾，或者停下，就地接受当前这个计划。如果一场 session 跑得非常长，原因通常是范围太大；把工作拆开，逐块追问。

**我单独装了 `grill-me`，什么都没发生。**
`grill-me` 是一份只有一行的 Skill，它的全部内容就是"跑一场 `/grilling` session"，所以它也需要装上这份 Skill。`grill-with-docs` 同理，它另外还需要 [domain-modeling](https://aihero.dev/skills-domain-modeling)。整套一起装可以避开这个问题；选择性安装就意味着连原语也要一起装上。

**`grill-with-docs` 跑了，但它从来没有加载 `grilling`。**
这是一个真实的 bug，仍未修复，用户在不同 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（运行外壳）和 model 上都报告过。一份 Skill 点名另一份 Skill，并不总能可靠地让那一份加载，而 `grill-with-docs` 点名了两份。迹象是一场所有问题一起涌来、毫无建议的 session。这说明 model 在临场即兴编一套追问，而不是在跑这套。直接问 agent 它有没有加载 `grilling` 和 `domain-modeling`，通常就能把它救回来。

## 有效的话应该是这样

- 一轮以一个编号列表到来，每个问题的建议单独占一行 `➡️`，你能用编号回答整轮。
- 一轮里没有任何问题需要先回答同一轮里的另一个问题。
- 后面的轮次问出了第一轮问不出的东西。
- 它自己去查事实（读文件、派一个 sub-agent），而不是来问你一个它本可以自己查到的东西。
- 后台跑的调研不会把这一轮卡住；只有依赖它的那些问题才等。
- 结束时它会停下，请你确认双方理解一致，而不是直接开工。
- 问题数量保持高位，轮次数量保持低位。

## 它在整体中的位置

`grilling` 是一个**原语**，不是你排程要执行的一个步骤：它是这套追问技巧唯一的事实来源，集中放在一处，于是每份需要追问的 Skill 都来这里取，而不是自己造一套。[grill-me](https://aihero.dev/skills-grill-me) 和 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 是它两扇由用户调用的门，而 `grill-with-docs` 是主构建链开始的地方，排在 [to-spec](https://aihero.dev/skills-to-spec) 前面。[wayfinder](https://aihero.dev/skills-wayfinder) 用它来解决决策工单，[triage](https://aihero.dev/skills-triage) 用它把一个含糊的报告追问成一份能干活的报告，[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在你选定一个要加深的候选之后用它走一遍那棵树。拿不准哪扇入口门合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
