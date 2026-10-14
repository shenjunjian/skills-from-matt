## 它做什么

`tdd` 以测试先行的方式构建一个功能或修复一个 bug：一个失败的测试，然后刚好让它通过的代码，接着下一个行为。它带着那套让这轮循环产出值得保留的测试的标准：什么是好测试、测试放哪、mock 是干什么用的，以及让一套测试变得一文不值的那三种反模式。

在你尚未事先约定的 seam（接缝）上，它一个测试都不写。在任何测试存在之前，它先点名它打算在哪里测试这些公开边界，然后停下来等你确认。测试的投入是有限的，这一步把它花在关键路径上，而不是花在每一个边界情况上。`tdd` 同时也是一份**参考**，不是执行者。它装着这轮循环的规则，而别的东西（你，或 [implement](https://aihero.dev/skills-implement)）来运行落实这些规则的那个 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）。

## 什么时候用它

输入 `/tdd`，或者当任务合适时 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（智能体）会自动用它：以测试先行的方式构建功能或修复 bug，或者你说出 "red-green-refactor" 的时候。

当有一个具体行为要构建、它有输入和可观察的输出，而你希望测试能在重构后仍然存活时，就用它。

| 你的处境 | 该去哪 |
| --- | --- |
| 一个输入输出都定义了的行为（业务逻辑、一条请求／响应契约、一次转换、校验） | `tdd` |
| 行为还没定下来 | [to-spec](https://aihero.dev/skills-to-spec)，它在任何代码写就之前就把测试 seam 约定好 |
| 真正的问题是接口的形状，不是测试 | [codebase-design](https://aihero.dev/skills-codebase-design) |
| 你有 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明）或 [工单](https://www.aihero.dev/ai-coding-dictionary/ticket)，希望整场构建替你跑完 | [implement](https://aihero.dev/skills-implement)，它按工单驱动 `tdd` |
| 配置、装配、粘合代码、类型标注、直白的 CRUD 转发 | 这里没有合适的；见下面那个未闭合的缺口 |

最后那一行是真实的缺口。这个 Skill 决定 seam 放在*哪里*，但它内部没有任何东西决定一次改动值不值得走这轮循环。如果你在一个没有独立事实来源可供断言的改动上运行它，你得到的就是一个把实现重述一遍的测试。这正是本 Skill 警告的同义反复反模式，只是从相反的方向撞上。它是 [issue #746](https://github.com/mattpocock/skills/issues/746)，仍未关闭。在它闭合之前，这个判断得你自己下，或者把规则写进你的 `CLAUDE.md`。

## 前置条件

需要安装 [codebase-design](https://aihero.dev/skills-codebase-design)。`tdd` 过去自带关于 deep module（深模块）和接口设计的笔记；v1.0 把它们删掉，改为依赖那个共享的 Skill，如今 `tdd` 使用它的接口设计词汇。此外没有别的要求；这个 Skill 是[无状态](https://www.aihero.dev/ai-coding-dictionary/stateless)的，也不写自己的文件。

## 这轮循环，以及它运转的 seam

本 Skill 建立在三个术语之上。

**red-green。** 写出失败的测试，然后只写刚好能让它通过的代码。不要为下一个测试预先写代码。没有 refactor 阶段。本 Skill 在 2026 年 6 月去掉了它，因为 agent 几乎从不执行这一步，而且 review 和实现作为独立会话分开进行效果更好。重构归 [code-review](https://aihero.dev/skills-code-review)。

**vertical slice（垂直切片）。** 在一个 seam 上写一个测试，然后是极简实现，然后重复。第一个循环是一支 **tracer bullet**（曳光弹），证明一条路径端到端走得通。相反的做法是水平切分：先写完所有测试，再写完所有代码。成批写出的测试验证的是*想象中*的行为。它们检查的是东西的形状，而不是用户做什么，并且在你还没理解实现之前就把你锁进一套测试结构。

**事先约定的 seam。** seam 是你观察行为所在的公开边界，无需伸手到里面去。这条规则没有例外。未经确认的 seam 上不写任何测试。在完整链条里，seam 更早就在 [to-spec](https://aihero.dev/skills-to-spec) 阶段约定好："`/tdd` 被告知只在事先约定的测试 seam 上工作，`/code-review` 检查是否只用了约定过的测试 seam。"单独调用时，`tdd` 会直接问你。

它天生要防止的三种反模式：

| 反模式 | 识别特征 |
| --- | --- |
| 与实现耦合 | 你重命名一个内部函数时测试就破了，尽管行为并未改变。mock 了内部的协作对象、断言调用次数、用数据库查询而不是接口来做验证。 |
| 同义反复 | 期望值的算法和代码的算法一样，于是测试按构造必然通过。期望值必须来自别处：一个已知正确的字面量、一个手工算出的例子、spec。 |
| 水平切分 | 一批测试在任何实现之前落地。 |

mock 只用于系统边界：外部 API、时间、随机数，有时是文件系统或数据库。绝不要 mock 你自己的模块。

## 常见问题

**它为什么不重构？描述里写的是 "red-green-refactor"。**

因为 refactor 这一步已经移除，而描述没改。移除是刻意的。agent 几乎从不执行那一步，而把实现和 review 放在不同会话里效果更好。结果按教条还算不算 TDD，远不如这轮循环是否产出更好的代码重要。触发短语和正文之间的不一致记在 [issue #589](https://github.com/mattpocock/skills/issues/589)，仍未关闭，所以 "red-green-refactor" 继续作为一个能触发本 Skill 的短语存在。你得到的是 red-green，重构在 [code-review](https://aihero.dev/skills-code-review) 里。

**它让我选一个测试 seam，而我完全不知道该选哪个。**

这是本 Skill 被报告最多的摩擦点（[issue #607](https://github.com/mattpocock/skills/issues/607)）。提示词只按名字列出候选 seam，不说明每个能抓住什么、漏掉什么，所以你是在几个标签之间做选择。目前还没有发布修复。实用的绕法是先向 agent 问清取舍再回答：组件级的 seam 会漏掉什么而集成 seam 能抓住，以及它慢多少。这也是链条选择在 `to-spec` 里事先约定 seam 的原因，那时你看得见整个功能，而不是一个提示词。

**它在测试之前就写了实现，尽管 Skill 说 red 先行。**

这会发生。一位用户就此事追问[模型](https://www.aihero.dev/ai-coding-dictionary/model)，得到一个异常坦诚的回答："我知道 Skill 写着'一次一个测试，看着它以正确的理由失败'。我读过了。我只是习惯性地回到了我平时的做法。"Skill 接受这件事。没有任何指令能让 agent 百分之百服从，而更严厉的措辞只会以微小的收益限制 agent 的创造力。即使 agent 没有严格遵守，这轮循环仍然值得跑，因为总体结果依然更好。如果某一个切片上严格要求很重要，就盯着那次运行，而不是指望 Skill 来强制执行。

**它应该先写浏览器或端到端测试吗？**

通常不该，而 Skill 也不会阻止。一位用户报告 agent 先写了一个 Playwright 测试，然后花掉很长一轮循环反复重跑，最后得出的结论是*测试*坏了，而那个功能还不存在。浏览器测试慢到 red-green 的反馈回路不再划算。在你仓库的 `CLAUDE.md` 里写明浏览器测试放在行为可用之后。

**`/tdd` 取代 `/implement`，或者课程里的 `/do-work` 吗？**

不。`/tdd` 记载方法论；`/implement` 是一个简单的工作→反馈→提交流程，是 `/do-work` 的直接替身。课程里那一个 `/do-work` 步骤如今拆给了 `/implement`、`/tdd` 和 `/code-review`。如果你问的是一张工单该跑哪一个，答案几乎都是 `/implement`。

**关于 deep module 和接口设计的指引去哪了？**

v1.0 时并入 [codebase-design](https://aihero.dev/skills-codebase-design)，并做了泛化，让几个 Skill 共用一套词汇。`refactoring.md` 同时离开；重构现在是 [code-review](https://aihero.dev/skills-code-review) 的职责，那个 Skill 带着源自 Fowler 的 code smell 基线。

**它知道我其他的工单吗？**

不知道。针对一张工单运行时，它可能提出本属于同批另一张工单的工作，因为它看不到工单图谱的其余部分（[issue #129](https://github.com/mattpocock/skills/issues/129)）。这不是 `tdd` 的职责。把 spec 和工单一起交给它会有帮助；而一开始就把工单切得合适更有帮助。

## 起作用的表现

- 在任何测试文件存在之前，它停下来、点名它打算在哪些 seam 上测试，并等待。
- 出现一个测试，变红，得到刚好够它通过的代码，然后才出现下一个测试，而不是一批测试接着一批代码。
- 测试名读起来像能力（"user can checkout with valid cart"），不像内部实现（"checkout calls paymentService.process"）。
- 断言里的期望值是你能够追溯到 spec 的字面量，不是照代码的算法重新算出来的值。
- 重命名一个内部函数，整套测试里没有任何东西被打破。
- mock 只出现在外部边界（支付 API、时钟），绝不围在你自己的模块外面。

## 它在整体中的位置

`tdd` 跑在主链条的构建步骤内部；它自己不是链条上的一步：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

[to-spec](https://aihero.dev/skills-to-spec) 事先约定测试 seam，[implement](https://aihero.dev/skills-implement) 按工单驱动 `tdd`，而 [code-review](https://aihero.dev/skills-code-review) 在之后检查是否只用了约定过的 seam，并承担 `tdd` 不再做的重构。它的另一个邻居是 [codebase-design](https://aihero.dev/skills-codebase-design)，`tdd` 所使用的 seam 与 deep module 词汇的共同来源。你也可以单独用它，只要有一个具体行为要构建、又不涉及完整的 spec。拿不准哪个 Skill 适合你的处境时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
