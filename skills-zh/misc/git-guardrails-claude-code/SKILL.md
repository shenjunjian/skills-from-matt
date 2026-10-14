---
name: git-guardrails-claude-code
description: 配置 Claude Code 钩子，在危险 git 命令（push、reset --hard、clean、branch -D 等）执行之前把它们拦下。当用户想要阻止破坏性的 git 操作、添加 git 安全钩子，或在 Claude Code 里拦住 git push/reset 时使用。
---

# 配置 Git 防护栏

配置一个 PreToolUse 钩子，在 Claude 执行危险 git 命令之前拦截并阻止它们。

## 会拦下哪些

- `git push`（所有变体，包括 `--force`）
- `git reset --hard`
- `git clean -f` / `git clean -fd`
- `git branch -D`
- `git checkout .` / `git restore .`

被拦下时，Claude 会看到一条消息，告诉它无权访问这些命令。

## 步骤

### 1. 询问作用范围

问用户：只为**这个项目**安装（`.claude/settings.json`），还是为**所有项目**安装（`~/.claude/settings.json`）？

### 2. 复制钩子脚本

随 Skill 打包的脚本在：[scripts/block-dangerous-git.sh](scripts/block-dangerous-git.sh)

按作用范围把它复制到目标位置：

- **项目**：`.claude/hooks/block-dangerous-git.sh`
- **全局**：`~/.claude/hooks/block-dangerous-git.sh`

用 `chmod +x` 让它可执行。

### 3. 把钩子加进 settings

加到对应的 settings 文件里：

**项目**（`.claude/settings.json`）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

**全局**（`~/.claude/settings.json`）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

如果 settings 文件已经存在，把这个钩子合并进现有的 `hooks.PreToolUse` 数组。不要覆盖其他配置。

### 4. 询问是否需要定制

问用户要不要在拦截清单里增删匹配模式，并据此编辑复制过去的脚本。

### 5. 验证

跑一个快速测试：

```bash
echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>
```

应当以退出码 2 结束，并向 stderr 打印一条 BLOCKED 消息。
