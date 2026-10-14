## 它做什么

`pr` 定义一个 pull request 正文的形状：一个展示这次改动的 **Summary**（概述），证明它确实能工作的 **Evidence**（证据），以及对合入风险高低的 **Merge Danger**（合并风险）判断。它是一份格式参考，不是一个工作流。它不推送分支、不开 PR、也不决定里面放什么。它告诉 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（代理）：当它要写 PR 正文时，那个正文该长什么样。

概述是一个可视化，不是一段散文。默认的 PR 正文用文字描述 diff。这一份挑那个能让关键论点讲清楚的**最小视图**（伪代码、一棵调用树、一棵组件树、一个文件树、一张 Mermaid 图，或者一份整形过的 diff），并把围着它的话说得简短。评审的人本来就开着那份 diff，所以正文要在他们读它之前先把形状摆给他们看。

## 什么时候找它

输入 `/pr`，或者每当 agent 要写一份 PR 正文时，它自己就会找上这个 Skill。

| 你的情况 | 该找谁 |
| --- | --- |
| 一个分支已经就绪，需要一份评审的人能扫读的正文 | `pr` |
| 代码写完了，但还没人评审它 | 先 [code-review](https://aihero.dev/skills-code-review)，再 `pr` |
| PR 已经开着，评审意见正陆续回来 | 这套里还没有对应的东西；`pr` 只写正文 |

## 模板

三个部分，按这个顺序：

- **Summary**：一个或多个小可视化，每一个都紧挨着它支撑的那段短文放。用一个，有时用几个，很少全部都用。只留下评审的人需要的那些调用、文件、props 和边界。
- **Evidence**：一个前，一个后。当改动是视觉性的、而 [environment](https://www.aihero.dev/ai-coding-dictionary/environment)（环境）能截图时，一张截图是最有力的证据。否则就用那个先前失败、现在通过的具体测试，写成伪代码，或者用那段发生了变化的控制台输出。
- **Merge Danger**：这次改动是 **one-way door**（单向门）还是 **two-way door**（双向门），以及它的 **blast radius**（波及范围）。双向门撤销起来便宜；单向门（一次破坏性迁移、一次公开 API 移除、一个难以回退的决定）不是。波及范围点名的是改动错了会坏掉的东西：布局位移、某个 API 的消费方、移动端的自适应。

门的判断是这份模板里最主导的想法。它把"这个合入安全吗"从一种直觉变成一个写下来说的断言，评审的人可以不同意。它也告诉对方该把 [human review](https://www.aihero.dev/ai-coding-dictionary/human-review)（人工评审）花在哪儿。一道波及范围小的双向门就快速扫过，一道单向门就慢慢读。

## 常见问题

**agent 自己给出的那个门判断，我能信吗？**

不能盲信，而把它写下来正是为了这点。写下这次改动的 agent 就是给它打分的那个 agent，所以它说"双向门，波及范围小"的时候你要最多疑。diff 也常常把一次改动能不能回退这件事藏起来。有一位用户说过，"回滚一个 commit 并不能把一批已经发出去的邮件收回来"；一次带开关的灰度发布，也只是在第一条按新格式写入的数据落地之前还算双向门。这个 Skill 给 agent 的是一个定义（破坏性动作和难以回退的决定算单向门），不是一份检查清单，所以 Merge Danger 那一行你要最仔细地核对。有两件事有帮助：确认 agent 面前摆着那张 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）或那份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)（规格说明），而不只是 diff；以及把你仓库里永远按单向门对待的那些改动（schema 迁移、任何对外发布或删除的东西）写在 agent 做判断前会读到的地方。

**它替我把 PR 开好吗？**

不开。`pr` 只管正文。[implement](https://aihero.dev/skills-implement) 结束时提交到当前分支。要一个开 PR 的 Skill 或选项（一个 `/to-pr`，或者让 `implement` 开 PR 而不是提交）的请求，仍然只是未落地的提案。一位用户的绕法是加一句覆盖本地 `implement` 的指令，叫它去开一个 PR。[implement-spec](https://aihero.dev/skills-implement-spec) 是例外：当你的工单跟踪器靠 PR 结活儿，或者你要了一个，它会开一个标记为草稿的 PR。因为 `pr` 由 model 调用，所以你每次让它开 PR，它都会用这个形状来写正文。

**它不还是给我产出一大堆文字和一堆图吗？**

这个 Skill 就是为挡住这件事而设计的，但它仍然可能发生。用户对 agent 写的 PR 正文的抱怨很一致："一段很长的概述，可我需要知道的就是改了什么、怎么验证的、会坏什么、合入安全不安全。"这个 Skill 告诉 agent 跳过开场白、散文写短、挑最小的那个视图，通常一个可视化，很少全部都要。如果你仍然拿到一摞图，那是 agent 无视了这条。如果正文很大是因为 diff 很大，问题在 PR 的体量上，`pr` 不会替你把它拆开。

**我的仓库已经有一份 PR 模板。哪一份说了算？**

默认都不。`pr` 有自己的模板，也不去找 `.github/pull_request_template.md` 或任何类似的东西。好几位用户要求这个 Skill 使用仓库的模板。如果你什么都不做，agent 手上就有两条互相冲突的指令指向同一份文档。这件事在你仓库的 agent 文档里定下来，比如把仓库模板填好，把 Summary、Evidence 和 Merge Danger 放到它下面。

**产出是 HTML 吗？为什么那张 Mermaid 图不渲染？**

产出是一份 markdown 的 PR 正文，不是一个 HTML 页面。GitHub 和 GitLab 会在 PR 描述里渲染 Mermaid 块，终端不会，所以在 agent 还在本地给你看的时候，那张图看起来就是原始文本。用 CLI harness（运行环境）的用户靠一个 ASCII 的 Mermaid 渲染器绕开这一点，或者让 agent 往 PR 上附一份 HTML 版本。Mermaid 只是六个视图里的一个。一棵调用树、一个文件树或一份整形过的 diff，在哪儿读出来都一样。

**它能替我筛掉评审回来的那些意见吗？**

不能。它写完正文就停。用户不止一次要求过一种对来自其他开发者或评审机器人的意见做分诊的办法（哪几条值得动手，哪几条不是问题）。这个 Skill 不做这件事。

**PR 在变的时候，它会让正文跟着更新吗？**

不会。它在一个时间点上写下正文，而一个在评审期间发生变化的 PR 会让那份正文过期。改动很大之后，让 agent 重写一次正文。agent 又是在写 PR 正文了，所以它会用同一个形状。

**我的改动没有 UI。Evidence 里放什么？**

除了截图，其余全放。只有当改动是视觉性的时，截图才是最有力的证据。对一次迁移、一个后台任务或一次重构，证据是那个先前失败、现在通过的具体测试，或者那段发生了变化的控制台输出。单独一句"测试都绿了"是一个断言，不是一个前和一个后。

**它能把正文标记为由 LLM 写的吗？**

它自己不会。一位用户的做法是在仓库的 agent 文档里放一条长期有效的规则：agent 写的每一个 issue、每一条评论、每个 PR 都以一行披露结尾。这条规则属于仓库，那里能覆盖 agent 发布的一切，而不是属于某一种文档的模板。

## 有效的话应该是这样

- 不打开 diff，你光看 Summary 那个可视化就能说出这个 PR 改了什么。
- 正文没有开场白：它从 Summary 那个标题开始。
- Evidence 那一段展示的是一个前和一个后，不是一句"测试通过"的断言。
- 每个 PR 都写下一个门判断和一个波及范围，而你要慢下来读的正是那些单向门。

## 它在整体中的位置

当构建以 pull request 的形式发布时，`pr` 位于 code-review 和 retro 之间：`to-spec → to-tickets → implement → code-review → pr → retro`。它由 model 调用，所以在那条链之外，只要 agent 写一份 PR 正文，它也会自己启动。

- [code-review](https://aihero.dev/skills-code-review) 在它之前跑，因为一份 PR 正文该描述的是一份已经被评审过的 diff。
- [implement](https://aihero.dev/skills-implement) 产出正文所描述的那些 commit。

拿不准这个情况要哪份 Skill 时，[ask-matt](https://aihero.dev/skills-ask-matt) 是罩在整个集合之上的路由。
