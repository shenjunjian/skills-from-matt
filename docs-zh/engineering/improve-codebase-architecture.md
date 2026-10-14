## 它做什么

`improve-codebase-architecture` 在代码库里巡查 **deepening opportunities**（加深的机会）。指的是这样一些位置：一个 shallow module（浅模块，interface 的复杂程度几乎和它掩藏的东西一样）可以变成一个 deep module。它把这些写成一份自包含的 HTML 报告，然后就你挑中的那一处 [grills](https://www.aihero.dev/ai-coding-dictionary/grilling)（追问）到底。

它从不改代码。整次运行产出一场对话，外加你操作系统临时目录里的一个 HTML 文件。重构你之后再做，在另一场 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）里，走正常的构建流程。所以它是一次巡查，不是一个重构工具，你可以在一个你还不打算动的代码库上跑它。

有两道闸门防止这份报告沦为一把通用清理建议。第一，每个候选项必须过**删除测试**：把这个模块删掉，它的复杂度会收到一个更小的 interface 背后，还是散到各个调用方那里？只有前一种情况能拿到一张卡片。第二，除非你把它指向某块具体区域，它先读最近的提交历史，把扫描聚焦在那些频繁变动的路径上。在没人碰的代码里做一次加深，是一笔永远收不回本的重构。

## 什么时候找它

要输入 `/improve-codebase-architecture` 才能调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent)（代理）不会自己去找它。

它不是主构建循环里的一步。你周期性地跑它，给代码库排进更多改进工作。人们在四种情形下用它：

| 情形 | 用法 |
| --- | --- |
| 例行保养 | 每隔几天跑一次，或者手头有空时跑，这样结构不会在两个功能之间烂掉。 |
| 一次大构建之前 | 把它指向 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明），问"我们怎么让这次改动变轻松？"这是对它最有效的提示词。 |
| 既有项目体检 | 在一个庞大、没有结构或者靠 [vibe-coded](https://www.aihero.dev/ai-coding-dictionary/vibe-coding)（凭感觉写）堆出来的仓库上跑它，看看它现在是什么形状。 |
| 遗留代码的测试工作 | 用它找出缺失的 seam，然后才好对不可测的代码写测试。 |

它和几个近邻容易混淆的地方：

- 要设计一个你已经选定的单个模块，用 [codebase-design](https://aihero.dev/skills-codebase-design)。这个 Skill 负责找出该动哪个模块，而 `codebase-design` 是你设计它的地方。
- 一整桩大到装不进单场 session 的工作，用 [wayfinder](https://aihero.dev/skills-wayfinder)。
- "这个具体的东西坏了"，用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。当真正的发现是没有一个好的 seam 能把这个 bug 钉住时，它会把你送回这里。

## 前置条件

要跑它，什么都不需要。如果 `GLOSSARY.md` 或者 `docs/adr/` 里的 ADR 存在，它会读，并用你领域自己的名词。于是某个候选项读起来是"加深 Order intake 这个 module"，而不是"重构那个 FooBarHandler"。

它会在两个地方写入。报告写到 `<tmpdir>/architecture-review-<timestamp>.html`，在仓库之外。在 grilling 循环里，它会在 `GLOSSARY.md` 里添加或磨利术语，那个文件不存在就创建。它还会主动提出把一个被你否掉的候选项记成一份 ADR，这样下一次运行不会再提它。

## depth，以及那份搜寻它的报告

这个 Skill 落在一个概念上：**depth**（深度）。一个 deep module 把大量行为放在一个小而稳定的 interface 背后。一个 shallow module 通过一个几乎和底下代码一样宽的 interface 把自己的实现暴露出来。报告要找的是浅薄的三种形态：

- 仅仅为了可测试才抽出来的纯函数，而真正的 bug 在各调用方怎么用它们（缺少 **locality**，局部性）。
- 越过自己的 **seam**（接缝）往外泄漏的模块。
- 一个你不打开五个文件就看不懂的概念。

对每一处，它提出能修好它的那次加深。

每个候选项是一张卡片：涉及的文件、摩擦点、一句大白话的解法、以 **locality** 和 **leverage**（杠杆）表述的收益、一张前后对比图，外加一枚强度徽章。

| 徽章 | 对你意味着什么 |
| --- | --- |
| `Strong` | 删除测试过得清楚，摩擦也是真的。认真对待这些。 |
| `Worth exploring` | 加深说得通，但回报取决于这段代码接下来往哪走。 |
| `Speculative` | 为了完整才列进来。这些你可以大多忽略。 |

报告以一个 **Top recommendation**（首要推荐）收尾，也就是它会先做的那个候选项。到这里 Skill 就停下，问你想探索哪个候选项。此刻你还没定下任何事，代码也一行未改。

## 你选定一个之后会发生什么

你选定一个候选项，就在它上面起一场 [grilling](https://aihero.dev/skills-grilling) 会话。它覆盖约束条件、什么放进 seam 背后、哪些测试还能留下、加深后的 interface 该长什么样。那场会话的产出是一个决定，不是一个 diff。从那里开始就走正常流程：把决定带进 [to-spec](https://aihero.dev/skills-to-spec)，然后 [to-tickets](https://aihero.dev/skills-to-tickets)，然后 [implement](https://aihero.dev/skills-implement)。

## 常见问题

**它围着一个想法追了我一个小时，而不是给我一堆选项。能关掉吗？**

能。调用的时候说明（"别追问我，直接把报告给我"）。这是关于这个 Skill 最常见的抱怨。有一位用户喜欢它，说它是"拿到一份透彻的改进分析省事的做法"，可自从加了 grilling 这个循环，他觉得它"几乎没法用"。在他的 session 里，它只提一个解法，然后问出"几十甚至上百个问题"。设计意图是报告先来，grilling 只在你已经选定的那个候选项上开始。但弱一些的 [models](https://www.aihero.dev/ai-coding-dictionary/model)（模型）会直接跳到对自己想到的第一个主意的提问。那个讨论串里的结果随模型差别很大。这是一个未关闭的 issue，这个 Skill 也还没有一份写进文档的免追问模式。

**报告打开是一堆没有样式的裸 HTML，一张图也没有。怎么回事？**

报告从 CDN 加载 Tailwind 和 Mermaid，所以打开它时要有网络。如果有什么东西拦住了这些脚本，页面就崩掉，而且没有报错。在被报告的那个案例里，一个安全钩子要求 SRI 哈希。agent 加上了，但 CDN 给浏览器的字节和算哈希的那次 `curl` 拿到的不一样，于是浏览器拦下了脚本。离线环境和受管控环境是同一个问题。agent 看不见这件事，因为它从不渲染那个页面。作为一个绕法，要求它用内联 CSS 和手工搭的 SVG 图，别用 CDN 那套。这是一个未关闭的 issue。

**它给了我十二个候选项。我在同一场 session 里挨个做完，还是另起一场？**

一个候选项一场 session。要是在一场对话里做完好几个，[context window](https://www.aihero.dev/ai-coding-dictionary/context-window)（上下文窗口）就被报告、追问、领域模型的改动和代码改动一起塞满。报告只是一个临时文件，所以要带走的是那个候选项，不是那个文件。挑一个，把它问透，把决定带进 `/to-spec`。其余的转成 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单），留着以后单独拾起来。把你选中的那项改进写进一份 spec，别直接跳到实现。人们常问这件事，而 Skill 自己没给这一步写下任何流程。

**我该怎么给它下提示词？**

就拿你接下来要构建的东西作提示词。有大构建在前面时，把它指向那份 spec，问"我们怎么让这次改动变轻松？"不给提示词的一次运行，会自己去找热点。用于例行保养这没什么，但给个方向能让报告可以照着动手。

**它在大型遗留代码库上管用吗？**

部分管用。它在那些缺乏一致结构的大型现有代码库上表现不错，也是任何一次性的结构搭建之后推荐的保养工具。但项目已经失控的用户报告说它"起了一点作用，可还是不太够用"。有一位维护八年前遗留代码库的开发人员报告说 model 在里面绕圈子，而同一个 Skill 在一个整洁的仓库上能产出一张清爽的图。那种情况还没有专门的 `/refactor` Skill。如果代码库没有共享词汇，先跑 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 造一套出来，那通常会让这个 Skill 的产出好上很多。

**它跟 `/codebase-design` 有什么不同？**

`/codebase-design` 是一份参考，不是会话驱动器。它提供词汇（module、interface、depth、seam、adapter、leverage、locality），这个 Skill 在用它们。你要把一个空白 agent 指到 `/codebase-design` 当作任务，它会以一种已知的方式失败。它自己没有流程可走，于是 agent 自己编一套，把代码重新探索一遍，跑很长时间才向你问出第一个问题。跑这个 Skill，让它去用那一份。

**它会不会告诉我代码库挺好的？**

很少，动手之前要知道这点。这个 Skill 存在的意义就是输出发现，所以它倾向于给出候选项，而不是得出"什么都没毛病"的结论。用那些强度徽章来校正这一点。如果一份报告里每个候选项都是 `Speculative`，那就是它什么都没找到。

**它在 Codex 或别的 harness 上管用吗？**

部分管用。探索那一步直接点名了 Claude Code 的 `Agent` 工具，带 `subagent_type=Explore`。没有这个工具的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（运行环境）可能会跳过并行探索，而不用自己那套等价物。Skill 照样能跑，但扫描没那么透彻。有人提过一份 harness 中性的重写，还没合并。

**在 TypeScript 里我到底怎么实现 deep module？**

这个 Skill 没带来好答案。人们常常想要一份 `TYPESCRIPT.md`，给这些原则配上具体的文件和模块布局，而它不存在。这个 Skill 告诉你一次加深该落在哪、seam 背后放什么。把它变成一套 package 或目录结构，得你自己来。

## 有效的话应该是这样

- 候选项点的是你领域的概念，不是凭空造的类名："the Order intake module"，不是"the FooBarHandler"。
- 候选项在你最近编辑过的文件里，不在仓库里没人碰的部分。
- 这次运行期间代码一行没改。唯一的新文件是临时目录里那份 HTML 报告。
- 它在报告之后停下，问你想选哪个候选项。它不会自己接着往下走。
- 每张卡片把回报讲成 locality 或 leverage，并说出哪些测试会变得更简单，而不是一句"结构更干净"。
- 当你为一个长久的理由否掉一个候选项时，它主动提出记一份 ADR，让下一次运行不再提它。

## 它在整体中的位置

`improve-codebase-architecture` 是**周期性保养**。你每隔几天跑一次，在任何一条链之外，用来排队待做的活儿，而不是去做它们。它的邻居：

- [codebase-design](https://aihero.dev/skills-codebase-design) 拥有每个候选项都要用的那套 depth 与 seam 词汇。
- [grilling](https://aihero.dev/skills-grilling) 在你选定候选项之后走那棵决策树。
- [domain-modeling](https://aihero.dev/skills-domain-modeling) 在你做出决定的过程中让 `GLOSSARY.md` 和那些 ADR 保持最新。

它的产出是一个想法，回到主构建流程里去，入口是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 或 [to-spec](https://aihero.dev/skills-to-spec)。它在主流程末端的对应项是 [retro](https://aihero.dev/skills-retro)。这个 Skill 改进 agent 工作于其中的代码，而 `retro` 在一次构建之后改进它周围的环境（检查项、标准、引导文件）。某个情况该用哪份 Skill，[ask-matt](https://aihero.dev/skills-ask-matt) 是罩在整个集合之上的路由。
