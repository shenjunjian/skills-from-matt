---
name: scaffold-exercises
description: 创建含 sections、problems、solutions 和 explainers 的练习目录结构，并通过 lint 检查。当用户想搭建练习脚手架、创建练习占位文件，或初始化一个新的课程章节时使用。
---

# 搭建练习脚手架

创建能通过 `pnpm ai-hero-cli internal lint` 的练习目录结构，然后用 `git commit` 提交。

## 目录命名

- **Sections（章节）**：`exercises/` 下的 `XX-section-name/`（例如 `01-retrieval-skill-building`）
- **Exercises（练习）**：章节内的 `XX.YY-exercise-name/`（例如 `01.03-retrieval-with-bm25`）
- 章节编号 = `XX`，练习编号 = `XX.YY`
- 名称用 dash-case（小写，用连字符）

## 练习变体

每个练习至少需要下列子目录中的一个：

- `problem/` - 学生做题的工作区，含 TODO
- `solution/` - 参考实现
- `explainer/` - 概念讲解材料，没有 TODO

搭占位结构时，除非计划另有说明，默认用 `explainer/`。

## 必需文件

每个子目录（`problem/`、`solution/`、`explainer/`）都要有一个 `readme.md`，它：

- **非空**（必须有实际内容，哪怕只有一行标题）
- 没有失效链接

搭占位结构时，写一个只含标题和描述的最小 readme：

```md
# Exercise Title

Description here
```

子目录里如果有代码，还需要一个 `main.ts`（超过 1 行）。但只是占位时，仅有 readme 的练习就可以。

## 工作流

1. **解析计划** - 提取章节名、练习名和变体类型
2. **创建目录** - 对每个路径执行 `mkdir -p`
3. **创建占位 readme** - 每个变体目录一个 `readme.md`，带标题
4. **运行 lint** - 用 `pnpm ai-hero-cli internal lint` 校验
5. **修掉报错** - 反复调整直到 lint 通过

## Lint 规则摘要

linter（`pnpm ai-hero-cli internal lint`）会检查：

- 每个练习都有子目录（`problem/`、`solution/`、`explainer/`）
- `problem/`、`explainer/`、`explainer.1/` 至少存在一个
- 主子目录里有 `readme.md` 且内容非空
- 没有 `.gitkeep` 文件
- 没有 `speaker-notes.md` 文件
- readme 里没有失效链接
- readme 里没有 `pnpm run exercise` 命令
- 每个子目录都需要 `main.ts`，除非它是仅有 readme 的

## 移动与重命名练习

重新编号或移动练习时：

1. 用 `git mv`（不是 `mv`）重命名目录 - 保留 git 历史
2. 更新数字前缀，维持顺序
3. 移动后重新运行 lint

示例：

```bash
git mv exercises/01-retrieval/01.03-embeddings exercises/01-retrieval/01.04-embeddings
```

## 示例：按计划搭占位结构

给定这样一个计划：

```
Section 05: Memory Skill Building
- 05.01 Introduction to Memory
- 05.02 Short-term Memory (explainer + problem + solution)
- 05.03 Long-term Memory
```

创建：

```bash
mkdir -p exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer
mkdir -p exercises/05-memory-skill-building/05.02-short-term-memory/{explainer,problem,solution}
mkdir -p exercises/05-memory-skill-building/05.03-long-term-memory/explainer
```

然后创建 readme 占位文件：

```
exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer/readme.md -> "# Introduction to Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/explainer/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/problem/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/solution/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.03-long-term-memory/explainer/readme.md -> "# Long-term Memory"
```
