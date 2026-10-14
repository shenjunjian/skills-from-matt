## 它做什么

当你没看懂一条消息时，你输入 `wait-what`。随后 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（代理）会把它刚说的话换个说法重讲一遍。它补上你缺的那部分上下文，用大白话写，并使用你项目 `GLOSSARY.md` 里的词汇。

这份 Skill 只有三行。这是设计，不是一份没写完的草稿。对抗啰嗦的 Skill 通常都会失败，因为它们自己长得太长。一份四百行的"要简洁"的 Skill 仍然会让 [model](https://www.aihero.dev/ai-coding-dictionary/model)（模型）保持啰嗦，因为 model 抄的是 Skill 的长度，不是它的要求。这一份只有一个精确打头的词，此外什么都没有。

## 什么时候找它

你通过输入 `/wait-what` 来调用它。agent 不会自己去用它，它也不该这么做。只有你知道自己是从哪里开始跟不上的。

一旦你发现自己开始略读，就用它。举个例子：agent 开始用它自己造的行话，一句话里塞了五个缩写，或者解释了一个你从没见过前提的决定。它修的是你已经在其中的那场对话。要压根别让行话说出来，用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它会提前把共享语言建起来。

## 名字就是机制

打头的那个词是 **wait**。"要简洁"是一条关于 agent 输出的指令，而 model 遵守它的方式是砍掉词，让你更看不懂。**wait** 讲的是*你*的状态。它说的是你在这里停止了理解。一个听到"简短点"的 agent 会写出被剪碎的片段。一个听到"等一下，你说得我听不懂"的 agent 会退回去，重新解释。

这个区别就是这份 Skill 的全部。每一个流行的治啰嗦的办法都在给*输出*命名：`/tldr`、`/no-fluff`、`/talk-normal`。model 会过度矫正成剪碎的片段，更短，却一点也不更清楚。给*听的人*命名，一次就要到了两半：更少的词**并且**是你缺的那部分上下文。

这份 Skill 说的是重讲**那个**，不是"最后那条消息"。让你跟丢的东西通常比一个段落更大，所以由 agent 决定要往回退多远。

## 它接的是你已经有的那套语言

正文复用你全局 `CLAUDE.md` 和你项目 `GLOSSARY.md` 里已经打头的那些词。ASD-STE100 Simplified Technical English 定下语域。那个通用语言提供名词。这份 Skill、`CLAUDE.md` 和 `GLOSSARY.md` 用的是同一套 [token](https://www.aihero.dev/ai-coding-dictionary/token)（词元），所以调用它不是一条新指令。它是对一条 agent 已经答应过的指令的提醒。

如果你没有 `GLOSSARY.md`（也没有一个 `GLOSSARY-MAP.md` 指向当前 context（上下文）对应的那一份），这份 Skill 照样能用。你只是丢掉了领域词汇那一半。

## 有效的话应该是这样

- 重讲出来的版本**更短也更清楚**，不是更短更生硬。
- 它补上了你缺的那个前提，而不只是删词。
- 项目里的名词替换掉了那些造出来的名词。你 `GLOSSARY.md` 里的那些术语回来了。
- 你可以连着用两次，而它不会退化成一味简短。

## 它在整体中的位置

你可以在任何时刻、任何对话里、任何别的 Skill 内部使用 `wait-what`。它事后修补一条消息。真正的解法是一门提前商定好的共享语言，那就是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：一场 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)（追问）session（会话），一边问一边跑 [domain-modeling](https://aihero.dev/skills-domain-modeling)，于是它把你俩用到的那些词记录进你的 `GLOSSARY.md`。拿不准这一刻哪份 Skill 合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
