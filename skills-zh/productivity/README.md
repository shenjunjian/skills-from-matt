# Productivity（生产力）

通用的工作流工具，不针对代码。

## 用户触发

只有你亲自输入才会触发（Claude Code：`disable-model-invocation: true`；Codex：在 `agents/openai.yaml` 中设置 `policy.allow_implicit_invocation: false`）。

- **[grill-me](./grill-me/SKILL.md)**：就一个计划或设计接受连环追问，直到设计树的每个分支都定下来。
- **[handoff](./handoff/SKILL.md)**：把当前对话压缩成交接文档，让另一个 agent 能接着做下去。
- **[teach](./teach/SKILL.md)**：跨多次会话教会用户一项新技能或一个概念，把当前目录当作有状态的教学工作区。
- **[to-questionnaire](./to-questionnaire/SKILL.md)**：把你自己答不了的决策变成一份 Markdown 问卷，交给唯一能答的那个人（异步填写，或在会议上一同完成）。
- **[wait-what](./wait-what/SKILL.md)**：某条消息一说出来你没听懂，立刻调用它。agent 会补上你缺失的上下文，用大白话、按你 `GLOSSARY.md` 里的词汇表重新讲一遍。

## 模型触发

模型或用户都能触发（触发短语写得很丰富，好让模型能主动取用）。

- **[grilling](./grilling/SKILL.md)**：就一个计划、决策或想法连环追问用户，直到设计树的每个分支都定下来。
- **[writing-for-agents](./writing-for-agents/SKILL.md)**：为 agent 写文档：Skill（技能）、AGENTS.md/CLAUDE.md，以及任何 agent 顺指针取用的文档。
