# Triage 标签

这些 Skill 用五个固定的 triage（分诊）角色来表述。本文件把这些角色映射到本仓库工单跟踪器里实际使用的标签字符串。

| mattpocock/skills 中的标签 | 我们跟踪器中的标签 | 含义                                   |
| -------------------------- | -------------------- | ---------------------------------------- |
| `needs-triage`             | `needs-triage`       | 维护者需要评估这个工单             |
| `needs-info`               | `needs-info`         | 等报告人补充更多信息               |
| `ready-for-agent`          | `ready-for-agent`    | 已完整定义，可交给 AFK agent       |
| `ready-for-human`          | `ready-for-human`    | 需要人来实现                       |
| `wontfix`                  | `wontfix`            | 不会处理                           |

当某个 Skill 提到一个角色（例如"应用 AFK-ready 的 triage 标签"），就用这张表里对应的标签字符串。

选定状态之前，每个工单都要按 [`SCOPE.md`](../../SCOPE.md) 判断一遍。被拒绝的工单以 "not planned" 关闭。
