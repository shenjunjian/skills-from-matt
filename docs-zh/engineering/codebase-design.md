## 它做什么

`codebase-design` 固定你用来设计模块的那几个词：**module**、**interface**、**depth**、**seam**、**adapter**、**leverage**、**locality**。它给每一个词下精确定义，禁掉那些松垮的替代词（"component"、"service"、"API"、"boundary"），并说出由这些定义推出的那几条原则。

它是一份参考，不是一套流程。它不跑循环，不产出交付物，也绝不停下来向你提问。其余每一份涉及设计的 Skill 都在用它的词汇。单独用它，它给你词，然后就停。调用它之前要知道这点。你把一场 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）指给一个既没有流程也没有停止规则的 Skill 然后说"开干"，agent 就会自己编一套流程出来。下面的问题会告诉你那副样子是什么样。

## 什么时候找它

输入 `/codebase-design`，或者当一个设计任务对得上时，agent 会自己找上它。

当你已经知道要重新设计哪部分代码、需要琢磨它的形状时再找它：seam 放在哪、interface 能收多小、一次提取到底有没有把复杂度从它的调用方那里拿走。它也可以用来把一场关于某个设计词是什么意思的争论定下来。

有几份 Skill 跟它挨得很近。按你手上的问题来挑：

| 问题 | Skill |
|---|---|
| 单个模块的形状：它的 interface、它的 seam、它的 depth | `codebase-design` |
| 领域的*用词*："account" 指三件事，两个人说的 "cancellation" 不是一回事 | [domain-modeling](https://aihero.dev/skills-domain-modeling) |
| 你还不知道*哪个*模块该重新设计 | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)（负责找出候选项的那次巡查） |
| 你想要的是一套被辩论证过的设计，不只是被说出来的名字 | [grilling](https://aihero.dev/skills-grilling) |
| 有一个具体行为要构建，你想要一组能在重构后活下来的测试 | [tdd](https://aihero.dev/skills-tdd) |

## 词汇表

词汇表就是这个 Skill。它对每个词都拿其他词来定义，并给每个词配上它要取代的那个词。

| 术语 | 含义 | 不要说 |
|---|---|---|
| **Module** | 任何同时有 interface 和实现的东西。刻意与尺度无关：一个函数、一个类、一个 package、一个跨层的切片。 | unit、component、service |
| **Interface** | 调用方为了正确使用它必须知道的一切：类型签名，加上不变量、顺序约束、错误模式、必需配置、性能特征。 | API、signature |
| **Depth** | interface 处的杠杆：调用方或测试每学一份 interface，能驱动多少行为。**Deep**：小 interface 背后藏着大量行为。**Shallow**：interface 的复杂程度几乎和实现一样。 | 无 |
| **Seam** | Michael Feathers 的词：一个你可以改变行为而不必在该处编辑的位置。它是 interface 的*所在处*，而把它放在哪本身就是一个决定，与背后放什么无关。 | boundary |
| **Adapter** | 在某个 seam 处满足一个 interface 的具体东西。它点的是一个角色，不是一类东西：内存里的 fake 和一个 Postgres repo 都是 adapter。 | 无 |
| **Leverage** | 调用方从 depth 处得到的东西：每学一份 interface 换来更多能力。 | 无 |
| **Locality** | 维护者从 depth 处得到的东西：改动、bug 和验证都聚在一处，所以一次修复制覆盖所有调用方。 | 无 |

这个 Skill 并*不*把 depth 定义成实现行数与 interface 行数之比，而 Ousterhout 自己的定义正是如此。那个指标会奖励往实现里堆料，所以这个 Skill 改用 depth-as-leverage。

## 四条原则

- **Depth 是 interface 的属性，不是实现的属性。** 一个 deep module 可以在内部由可互换的小零件搭成，调用方就是看不见它们。一个模块可以有只有自己用的内部 seam 供它的测试使用，同时在它的 interface 上有一个外部 seam。
- **删除测试。** 设想把这个模块删掉。复杂度跟着消失了，说明这个模块只是一层管道。复杂度在 N 个调用方那里重新冒出来，说明这个模块一直在掩藏真实的复杂度。
- **interface 就是测试面。** 调用方和测试穿过同一个 seam。如果你想测试到 interface *之外*，说明这个模块形状不对。
- **一个 adapter 意味着假想的 seam，两个 adapter 才是真实的 seam。** 在有什么东西跨它变化之前，别切开 seam。单 adapter 的 seam 只是多一层间接。

两份配套文件讲得更深，这个 Skill 按需读取它们，而不是一开始就摊开。[DEEPENING.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DEEPENING.md) 把一个候选项的依赖归成四类（in-process、local-substitutable、remote-but-owned、true-external），因为类别决定了你怎么跨 seam 测试那个被加深的模块。[DESIGN-IT-TWICE.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DESIGN-IT-TWICE.md) 派出并行的 [sub-agents](https://www.aihero.dev/ai-coding-dictionary/subagent)（子代理），为同一个模块产出三个以上差异极大的 interface，然后在 depth、locality 和 seam 位置上比较它们。

## 常见问题

**在 TypeScript 里我到底怎么构建一个 deep module？**

这是关于这个 Skill 被问得最多的问题，而 Skill 不回答它。它定义 deep module *是什么*；对于怎么拦住一条随意的 import 越过 interface 直接伸手进内部，它一个字没说。[Issue #458](https://github.com/mattpocock/skills/issues/458) 说得直白："假设我们很满意这个 interface，它藏住了细节，等等。但我们怎么强制它？我认为没有 linting 或者明确的护栏，人和 LLM 都会慢慢把它搅乱。"那个讨论串里的回答给了三个选项：包进一个 class 或 IIFE，并接受这个 class 会长得巨大；把它做成 monorepo 里的一个 package，并接受 monorepo 的那套工具链；或者用 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 这样的 linter，禁止绕过 interface 的 import。这几种机制里，Effect 最好，dependency-cruiser 次之。仓库的 `in-progress/` 桶里有一份 `setup-ts-deep-modules` Skill，它铺下 `src/packages/<name>/index.ts` 这套约定，但它是一条 beta 通道的 Skill，没有文档页，也不带 lint 规则。

**我把一场 session 指给它，它烧掉 100k [tokens](https://www.aihero.dev/ai-coding-dictionary/token)（词元）去重新设计我根本没问过的事情。**

这是已知问题，登记在 [issue #449](https://github.com/mattpocock/skills/issues/449)。这个 Skill 由 model 调用，并且自称是词汇表，但它里面没有任何东西硬拦住 agent 把它当成一套可执行的流程。被要求"在 /codebase-design 里接着干，推进那些未决的决定"时，一个 agent 挑中了这份 Skill 里最像流程的那部分：`DESIGN-IT-TWICE.md` 里的并行 sub-agents。它把前一场 session 已经摸清的区域重新探索了一遍，并且工作了很长时间才问出第一个问题。驱动型 Skill 都带着护栏（检查点、一次只问一个问题、不自动推进），这个 Skill 一个都没有，因为它是一份参考。绕法是调用一个驱动型 Skill（`/grill-with-docs`、`/improve-codebase-architecture` 或 `/tdd`），把 `codebase-design` 当作它的词汇表。这个 issue 还开着。

**`design-an-interface` 去哪了？还有一个 `/interface-design` Skill 吗？**

这个 Skill 取代了 `design-an-interface`，并接管了它的内容。什么都没丢。它的 "design it twice" 技巧（由并行的 sub-agents 生成差异极大的设计，来自 Ousterhout）在这里以 `DESIGN-IT-TWICE.md` 的形式发布。另外，好几个人要求为 deep-module/thin-interface 这套理念单开一个 `/interface-design` Skill；这个 Skill 已经覆盖了这套理念，也没有计划另开一份。你要是冲着这两个名字中的任何一个来的，这一页就是。

**这不就是一套文件结构约定吗，比如目录、barrel 文件、feature slice？**

不是。人们在这点上推回过很多次，而 Skill 没变。[Issue #95](https://github.com/mattpocock/skills/issues/95) 提出一套形式化的分形树文件结构，作为 deep module 的具体实现；回复说这两件事互不相干："deep modules 讲的是 interface 的设计以及通过严格 interface 来访问，跟文件系统长什么样无关。用这套做法做出 shallow modules 也完全可能。"同一个话题在 #458 里又出现过："我觉得你把 module 这个概念跟文件系统绑得太紧了。文件系统当然可以是关于 module 形状的一个有用暗示，但构建 deep module 并不需要用文件系统。"词汇表把 **module** 定义成与尺度无关，是刻意的。

**`tdd` 真的用了这套词汇吗？**

现在用了，但有很长时间它没有。v1.0 把 `tdd` 里内联的 deep-module 说明删掉了，换成这份共享的 Skill，可没人补上一个指路的东西去接替它们，于是 `tdd` 自己定义了 "seam"，并且什么都不引用。那个指向现在补进 `tdd` 了，当未决问题是 interface 的形状而不是测试时，agent 会顺着它走。`tdd` 仍然拥有 "seam" 作为你*在哪一层测试*的边界；这个 Skill 拥有边界背后那个模块的形状。

**design-it-twice 这个模式在 Claude Code 之外能用吗？**

用不干净。`DESIGN-IT-TWICE.md` 写着"用 Agent 工具并行派出 3 个以上 sub-agents"，而 "Agent" 是一个 Claude Code [tool](https://www.aihero.dev/ai-coding-dictionary/tool)（工具）的名字。仓库为其他 [harnesses](https://www.aihero.dev/ai-coding-dictionary/harness)（运行环境）也准备了元数据，包括 Codex，那边可能压根没有叫这个名字的工具。所以并行设计这一段的可移植性不如这份 Skill 的元数据暗示的那样。[Issue #564](https://github.com/mattpocock/skills/issues/564) 跟踪这一点，仍然开着。

**我能往词汇表里加自己的概念吗，比如 connascence、module secrets、[progressive disclosure](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)（渐进式披露）？**

有人提过这些。[Issue #180](https://github.com/mattpocock/skills/issues/180) 加上 Parnas 的 module secrets 和 Page-Jones 的 connascence，作为一层命名，用来指*是什么*在跨 seam 泄漏，还附了一份能用的 diff；[issue #303](https://github.com/mattpocock/skills/issues/303) 提出在实现内部做 progressive disclosure，这样一个在公开 interface 上够深的模块，内部也有结构。两个都还开着，都没合并。出厂的词汇表刻意保持小，Skill 自己给了理由：一致的语言正是全部目的，而一个人人都用不一致的词，比没有这个词更糟。

## 有效的话应该是这样

- 设计讨论不再产出 "component"、"service"、"boundary" 这些词，开始产出 "module"、"interface"、"seam"。
- 有人能指着一处提议的提取，说清它通不通删除测试，不含糊。
- 一个提议的 seam 会连带说出第二个 adapter，而不只是第一个。
- 关于一个 interface 的讨论覆盖了不变量、顺序和错误模式，不只是类型签名。
- 调用它不会开启一场 session。如果 agent 只凭 `/codebase-design` 就开始读文件、提重构方案，它就是把参考当成了驱动器。

## 它在整体中的位置

`codebase-design` 是一个 **随时可取的独立项**，是垫在工程类 Skill 底下的那一层词汇，不是任何一条链里的一步。离它最近的邻居是 [domain-modeling](https://aihero.dev/skills-domain-modeling)，那是*问题领域*用词的平行参考，而不是模块的形状。这两个通常一起需要，因为把一个 deep module 命名好两边都得用。另一个是 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)：它巡查代码库寻找加深的候选项，并且用这份词汇表写下每一个，所以它负责找到那个模块，而你在这一页的词语里设计它。拿不准哪个 Skill 或哪条 flow 合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
