## 它做什么

`writing-for-agents` 是撰写面向 agent 的文档时的参考手册：一个 Skill、一份 `AGENTS.md` 或 `CLAUDE.md`、一个 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明）、一段运行时 prompt、一个 README，任何需要 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 去读的文档。格式各有不同，写法并无不同。同样的几根杠杆让每一份都变得可预期，于是 agent 每次运行都遵循同一套*流程*（不一定得到同一份输出）。

它首选的修法是删除，不是解释。让一个 agent 替另一个 agent 写指令，它会把大半篇幅花在解释[模型](https://www.aihero.dev/ai-coding-dictionary/model) 本来就懂的东西上。这类行每一行都是 **no-op**（空操作）：它消耗 [context](https://www.aihero.dev/ai-coding-dictionary/context)（上下文），却不改变任何行为。这份参考帮你把它们找出来，所以它对一份你已经写好的文档，和对一个空白文件一样有用。

v1.1 之前它叫 `writing-great-skills`。新名字更贴合它一直以来的样子：其中几乎没有哪一部分是 Skill 专属的。只属于 Skill 的那些机制（frontmatter、模型调用还是用户调用的取舍、router Skill）放在一个链接出去的 `SKILL-MECHANICS.md` 里，只有当你面前这份文档是一个 Skill 时才去读它。

## 什么时候找它

输入 `/writing-for-agents`；或者当你在创建、编辑一个 Skill，在修改 `AGENTS.md` 或 `CLAUDE.md` 时，agent 会自己找它。

其余所有交给 agent 读的东西，都手动去找它：你的文档、specs 和 [工单](https://www.aihero.dev/ai-coding-dictionary/ticket)，系统 prompt 和 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) prompt。判断标准就一个问题：agent 会读这份东西吗？agent 怎么拿到这份文档并不重要：可能是一个指针指向它，可能是人把它粘进去，也可能它就在仓库里。要弄清一个代码库里有什么，用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。这份参考管的是文档读起来怎么样，不管它知道什么。

## 两种负载

核心思想是两种预算，每一份文档、每一个指针都在花它们：

- **Context load**（上下文负载）：常驻材料占用 agent 窗口的代价：一行 `AGENTS.md`、一段 Skill 描述、任何每一 [turn](https://www.aihero.dev/ai-coding-dictionary/turn)（轮次）都躺在上下文里的东西，不管它这一轮有没有被触发。
- **Cognitive load**（认知负载）：你记住有哪些文档存在、什么时候该找哪一份，这份代价由你付。你就是那个索引。别想着把这项成本压到最小，因为它是人的自主权的价格。

一旦你按这两种负载来思考，大多数写作决策（拆还是不拆、内联还是折叠披露、指过去还是推过去）就都变成同一个权衡，只是发生在不同的地方。

## 几根杠杆

- **[Context pointers](https://www.aihero.dev/ai-coding-dictionary/context-pointer)**（上下文指针）：留在上下文里的一条参考，它命名了上下文之外的材料，并编码了什么时候该去取。一段 Skill 描述和一行提到某份文档的 `AGENTS.md`，是同一种东西。决定 agent 多大程度上照着做的，是指针的*措辞*，不是它的目标。
- **Information hierarchy**（信息层级）：从文内步骤，到文内参考，再到藏在指针背后、需要时才披露的参考，这是一级级往下的梯子。**[Progressive disclosure](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)**（渐进式披露）就是把材料往下挪，让顶部保持好读。
- **Completion criteria**（完成标准）：每一步的"做完了"条件有多清晰、多苛刻，以及这份苛刻带来的 **legwork**（跑腿功夫）。它们是对抗 **premature completion**（过早收工）的防线。
- **Leading words**（引导词）：一个已经存在于模型预训练里的紧凑概念（*tight*、*red*、*tracer bullet*），agent 在执行这份文档时就拿它来思考。它在两处起作用：在正文里引导执行，在指针里触发调用。
- **Pruning**（修剪）：单一事实来源、相关性，外加逐句施加的 no-op 检验，用来对付 **duplication**（重复）、**sediment**（沉积）和 **sprawl**（蔓生）。

## 常见问题

**`/writing-great-skills` 去哪了？**
它就是现在这个 Skill，在 v1.1 改了名。早在改名之前，用户就已经把它用在 `AGENTS.md`、文档、spec、工单和运行时 prompt 上了。结构、引导词、修剪适用于任何 agent 要读的文本。没有别名。按新名字重装一次。

**"Writing for agents"：所以是让 agent 来写？**
正相反。你是作者，agent 是读者。这才是难的地方。你写给的读者已经读过一切，所以解释是浪费，精确就是全部的工作。

**我就不能让 agent 替我写吗？**
能，它会写出一坨啰嗦的东西。放着不管，模型会去解释它已经知道的东西，也不会自己做 no-op 检验，更不会自己想出去找引导词。用这份参考审一遍它写的草稿，这份参考大半价值就在这一遍审查上。

**我让一个 agent 精简一份文档，它把功能砍掉了。**
被要求"精简"的 agent 会拿长度当优化目标，因为长度是它看得见的那个东西。no-op 检验查的是行为，不是风格：删掉那一行，然后问 agent 的行为变没变。一句话不合格就整句删掉，而不是从里面抠字；对某句话有分歧，跑一遍文档来定，不要靠争论。

**怎么知道它写完了？**
它能用，而且你再也找不到重复、沉积或 no-op。没有自动化评测。你要靠手动跑一遍这份文档来检查，用失败模式的词汇去诊断问题。文档行为不对时，同一套词汇也能帮你修它：先说出失败模式，再修那个模式。

**这个该放进 `CLAUDE.md` 还是别处？**
问你想付哪种负载。`CLAUDE.md` 无条件载入每一个 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）。藏在指针背后的材料，在它被触发之前，只付指针自己那一行的代价。十次会话里只用得上一次的东西，在另外九次里都在花上下文负载。

**每出一个新模型，我都得重写文档吗？**
大体不用，而且过度贴合某一个模型会带来它自己的问题。为新模型做更新，通常又是一遍 no-op 修剪，不是重写。

**我的 Skill 只在我拿来建它的那个具体任务上管用。**
常见的那个做法（把活干一遍，然后让 agent 把它写成 Skill）贴合那一次运行贴得太紧，例子也就写得太具体。把那次运行留作证据，然后刻意做泛化：剥掉属于那个仓库、那些文件的东西，为那一类任务而写。

**英语不是我的第一语言，我会失去引导词的优势吗？**
不会。在最少 [tokens](https://www.aihero.dev/ai-coding-dictionary/token)（词元）里塞进最多行为的那个词，这份参考会替你找出来。

## 起作用的表现

- 文档越写越好，也就越写越短，剩下的少到让你吃惊。
- 你能指着一个引导词，看到它在不只一处改变了 agent 的行为。
- 没有任何东西被说了两遍，以任何形式。重复是一份文档从没被测过的最可靠的信号。
- 只有一条分支才需要的参考放在指针背后，而不是放在主文件里。

## 它的位置

这是一份随时可以取的独立参考。它作用于整套 Skill，而不只是某一个。这里每个 Skill 都是照着它写的；它同样覆盖别的 Skill 产出的那些文档（一份 `GLOSSARY.md` 和它的 ADR、一个 spec、一张工单），只要 agent 得去读它们。唯一一个直接调用它的是 [retro](https://aihero.dev/skills-retro)，它在提出任何指引文件或 Skill 之前会先载入这份参考。拿不准哪个 Skill、哪条流程适合眼前这个活时，[ask-matt](https://aihero.dev/skills-ask-matt) 会带你把整套过一遍。
