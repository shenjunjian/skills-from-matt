> **已归档。** 本 Skill 在 v1.3.0 中从插件移除，不再维护。没有任何东西接替它：agent 处理 merge 或 rebase 冲突时不再依赖专门的 Skill。本页保留作为参考。

## 它做什么

`resolving-merge-conflicts` 逐 hunk 处理正在进行中的 git merge 或 rebase，然后运行项目自带的检查，并以一次 commit 完成整个操作。

它拒绝把冲突当作文本问题。在碰任何一个 hunk 之前，它先把两侧各自追溯回其 **[一手资料](https://www.aihero.dev/ai-coding-dictionary/primary-source)**（commit message、PR、原始工单），因此它是在两个意图之间做选择，而不是在两块文本之间做选择，并且在两者相容的地方把两边都保留下来。不相容的地方，它选择符合本次 merge 既定目标的一侧，并说明取舍。它不编造新行为来掩盖冲突，也从不使用 `--abort`。它总是把这次合并推进到一次完成的 commit。

## 什么时候用它

输入 `/resolving-merge-conflicts`，或者当任务合适时 [agent](https://www.aihero.dev/ai-coding-dictionary/agent)（智能体）会自动用它。

当 git 已经停在它自己无法解决的冲突上时用它。它的范围限于你眼前的冲突，不含冲突前后的任何事：

| 你的处境 | 用什么 Skill |
| --- | --- |
| 正在 merge 或 rebase 中，工作树里有冲突标记 | 本 Skill |
| merge 已完成，现在有东西行为异常，原因你看不见 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |
| 规划怎么切分工作，让分支之间少撞车 | 都不用：见下面关于并行工作的问题 |

## 一手资料优先于 `ours` 和 `theirs`

本 Skill 存在的目的是防止靠标志位解决冲突：`--ours`、`--theirs`，或者手动删掉看起来不那么重要的那一块，好让标记消失、构建通过。这样的解决在语法上可以完美无缺，却悄悄丢掉某人刻意做的一处改动。

你无法保留一个你没读过的意图。所以工作从历史开始（commit、PR、[ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)（工单）），之后才转向 diff。检查这一步的存在也是同一个理由。Skill 会找到仓库自带的[自动化检查](https://www.aihero.dev/ai-coding-dictionary/automated-check)，在提交之前运行它们，因为在 git 里，一次 merge 是最容易产出"同时满足两个分支、却两个分支的测试都通不过"的代码的地方。

## 常见问题

**Claude Code 自己就能把冲突解决得相当好。为什么这还需要一个 Skill？**

额外的价值在于"找到一手资料"和"运行反馈回路"这两步，否则你每次都得手动提示。未经提示的 agent 通常只凭 diff 给出一个看着合理的解决方案，然后停手。本 Skill 的价值是那两个它不允许 agent 跳过的步骤：读懂每一侧存在的原因，以及随后运行检查。相比一个好的[模型](https://www.aihero.dev/ai-coding-dictionary/model)，这是很小的提升，而这正是本意。至少有一位读者预言，整个 Skill 会随着模型进步而变成空操作。

**我该不该一开始就让并行的 agent 别碰同一批文件，以避免冲突？**

通常不该。在并行任务之间给文件划区，代价大于收益，因为 agent 处理 merge 冲突已经足够好，这个取舍没有看上去那么严酷。唯一值得守住的纪律是先做大重构。一个大重命名在十条分支已经拉出之后才落地，这才是那件始终昂贵的事。

一条来自并行 worktree 用户报告的注意事项：当同级的各个 [session](https://www.aihero.dev/ai-coding-dictionary/session)（会话）都在自己的工作树里构建一张工单时，往回合并最好由写出这处改动的那个会话来做，因为只有它已经了解意图。如果让一个 agent 在最后统一解决所有人的冲突，它就丢掉了本 Skill 第 2 步随后必须重建的 [context](https://www.aihero.dev/ai-coding-dictionary/context)（上下文）。

**为什么永远不用 `--abort`？**

中止会把已做的解决工作丢掉，并在你下次尝试时把同一个冲突原样还给你。本 Skill 为"这次合并注定要完成"的情形而写。如果你已经决定它不该发生，那是调用之前要做的决定，不是循环内部的一条分支。

## 起作用的表现

- agent 在解决过程中向你引用 commit message、PR 或工单，而不只是 diff 的 hunk。
- 每个 hunk 最终都同时具备两侧的行为，或者留下一条明确的说明，点出丢掉了什么、为什么。
- 结果里没有出现任何一条分支上都不存在的东西。
- agent 找到了 typecheck、测试和格式检查，并在提交*之前*把它们跑到全绿，而不是等你发现某处坏了之后才跑。
- 结束时工作树干净、操作完成，多 commit 的 rebase 里剩下的每个 commit 也都完成。

## 它在整体中的位置

一个随手可用的独立 Skill，不依赖任何其他 Skill：git 卡住时它开始，工作树干净并完成提交时它结束。它唯一真正的邻居是 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)，在"merge 解决得很干净、但合并后的代码行为异常"这个交接点接手：那是诊断问题，不是冲突问题。它在从想法到交付的主流程之外，所以它前后各该跑什么，看 [ask-matt](https://aihero.dev/skills-ask-matt) 这张地图。
