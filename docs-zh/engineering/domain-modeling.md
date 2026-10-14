## 它做什么

`domain-modeling` 在你做设计的同时，构建并打磨一个项目的 **ubiquitous language**（统一语言）。它会挑出与词汇表冲突的那个词，在你用了含糊词的地方要一个精确的词，并且拿一个具体场景去检验一条关系，直到边界分毫不差。

它是**主动**的那门功夫，不是被动的那门。任何 Skill 都能读 `GLOSSARY.md` 借用它的词汇。这个 Skill 留给你*改动*模型的时候，这就是它会打断你的原因。它在你把某个词定下来的那一刻就把它写进 `GLOSSARY.md`，就写在对话中间，而不是到最后再交出一份整理好的词汇表。最后才写的词汇表是一场 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）的摘要；随写随记的版本才是这场 session 的产出。

## 什么时候找它

输入 `/domain-modeling`，或者当一个任务对得上时，agent 会自己找上它。实际用起来，自动调用是这份 Skill 最弱的一环。当 `grill-with-docs` 或 `wayfinder` 说要加载它时，[models](https://www.aihero.dev/ai-coding-dictionary/model)（模型）常常加载的是 `grilling`，把这一个跳过。如果一场 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 会话跑完了，而 `GLOSSARY.md` 到头来一字未动，就是发生了这件事。跟另一份 Skill 一起按名字调用它。

当*用词*是问题时找它：

| 情况 | 该怎么做 |
| --- | --- |
| 两个人说的 "cancellation" 不是一回事 | `domain-modeling`：定下权威的那个词，把其余的列进 `_Avoid_` |
| "Account" 在三个文件里干着三份活 | `domain-modeling`：把它拆成 Customer 和 User |
| 你刚做出一个难以撤销的架构选择 | `domain-modeling`：这个选择过线的话，它会提议写一份 ADR |
| 模块的*形状*是问题：seam 放哪、interface 多深 | [codebase-design](https://aihero.dev/skills-codebase-design) |
| 你想在动手之前把整个方案审问一遍 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它在底下驱动本 Skill |
| 你只想查一个词，不想改它 | 什么都不用。读 `GLOSSARY.md`。那只是个文件。 |

## 前置条件

事先什么都不需要。这个 Skill 往两个地方写，两处都是懒创建：

- **`GLOSSARY.md`**，在仓库根。你在它上面定下第一个词时，它才被创建。根目录有 `GLOSSARY-MAP.md` 的仓库里，词条改写到地图指向的那份按 context 划分的 `GLOSSARY.md` 里。
- **`docs/adr/`**。第一份过线的 ADR 出现时，这个目录才被创建。

开始之前没有任何东西必须存在，这个 Skill 也不会预先创建任何东西。

## 两份交付物，两条门槛

词汇表和 ADR 的门槛不同。这份 Skill 的大部分麻烦来自把两者混起来。

| | `GLOSSARY.md` | `docs/adr/NNNN-slug.md` |
| --- | --- | --- |
| 装什么 | 词条。一个东西**是什么**，一两句话说清，被淘汰的同义词列在 `_Avoid_` 下 | 一个决定，一到三句话：背景、选择、理由 |
| 写下的门槛 | 一个含糊的词被定成了权威用词 | **三条全中**：难以撤销、缺了背景会让人意外、是真实权衡的结果 |
| 何时写 | 随手写，就在那个词定下的那一刻 | 提议，不擅自代劳 |
| 绝不装 | 实现细节、一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明）、草稿本、通用编程概念 | 这一场 session 里做过的每一个决定的流水账 |

ADR 那三项检验漏掉任何一项，就没有 ADR。轻易可撤销的决定反正会被撤销；不令人意外的决定没人会问；没有真实替代项的那一个，记下来的只是你做了件明摆着的事。

`GLOSSARY.md` 那条规则最要紧，因为实践中偏偏是它被破坏。**它是词汇表，别的什么都不是。** 没人管的时候，models 把"写进 `GLOSSARY.md`"当成把你给出的每个回答都存下来的许可，于是这个文件变成一份持续膨胀的 spec。这是这份 Skill 跨多个 model 都被报告得最多的问题。

## 交叉引用，以及它到哪里为止

当你说出某样东西是怎么运作的，这个 Skill 会去查代码，并指出任何矛盾。*"你的代码取消的是整个 Order，你刚才却说部分取消也是可能的，哪个对？"* 它让语言和代码对上，是公开说出来的，而且发生在它改动任何一边之前。

它交叉引用**代码**、已提交的 `GLOSSARY.md` 和 ADR，此外没有别的。它不去搜你的工单跟踪器。要是你的团队几个月前为一个命名冲突争过、并把它结在了一个已关闭的工单里，这个 Skill 会当成新事再提一遍。[有一个未关闭的请求](https://github.com/mattpocock/skills/issues/717)处理这件事。在那之前，绕法是把这条指示写进你自己的 `docs/agents/domain.md`，这些 Skill 本来就会读那里。

## 常见问题

**我的 `GLOSSARY.md` 有 500 行。1000 行。3000 行。怎么办？**
行数是症状。病因是这个文件把实现细节和压根不该进词汇表的决定都收了进来。修法是一条直接的指示：`/grill-with-docs make my GLOSSARY.md more concise and remove any implementation details from it`。拿它跑一份已经发福的文件，大部分内容会走掉。只有当文件已经清瘦、而它仍然覆盖两个读者不愿同时装下的领域时，才用 `GLOSSARY-MAP.md` 去拆。拆一份臃肿的文件，你只会得到几份臃肿的文件。这个 Skill 在这里的指导还不足以从源头上防住增长，跟踪这一点的 issue 仍然开着。

**为什么它叫 `GLOSSARY.md` 而不是 `GLOSSARY.md`？**
这是整个 Skill 集合里争得最多、也没有定论的命名问题。反对当前名字的理由站得住：既然它是"词汇表，别的什么都不是"，那 `GLOSSARY.md` 已经把话说明白了，而且正如一位读者说的，"跟 ai agent 打交道，一切皆 [context](https://www.aihero.dev/ai-coding-dictionary/context)（上下文）"。支持它的理由是那张地图：`GLOSSARY-MAP.md` 指向若干份 `GLOSSARY.md` 文件，读起来比 `GLOSSARY-MAP.md` 顺，而 `context` 是 DDD 里指模型中一块有界领域的既定用词。至少有一个人纯粹为了改这个文件名维持了一份本地 fork。你也可以这么干，但集合里其余每一份 Skill 都去找 `GLOSSARY.md`，改名意味着把它们全部打上补丁。

**`/ubiquitous-language` 去哪了？**
它被删掉了，而且不是废弃。它的活挪进了 `domain-modeling`，后者持续维护整个模型，而不是从一次对话里倒出一份词汇表。词汇约束现在更重要，不是更不重要。它跑在 grilling、triage 和 mapping 底下，不再是一道你得记得单独走一遍的程序。

**一个原本没有词汇表的代码库，我怎么给它建一份？**
明确开口要，别等它自己攒起来。`/grill-with-docs help me scaffold my existing repo with a GLOSSARY.md` 是文档写明的路子。预期是一轮很长的审问。有位用户报告文件成形之前问了 50 多个问题。在棕地仓库上顺手攒，词汇表建得太慢。

**我保留这份领域模型，同时用我自己的 ADR 格式，行吗？**
今天做不到干净。词汇表那一半和 ADR 那一半装在同一份 Skill 里，所以一套已经有既定 ADR 约定（不同模板、不同位置、不同命名）的团队，会拿到与自己的风格互相冲突的指示。眼下两个选择：把这份 Skill 拷到本地改，或者在你仓库自己的 agent 文档里覆盖 ADR 约定。把两半拆开是[一个未关闭的请求](https://github.com/mattpocock/skills/issues/557)。

**一份词汇表真的值回它的成本吗？它又多出一份要 review 的交付物，而且会变陈旧。**
有时候确实不值。DDD 越靠近实现就越没用。回报在偏上游的地方，在命名和概念对齐上，不在聚合和分层的仪式上。同义词控制在你命名的那些边界上才要紧：模块名、表名、状态枚举、工单标题、CLI 命令。在普通行文里它远没有那么要紧。还有一种反对意见：领域词压缩的是*已经共享这些词的人之间*的沟通，而 agent 对一句大白话描述会给出同样的回应。按这种读法，词汇表的价值在于让你和你的 reviewer 跟 agent 在做的事保持一致，而不是让 agent 本身更强。一次一天的构建，跳过它。还有一份没人 review、由 agent 写成的词汇表比没有更糟：它塞满了自信的说法，后续的 session 会把它们当作事实。

**它能把我含糊的提示自动转成领域语言吗？**
不能，也没有计划做一份这样的 Skill。一份你自己都不理解的领域语言，写下来就毫无意义。这个 Skill 在你已经理解之后负责把精确执行下去，它不会替你造出你本来没有的词汇。相关的陷阱是只借用领域词、不做建模。对的名词堆在错的观念之上，产出的东西读着对、其实不对。

## 有效的话应该是这样

- 它会打断你的话，问你指的是两者中哪一个，而不是自己挑一个接着走。
- `GLOSSARY.md` 在对话**过程中**就在改，不是到最后才集中改一波。
- 它拒绝为你明天就能撤销的事情写 ADR，并且说出三项检验里哪一项没过。
- 新词条用一两句话说清一个东西*是什么*，并在 `_Avoid_` 下点明你放弃的那些词。
- 当你的代码和你的说法不一致时，它会把你的代码引用回来给你看。
- `GLOSSARY.md` 变短的次数和变长的次数一样多。

## 它在整体中的位置

`domain-modeling` 是一份 **由 model 调用的参考**，它跑在别的 Skill *底下*的时候比单独跑的时候多。[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 在一场 grilling 会话里驱动它，[wayfinder](https://aihero.dev/skills-wayfinder) 在绘制地图时加载它，[triage](https://aihero.dev/skills-triage) 用它把 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）保持在这个项目自己的用词里，[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在决定落定时调用它。血缘最近的同胞是 [codebase-design](https://aihero.dev/skills-codebase-design)。两个一起构成垫在其余一切底下的词汇层，这一个管*领域*，那一个管模块的*形状*。它也可以直接调用，当你想要这门功夫、又不想承诺那份本来会加载它的 Skill 的那些步骤时。拿不准哪个 Skill 合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
