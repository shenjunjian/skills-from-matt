# Engineering

我每天写代码时用的 Skill。

## User-invoked（用户调用）

只有你手动输入才能触发（Claude Code：`disable-model-invocation: true`；Codex：在 `agents/openai.yaml` 里设 `policy.allow_implicit_invocation: false`）。

- **[ask-matt](./ask-matt/SKILL.md)**：问一下哪个 Skill 或哪条流程适合你现在的处境。它是对本仓库里所有 user-invoked Skill 的一个 router（路由器）。
- **[grill-with-docs](./grill-with-docs/SKILL.md)**：一场 grilling（追问）会话，同时搭建项目的领域模型，磨清术语，就地更新 `GLOSSARY.md` 和 ADR。
- **[triage](./triage/SKILL.md)**：让工单沿着一套分诊角色的状态机往前走。
- **[improve-codebase-architecture](./improve-codebase-architecture/SKILL.md)**：扫描代码库，找模块加深（deepening）的机会，用一份可视化 HTML 报告呈现出来，然后对你挑中的那一个做 grilling。
- **[setup-matt-pocock-skills](./setup-matt-pocock-skills/SKILL.md)**：为这些 engineering Skill 配置本仓库（工单跟踪器、分诊标签、领域文档布局）。每个仓库跑一次。
- **[to-spec](./to-spec/SKILL.md)**：把当前对话变成一份 spec（规格说明），并发布到工单跟踪器上。
- **[to-tickets](./to-tickets/SKILL.md)**：把任意计划、spec 或对话拆成一组 tracer-bullet 工单，每张工单都声明自己的阻塞边；在本地文件里以文本形式声明，在真实跟踪器上则用原生阻塞链接。
- **[implement](./implement/SKILL.md)**：把 spec 或一组工单所描述的工作做出来，在事先约定的 seam（接缝）处驱动 `/tdd`，提交之前用 `/code-review` 收尾。
- **[implement-spec](./implement-spec/SKILL.md)**：在一个集成分支上实现整份 spec。它把工单当作任务图来推进，在就绪的 frontier（前沿）上并行运行多个 implementer subagent（实现子代理）以拿到最大并发度，然后用 `/code-review` 收尾。
- **[wayfinder](./wayfinder/SKILL.md)**：把一大块工作（超过一次 agent 会话能容纳的量）规划成工单跟踪器上一张共享的决策工单地图，一次解决一张，直到通往目的地的路清晰可见。
- **[retro](./retro/SKILL.md)**：一次会话结束后，为编码 agent 的运行环境提出改进建议（导航指针、自动化检查、编码标准、指引文件、工具链），最严重的排最前。

## Model-invoked（模型调用）

模型或用户都能触发（触发措辞写得比较丰富，方便模型去找它们）。

- **[prototype](./prototype/SKILL.md)**：搭一个用完就扔的原型来回答一个设计问题：单个可分享的 HTML 文件应对状态/逻辑，或者几个可以来回切换的 UI 变体。

- **[diagnosing-bugs](./diagnosing-bugs/SKILL.md)**：针对难缠的 bug 和性能回退的自律诊断循环：建一个能在这个 bug 上变红的反馈循环 → 最小化 → 提出假设 → 加埋点 → 修复 → 补回归测试。
- **[research](./research/SKILL.md)**：针对高可信度的一手资料调查一个问题，把结论作为一个带引用的 Markdown 文件写进仓库，由一个后台 agent 运行。
- **[tdd](./tdd/SKILL.md)**：以 red-green-refactor（红-绿-重构）循环做测试驱动开发。按 vertical slice（纵向切片）一次一片地建功能或修 bug。
- **[domain-modeling](./domain-modeling/SKILL.md)**：主动构建并磨清项目的领域模型：质疑术语、用场景做压力测试、就地更新 `GLOSSARY.md` 和 ADR。
- **[codebase-design](./codebase-design/SKILL.md)**：设计 deep module（深模块）共用的纪律与词汇：小接口、干净的 seam（接缝）、可以通过接口来测试。
- **[code-review](./code-review/SKILL.md)**：对某个定点之后的 diff 做双轴评审：**Standards**（它是否遵循本仓库的编码标准，外加一套 Fowler 坏味道基线？）和 **Spec**（它是否忠实实现了源头的工单/spec？），两轴以并行 subagent（子代理）运行。
- **[pr](./pr/SKILL.md)**：pull request 正文应有的形状：一段摘要，用能讲清这次改动的最小视觉呈现；改动可用的前后证据；一个合并风险判断（单向门还是双向门，外加影响半径）。
- **[wizard](./wizard/SKILL.md)**：生成一个交互式 bash 向导，带人走一遍只有人自己能做的步骤：开通基础设施、配置凭据或 CI 密钥、在陌生的第三方控制台里点来点去、跑一次性的迁移或切换。
