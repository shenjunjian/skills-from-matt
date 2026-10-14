---
name: claude-handoff
description: 把当前会话交给一个新的后台 agent，让它立刻接手工作。
argument-hint: "下一个会话要用来做什么？"
disable-model-invocation: true
---

为当前会话写一份交接摘要，让一个新 agent 能接着干下去。把它存到用户操作系统的临时目录，然后启动一个后台 agent，用这份摘要作为它的提示词：`claude --bg --name "<descriptive name>" -- "$(cat <summary file>)"`。传文件而不是传内容，是为了让 shell 不去执行摘要里的反引号，也不去展开其中的 `$`。它在当前工作目录里启动，并立刻返回；用户用 `claude agents` 管理它。

始终带上 `-n`/`--name` 和一个描述性的名字（例如 `--name "Fix login bug"`）；它设置的是任务列表、会话选择器和终端标题里显示的那个名字。

摘要里要包含一节 "suggested skills"（建议使用的 Skill），写明下一个 agent 该调用 Skill 工具去用哪些 Skill。

不要复制别的产出物（spec、计划、ADR、工单、提交、diff）里已经装下的内容。改用路径或 URL 引用它们。

把任何敏感信息涂抹掉，比如 API key、密码、可识别个人身份的信息，因为这份摘要会变成那个 agent 的提示词。

如果用户传了参数，就把它们当成下一次会话聚焦点的描述，据此裁剪摘要。
