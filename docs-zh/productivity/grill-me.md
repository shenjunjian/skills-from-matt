## 它做什么

`grill-me` 接住一个**模糊的想法**，然后对你不断追问，直到你能对它做出承诺。你不需要一个已经想清楚的计划才能开始，因为这场 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）存在的目的就是把计划产出出来。它按**轮次**提问。每一轮是完整的 **frontier（前沿）**，也就是所有前置问题都已经被你定下来的那些问题。所以它绝不会问你一个依赖于它还没听到的答案的问题。

它是 **[stateless](https://www.aihero.dev/ai-coding-dictionary/stateless)**（无状态的）。它不写任何文件，也不留下任何工作区。唯一的成果是你脑子里那个想法变得更清楚了。

## 什么时候找它

要输入 `/grill-me` 才能调用它，[agent](https://www.aihero.dev/ai-coding-dictionary/agent)（代理）不会自己去找它。在一个**全新的对话**里启动它，不要接着一个已经有 agent 替你写好的计划往下走。

只要你有一个值得认真对待的想法（一个功能、一个产品方向、一个商业决策、一篇要写的东西），就可以立刻用它，远早于你想清楚它涉及什么。模糊不是等待的理由，因为这场 session 的存在就是为了消除模糊。如果你已经能精确地把这件事描述出来，那就不必追问它了。

三份 grilling Skill 里你要哪一份，取决于你面前摆着什么：

- **任何事、任何地方。** 用 `grill-me`。它不需要仓库，不写任何文件，主题也不必是代码。
- **有一个代码库要对齐。** 用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。它是同一套追问，但是 [stateful](https://www.aihero.dev/ai-coding-dictionary/stateful)（有状态的）：它读你的代码，并把学到的东西写进 `GLOSSARY.md` 和 ADR。
- **一场 session 装不下。** 用 [wayfinder](https://aihero.dev/skills-wayfinder)。它把这项工作画成一张地图，并在地图里跑 grilling 会话。

把 [plan mode](https://www.aihero.dev/ai-coding-dictionary/agent-mode)（规划模式）关掉。规划模式会催着 agent 赶紧产出一个计划，而你要的是它继续提问。

## 它是对话，不是面试

问题由这份 Skill 提出，但**你**掌握范围。这一点人们常常忽略。它决定了一场 session 究竟是把想法变成了决定，还是产出了一套自信的胡说。

失败模式是**被动**：四十个问题一路回答"同意、同意、同意"，最后拿到一份 agent 写的、你只是点了点头的计划。它感觉很有效率，因为它持续了很久。但你什么都没决定，而结果的确定性比它实际的确定性看起来更强。

主动意味着掌舵。一个问题不如你要的那么细致，就顶回去。范围开始漂移，就说出来。回答"我不知道"，而且是真心的。这份 Skill 是帮工程师的，不是替换工程师的。结果的质量取决于你回答的质量，不取决于问题的数量。

反过来那个错误真实存在，但更少见：在追问里待得太久，永远走不到代码。

## 能追问的和不能追问的

有些问题靠谈就能回答。另一些不行，再多追问也到不了答案。

"用一个长表单还是三个页面？"和"这个交互应该是什么感觉？"是**无法追问的**。你得先有东西可以反应，才答得出。碰到这类问题，就停止追问。用 [prototype](https://aihero.dev/skills-prototype) 做一个一次性版本，看一眼，然后回来用一行字回答。

当你试图把一个无法追问的问题谈清楚时，session 就会变得过长。agent 不停换措辞，你不停猜，范围随着不确定性一起膨胀。

## 有效的话应该是这样

- 你对某件事有异议。一场没有被你反驳的 session，是一场你并不需要的 session。
- 问题分几轮到来，而不是长时间一个一个地问，而且后面的轮次明显建立在你前面说过的话上。
- 最后你到了一个自己没预料到的地方，因为某个问题让你看见了一个你一直在不自觉做出的决定。
- 结束时，你能向一个不在场的人辩护每一个选择。

## 常见问题

**我该预期多少个问题，我怎么知道它什么时候结束？**
数轮次，不数问题。四轮里问四十六个问题，是一场平常的 session。它结束于 frontier 为空：每一个分支都走完了，没有任何东西还停留在没说出口的假设上。

**它问了我两百个问题，哪里出错了？**
通常是范围太大。先让 agent 把工作拆成更小的块，然后逐块追问。超长的 session 还会漂进 **[笨区](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**，那里的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)（上下文窗口）已经满到让问题质量下降。

**我能回到一次只问一个问题的方式吗？**
可以。把下面这段加到你的全局 `CLAUDE.md`：

```
When grilling, ask one question at a time.
```

**如果我真的不知道答案呢？**
直说。"我不知道"是一个真实的回答，而一个你答不上来的问题，通常是在提示你去做原型，而不是去猜。

**我是不是该在写 spec（规格说明）之前开一场新 session？**
不是。这场 session 的价值就在于你刚刚建立起来的 [context](https://www.aihero.dev/ai-coding-dictionary/context)（上下文）。把同一个对话直接交给 [to-spec](https://aihero.dev/skills-to-spec)。

**model（模型）重要吗？**
比大多数 Skill 都重要。追问依赖 [model](https://www.aihero.dev/ai-coding-dictionary/model) 自己对系统如何出故障的了解，所以用你手上最好的那个。实现大多跟着上下文走，所以那里用一个便宜的模型就够了。

## 它在整体中的位置

`grill-me` 是一份**独立的、你可以在任何地方对任何东西使用的 Skill**。因为它是无状态的，所以它是便携的。它不需要仓库、工作区或任何配置，也不假设这个想法跟软件有关。人们用它来做商业决策、做写作、决定下一步做什么：任何自己一个人想不清楚的事。

便携是它与 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 唯一的区别。那份 Skill 跑同一套追问，但会读一个代码库作为对齐对象，并把学到的东西记录成 `GLOSSARY.md` 和 ADR。两者底层都使用 [grilling](https://aihero.dev/skills-grilling) 这份 Skill。`grill-me` 是由用户调用、且不保留任何状态的入口。

如果你追问的东西确实变成了软件，你可以把同一个对话交给 [to-spec](https://aihero.dev/skills-to-spec)，接着走构建流程（这是一个选项，不是这份 Skill 的目的）。拿不准哪条流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
