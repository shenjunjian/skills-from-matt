---
name: handoff
description: 把当前对话压缩成交接文档，供另一个 agent 接手。
argument-hint: "下一个会话要用来做什么？"
disable-model-invocation: true
---

写一份交接文档，总结当前对话，让一个新起的 agent 能接着做下去。保存到用户操作系统的临时目录（`$TMPDIR`，没有就用 `/tmp`；Windows 上是 `%TEMP%`），不要存进当前工作区。

文档里要有一个 "suggested skills" 一节，写明下一个 agent 应该用 Skill 工具调用哪些技能。

不要重复已经记录在其他产物里的内容（spec、计划、ADR、工单、提交、diff）。需要时只引用它们的路径或 URL。

抹去任何敏感信息，例如 API key、密码、个人身份信息。

如果用户传入了参数，把它们当作对下一个会话重点的描述，据此调整文档的内容。
