## 它做什么

`implement` 构建已经定好的工作。你把它指向一张 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）、一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明），或者你刚在对话里商定的那个计划，它就写代码、在 seam（接缝）上驱动 [tdd](https://aihero.dev/skills-tdd)、边写边做类型检查、结束时跑 [code-review](https://aihero.dev/skills-code-review)，并提交到当前分支。

它从不重开计划。没有提问，没有澄清环节，也不会提议换一个做法。上游定下来的一切就是输入，这个 Skill 全部的事就是把它变成一个 commit。这就是它和对着一个空白 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（代理）敲"把它做出来"的区别，后者常常在构建的过程中重新设计这项工作。

## 什么时候找它

这个要你自己输入 `/implement`，agent 不会自己去找它。它带着 `disable-model-invocation: true` 发布，所以别的 Skill 也调不动它。凡 [ask-matt](https://aihero.dev/skills-ask-matt) 或 [to-tickets](https://aihero.dev/skills-to-tickets) 说"然后每张工单跑 `/implement`"的地方，那都是对你下的指令，agent 不会自己去做。

工作当前存放在哪，决定了这个 Skill 对不对：

| 这份工作…… | 该找谁 |
| --- | --- |
| 是跟踪器上的一张工单 | `/implement #42`，每个 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）做一张工单，工单之间 [clearing](https://www.aihero.dev/ai-coding-dictionary/clearing)（清空）上下文 |
| 是一份 spec，还没拆，而构建跨越多个 session | 先 [to-tickets](https://aihero.dev/skills-to-tickets)，再每张工单跑 `/implement` |
| 是一份 spec，而构建很小 | 直接对着 spec 跑 `/implement` |
| 只存在于你刚进行的那场对话里，而且它还小 | 就在那同一个窗口里跑 `/implement` |
| 还没写在任何地方 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)；没有代码库就用 [grill-me](https://aihero.dev/skills-grill-me) |
| 是一个具体行为，你想测试先行，没有 spec | 直接用 [tdd](https://aihero.dev/skills-tdd) |
| 已经构建好了，你想让人查一查 | 直接用 [code-review](https://aihero.dev/skills-code-review) |

同一场 session 这种情形值得单独点出来，因为 Skill 自己的第一行没覆盖它。`SKILL.md` 写的是"spec 或工单"，这会把 [model](https://www.aihero.dev/ai-coding-dictionary/model)（模型）推向去找一个并不存在的文件。如果计划只活在对话里，调用它的时候把这点说清楚。

## 前置条件

`implement` 提交到你在的那条分支。它不创建分支，也不问。开工前先确认你在自己想让这份工作落到的那条分支上。

如果工单来自 [to-tickets](https://aihero.dev/skills-to-tickets)，那么 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 已经把它们所居住的跟踪器配置好了。`code-review` 读同一份配置，用来在收尾时找到出处那份 spec。

## 一次运行做什么

一次运行有五步，按顺序：

1. 读工单或 spec，琢磨出 seam。
2. 在事先商定的 seam 上驱动 [tdd](https://aihero.dev/skills-tdd)，一次一个 red-green 切片。
3. 频繁做类型检查，边做边跑单个测试文件。
4. 最后跑一次完整测试套件。
5. 跑 [code-review](https://aihero.dev/skills-code-review)，然后提交到当前分支。

一次运行覆盖一张工单。[to-tickets](https://aihero.dev/skills-to-tickets) 产出的工单是 tracer-bullet 式的 vertical slice（垂直切片），大小按一个空白 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)（上下文窗口）来裁，所以设想的节奏是：清空上下文，实现一张工单，提交，再清空。每张工单自包含，所以上一张工单的上下文可以丢掉。

## 事先商定的 seam

这个 Skill 的核心概念是 **seam**，也就是你观察行为的那个公开边界，不必伸进去。测试住在 seam 上。seam 在任何代码存在之前就商定好，测试就活得久，你可以在下面重写实现而不动它们。

"事先商定"这部分很要紧，同时也是这个 Skill 最薄弱的一环。`implement` 内部没有任何东西去商定 seam。`tdd` 才是那个会提问的 Skill，它拒绝在一个未确认的 seam 上写测试。所以实际上，商定这件事要么发生在上游的 spec 里，要么发生在一次运行的头一个来回里。如果哪儿都没发生，这次运行就变成"只管写代码"，而且没有任何东西会警告你。在 spec 里把 seam 名字写出来，就是拦住这件事的办法。

## 常见问题

**它跑完了，可我的工单还是打开状态，验收标准也还没勾。**

正确，也在预期之内。`implement` 没有收尾这一步。它结束于 commit，从不碰那张工作项。在 GitHub Issues 上和本地 markdown 跟踪器上都是这样，所以这不是跟踪器集成的问题。它也不处理 `code-review` 给出的那些发现，不去勾出处那张 issue 上的 `- [ ]` 方框。工单自己去关，标准自己去核对。这一点在依赖链上最要紧，因为 `to-tickets` 把前沿定义成所有阻塞项都已关闭的那些工单。什么都不关闭，就没有任何东西看起来解除了阻塞。

**能一次把它指向我所有工单吗，或者并行跑几个？**

`/implement` 不行：一次调用，一张工单。要在一次运行里做完整份 spec，用 [implement-spec](https://aihero.dev/skills-implement-spec)，它把已就绪的前沿上的每张工单交给一个在自己 worktree 里的 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent)（子代理），再把结果合到同一条集成分支上。在同一个检出目录里并排跑好几个 `/implement` session，比"不支持"更糟。有一份现场报告描述：一个 session 里的 `git commit --amend` 落到了另一个 session 的 commit 上，一个 stash 从 `refs/stash` 上消失，还有 commit 落错分支，全部发生在一下午的三张 issue 之间。这些 session 共用一个工作目录、一个 index、一个 HEAD。人们用 git worktree 绕开这一点，但 `refs/stash` 在所有 worktree 之间也是共用的，所以只靠 worktree 修不好 stash 那一条。

**它能开一个 pull request 而不是提交吗？**

没有内建。它直接提交到当前分支。好几个人觉得这样太急，因为代码在他们验证它能工作之前就已经落地了。没有配置开关，也没有 PR 模式。人们在调用时覆盖它（"提交到一个分支并开一个 PR"），或者改自己本地那份 Skill。当 agent 确实要写 PR 时，[pr](https://aihero.dev/skills-pr) 负责它的正文形状。

**`code-review` 说它看不见我的改动。**

`code-review` 审的是 `git diff <fixed-point>...HEAD`，这个范围不含已暂存的改动和工作区改动。`implement` 在提交之前跑它，所以除非中间已经有一个 commit，那份 diff 里根本没有可审的东西。多个人报告过这一点，两边都还没修。先提交，再对着你分叉出去的那个点做评审。

另外，有些人才不想把评审放在这次运行里：一个 agent 评审它自己刚写的代码，会偏向自己的方案。在一个空白 session 里对着一个固定点跑 [code-review](https://aihero.dev/skills-code-review) 是个成立的替代做法。正是这同一份偏向，让那份 Skill 把它的两条轴放进各自独立的 sub-agent 里跑。

**一张工单烧掉 150k tokens。是我用得不对吗？**

多半不是。更可能是工单太大。一次运行要做代码库探索、每个 seam 一轮 red-green、一整套测试，还有一次评审，所以一张有些分量的工单超过 100k [tokens](https://www.aihero.dev/ai-coding-dictionary/token)（词元）是正常现象，不是哪里坏了的信号。修法在上游。在 [to-tickets](https://aihero.dev/skills-to-tickets) 里把工单尺寸定合，让每张装得进一个空白窗口。要是单张工单一直超，就拆它，而不是去调高 [effort](https://www.aihero.dev/ai-coding-dictionary/effort)（推理投入）档位。

**在空白 session 里跑 `/implement #2`，它做的是一件完全不相干的东西。**

agent 把 `#2` 解析到了上下文里另一份带编号的清单上，比如一个 todo 文件或检查表，而不是配置好的那个跟踪器。`implement` 现在会从工单跟踪器抓取你给的引用，并在开工前说出它的标题；引用有歧义时会来问你。核对那个标题是不是你要的工单；给 issue 的 URL 或者 `owner/repo#2` 就能彻底消除歧义。

## 有效的话应该是这样

- session 一开头去读工单或 spec，并复述它将要构建什么，而不是来问你要构建什么。
- 你能在 trace 里看到一次真实的 `tdd` Skill 工具调用，而不只是 diff 里冒出来的测试。
- 类型检查和单个测试文件在这次运行中反复跑，完整套件在接近末尾时跑一次。
- 这次运行走到当前分支上的一个 commit，不需要你催促它继续。
- diff 是一张工单份量的改动：一条穿过每一层的垂直切片，不是好几张工单扫在一起。

## 它在整体中的位置

`implement` 是主链里的构建环节：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

它的邻居是 [to-tickets](https://aihero.dev/skills-to-tickets)，它产出这个 Skill 消费的工单，并声明决定工单顺序的那些阻塞边；[tdd](https://aihero.dev/skills-tdd)，它在每个 seam 内部驱动的那份；以及 [code-review](https://aihero.dev/skills-code-review)，它在提交前跑的那份。它坐落在那些规划类 Skill 的下游，并且信任它们。它不重新校验交给它的东西的形状，所以一张结构很差的地图、一个按技术层横切的工单，都会照着原样被构建出来。

正是这份信任，让 [wayfinder](https://aihero.dev/skills-wayfinder) 在 [to-spec](https://aihero.dev/skills-to-spec) 处汇入这条链，而不是把自己的地图直接循环送进 `implement`。只有当这桩工作原来很小，才可以从地图直接去 `implement`。

拿不准自己走的是哪条 flow 时，[ask-matt](https://aihero.dev/skills-ask-matt) 是罩在整个集合之上的路由。
