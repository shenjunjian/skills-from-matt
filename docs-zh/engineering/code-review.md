## 它做什么

`code-review` 沿两条轴审查 `HEAD` 与你指定的某个固定点（一次 commit、一个分支、一个 tag、`main`、`HEAD~5`）之间的 diff。**Standards**（规范）这条轴问的是代码有没有照本仓库的写法来写。**Spec** 这条轴问的是代码有没有做到最初那张工单或 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明）要求的事。两条轴各自跑在自己的 [sub-agent](https://www.aihero.dev/ai-coding-dictionary/subagent)（子代理）里，谁也看不到另一个的推理过程。

这个 Skill 从不合并或重排两条轴的结果。报告以*每条轴各自*最严重的问题收尾，拒绝在两条轴之间点名一个总冠军。一次改动可以过一条轴、栽在另一条上。代码遵守了每一条约定、却实现了错的东西，那就过 Standards、挂 Spec。代码恰好做了 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）要求的事、却破坏了仓库的约定，那就反过来。混在一起的结论会让过关的那条轴把挂掉的那条遮住。

## 什么时候找它

输入 `/code-review`，或者当你要求审查一个分支、一个 PR、进行中的工作、或者任何"从 X 以来"的东西时，agent 会自己找上它。

| 你的情况 | 该找谁 |
| --- | --- |
| diff 已经有了，你想知道它是不是做得对*并且*做的是对的东西 | `code-review` |
| 你想在 diff 里搜 bug：空值路径、竞态、off-by-one | Claude Code 自带的内置 review，不是这个（见下面的重名冲突） |
| 还没写任何东西，你想以测试先行把它写出来 | [tdd](https://aihero.dev/skills-tdd) |
| 一整份 spec 需要构建出来，包含 review | [implement](https://aihero.dev/skills-implement)，它自己会调用本 Skill |
| 整个代码库都偏了，不是一个 diff 的事 | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) |
| 有东西坏了，你不知道为什么坏了 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

固定点必须由你给出。你不给，这个 Skill 会开口问，而不是猜。在派生任何东西之前，它会检查那个 ref 能否解析、diff 是否非空，所以分支名打错了会当场在你面前失败，而不是钻进两个 sub-agent 里面才炸。

## 前置条件

Standards 这条轴不需要任何东西。它读仓库里写了规范的那些文件（`CODING_STANDARDS.md`、`CONTRIBUTING.md` 之类），仓库什么都没写时退回内置基线。

Spec 这条轴需要一份 spec 存在并且能被找到。它按这个顺序找：

1. commit message 里的工单引用（`#123`、`Closes #45`、GitLab 的 `!67`），通过 tracker 文档取回。
2. 你作为参数传进去的一条路径。
3. `docs/`、`specs/` 或 `.scratch/` 下与分支名或功能名匹配的 spec 文件。
4. 开口问你。

第 1 步依赖 tracker 文档，那是 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 写出来的。没有那份文档时，只要你递给它一条路径，这条轴照样能用。完全没有 spec 时，这个 Skill 跳过 Spec sub-agent，报告里写"no spec available"，而不是自己编需求。

## 两条轴

| | Standards | Spec |
| --- | --- | --- |
| 问题 | 做得对吗？ | 做的是对的东西吗？ |
| 读取 | 仓库写下的规范，加上坏味道基线 | 最初那张工单或 spec |
| 报告 | 违反写下的规范（可以是硬性判定），以及坏味道（一律是主观判断） | 缺失或只做了一半的需求、范围蔓延、需求实现错了 |
| 每条发现都引用 | 规范文件和那条规则，或者具名的坏味道加上对应的 hunk | spec 里的某一行 |

这套设计是为了避开那种不了解你的规范的通用 review Skill。这种 Skill 会把你代码库里有意为之的地方标出来，却漏掉你代码库赖以成立的不变量。所以仓库自己的文档才是 Standards 这条轴上的 [primary source](https://www.aihero.dev/ai-coding-dictionary/primary-source)（一手来源），而且**仓库总是覆盖一切**。

**坏味道基线**排在仓库规范之下。它是 Fowler 的 _Refactoring_ 第 3 章里的十二种代码坏味道：Mysterious Name、Duplicated Code、Feature Envy、Data Clumps、Primitive Obsession、Repeated Switches、Shotgun Surgery、Divergent Change、Speculative Generality、Message Chains、Middle Man、Refused Bequest。每一种都是一个带标签的启发式判断（"可能是 Feature Envy"），从来不算硬性违规。每一种都说明这个坏味道是什么、怎么修，所以每条发现都附带着修法，不只是一句抱怨。两条轴都会跳过你的 linter 已经在强制的内容。

## 常见问题

**它和 Claude Code 自己的 `/code-review` 撞名了。我该怎么办？**

这是这个 Skill 被报告最多的问题，还没修。Claude Code 自带一个 `/code-review`，干的是另一件事：它在 diff 里搜 bug，而这个 Skill 检查 spec 符合度和仓库规范。装这套库时，两者里有一个会赢，哪个赢取决于你怎么装的：

- **插件 marketplace。** 每份 Skill 都带上 `mattpocock-skills:` 前缀，内置那个在无前缀的名字下就几乎够不着了。
- **普通 skills 安装。** 本地文件赢，这个 Skill 把内置的遮蔽掉。

一种答案是把 Claude Code 自带的 skills 整个删掉。这省下大量 [context](https://www.aihero.dev/ai-coding-dictionary/context)（上下文），撞名也就不再是要紧事。遮蔽本身大概算 Claude Code 的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（运行环境）bug（Skill 作者本该能给自己的 Skill 起任何名字），所以另一种答案是给本地副本改名。对 frontmatter 的改动或者改了名的目录，`npx skills update` 都会给你撤销回去。用户们说得住的绕法是把这个 Skill fork 成一个新名字，并把 `code-review` 从受管理的集合里摘掉。记一份你从哪个 commit fork 的，好让你日后手工同步。

**它的 sub-agent 不停地在再次调用 `/code-review`，又派出更多 agent。**

这是已知的未修复 bug。好几个人复现过，涉及不止一种 harness。Standards 和 Spec 的提示里没有禁止转派，所以 sub-agent 能重新找到这个 Skill 再一次铺开。有一份报告里 agent 数量超过了 50。人们在 fork 上打的补丁是给两份 sub-agent 的任务书各追加一行："不要调用 `/code-review`，也不要派生额外的 agent：直接完成这次审查。"也有人更愿意在 harness 层面处理，这样每份 Skill 都继承这条护栏。这两样都还没进入出厂的那份 Skill。要是你让它无人值守地跑，盯住 agent 计数。

**我应该在写代码的那场 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）里跑它吗？**

另开一场更好。有位读者说过：同一份上下文自己审自己不算 review，那是一个带 slash command 的确认偏误。在写作代码的那场 session 里审查的 agent，它的上下文里装着塑造这段代码的每一个假设。一个独立的审查者不会有那份上下文。这也是为什么人们要求 [implement](https://aihero.dev/skills-implement) 去掉它内置的 review 步骤，因为那一步正在刚写完这个 diff 的 session 内部执行审查。独立的做法是你自己从一场干净的 session 里调用 `/code-review`。

**每张工单跑一次，还是最后跑一次？**

都行，这个 Skill 不替你做决定。按工单跑能让每个 diff 小到 Spec 这条轴有一份清楚的 spec 可对，这正是 `implement` 用的模式。攒到分支末尾能抓到工单之间的相互影响，而按工单的那些遍次各自都会漏掉它。拿不准的话，按工单审查，最后再对着分支起点跑一遍收尾。

**它的发现我能信吗？**

不核对就不能信。sub-agent 的输出是一个假设，不是证据。有一个团队报告说，基于文字描述的审查漏掉了十几个破坏性变更，后来才被发现。这个 Skill 把两份报告照原样合在一起，或者略作清理。它不会逐条拿文件去复核每个说法，所以一条发现可能引用了错误的位置，或者夸大了影响。先读每条发现上的引用，再照着动手。这个 Skill 要求每条发现都带着引用（规范里的一条规则、一个坏味道加上它的 hunk，或者 spec 的一行），正是这一点让发现变得可以核查。

**为什么我每跑一次它都能找到新问题？**

每次修复都往要审的代码里添新东西，而 Standards 这条轴里靠主观判断的那一半每次跑出来的结果也不同。有位读者描述了这个循环："/code-review 和 /improve-code-architecture 每跑一次总能找到新东西。我实现修复，重跑这些 Skill，一遍又一遍。"这里没有收敛保证。把一趟审查当成一份线索清单。照着那些背后有明确规则引用的动手，然后收手。别循环跑到它报告干净为止，因为它永远不会干净。

**它会审查我未提交的改动吗？**

不会。它做 `<fixed-point>...HEAD` 的 diff。三点写法从 merge-base 起算，并且排除已暂存的改动和工作区的改动。如果 `implement` 还没打中间提交，审查就看不见正要进下一个 commit 的那部分工作。先提交，再审查，之后 amend 或者加一个 fixup。

## 有效的话应该是这样

- ref 不对或 diff 为空时，它在派生任何 sub-agent 之前就拒绝开始。
- 报告以 `## Standards` 和 `## Spec` 下的两个独立段落交回，不是一份合并清单。
- 每条 Standards 发现要么点名你某个仓库文件里的一条规则，要么点名十二种坏味道之一，并引用对应的 hunk；每条 Spec 发现都引用 spec 的一行。
- 结尾的摘要是每条轴各给出最严重的一个问题，拒绝挑一个总胜者。
- 没有可用的 spec 时，Spec 段落直说这一点，而不是列出它从代码里推断出来的需求。

## 它在整体中的位置

`code-review` 是构建链靠尾部的审查步骤：`grill-with-docs → to-spec → to-tickets → implement → code-review → retro`。你把它指向任何分支或 PR，它也能单独跑。

- [implement](https://aihero.dev/skills-implement) 是离它最近的邻居。它主导构建，并在提交之前把本 Skill 当作自己的收尾审查调用一次。[implement-spec](https://aihero.dev/skills-implement-spec) 也是同样做一次，覆盖整条集成分支。
- [retro](https://aihero.dev/skills-retro) 排在它后面并且给它调参。当一场 session 显出审查漏掉了某一类错误，`retro` 会提出那条检查，或者提出 Standards 这条轴随后会读的那条 `CODING_STANDARDS.md` 规则。
- [pr](https://aihero.dev/skills-pr) 在被审的工作推上去之后撰写 pull request 正文。
- [to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 产出 Spec 这条轴用来对照的那份文档，所以含糊的 spec 会让那条轴也跟着含糊。
- [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是覆盖整个代码库的对应物，因为这个 Skill 只看一个 diff。

拿不准这种情况该用哪个 Skill 时，[ask-matt](https://aihero.dev/skills-ask-matt) 会在整个集合里为你路由。
