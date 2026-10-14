# 超出范围知识库

仓库里的 `.out-of-scope/` 目录用来持久记录被拒绝的功能请求。它有两个用途：

1. **机构记忆**：记下一个功能为什么被拒绝，这样工单关闭之后理由不会丢失
2. **去重**：当新工单与某条既往的拒绝记录相符时，skill（技能）可以直接给出之前的决定，而不必重新争论一遍

## 目录结构

```
.out-of-scope/
├── dark-mode.md
├── plugin-system.md
└── graphql-api.md
```

每个文件对应一个**概念**，而不是一个工单。多个工单请求同一件事时，归到同一个文件下。

## 文件格式

文件要用轻松、易读的风格来写，更像一篇简短的设计文档，而不是数据库记录。用段落、代码示例和例子把理由讲清楚，让第一次读到它的人也能看明白并且觉得有用。

```markdown
# Dark Mode

This project does not support dark mode or user-facing theming.

## Why this is out of scope

The rendering pipeline assumes a single color palette defined in
`ThemeConfig`. Supporting multiple themes would require:

- A theme context provider wrapping the entire component tree
- Per-component theme-aware style resolution
- A persistence layer for user theme preferences

This is a significant architectural change that doesn't align with the
project's focus on content authoring. Theming is a concern for downstream
consumers who embed or redistribute the output.

```ts
// The current ThemeConfig interface is not designed for runtime switching:
interface ThemeConfig {
  colors: ColorPalette; // single palette, resolved at build time
  fonts: FontStack;
}
```

## Prior requests

- #42: "Add dark mode support"
- #87: "Night theme for accessibility"
- #134: "Dark theme option"
```

### 文件命名

给概念起一个简短、有描述性的 kebab-case 名字：`dark-mode.md`、`plugin-system.md`、`graphql-api.md`。名字要让人浏览目录时不必打开文件就能看懂被拒绝的是什么。

### 怎么写理由

理由要有实质内容：不是"我们不想要这个"，而是为什么不要。好的理由会引用：

- 项目范围或理念（"这个项目专注于 X；主题定制是下游使用者的问题"）
- 技术约束（"要支持这个就得做 Y，这与我们的 Z 架构冲突"）
- 战略决策（"我们选择用 A 而不是 B，因为……"）

理由要经得起时间考验。别引用一时的状况（"我们最近太忙"）：那不是真正的拒绝，只是推迟。

## 什么时候检查 `.out-of-scope/`

在 triage（分诊）过程中（第 1 步：收集上下文），读取 `.out-of-scope/` 下的全部文件。评估新工单时：

- 检查请求是否与某个已有的超范围概念相符
- 相符与否看概念相似度，不看关键词："night theme" 与 `dark-mode.md` 相符
- 如果相符，就把结果呈现给维护者："这和 `.out-of-scope/dark-mode.md` 类似。我们之前拒绝过，理由是 [reason]。你现在还是同样的看法吗？"

维护者可能：

- **确认**：新工单被加进已有文件的 "Prior requests" 列表，然后关闭
- **重新考虑**：删除或更新这个超范围文件，工单走正常分诊流程
- **不认同**：两个工单相关但不同，走正常分诊流程

## 什么时候写入 `.out-of-scope/`

只有当一个 **enhancement**（功能改进，不是 bug）被*拒绝*并标记为 `wontfix` 时才写入。这一点对 enhancement 类型的 PR 与对工单完全一致：被拒绝的 PR 同样记录在这里，好让同一个请求不再以新代码的形式回来。

当某件事被以 `wontfix` 关闭的原因是**已经实现**时，**不要**写到这里。那是已建成的功能，不是被拒绝的功能；记录它会让去重检查被虚假的拒绝污染。正确的做法是在关闭评论中指出这个功能已存在于何处。

流程：

1. 维护者判定某个功能请求超出范围
2. 检查是否已有与之相符的 `.out-of-scope/` 文件
3. 有：把新工单追加到 "Prior requests" 列表
4. 没有：新建一个文件，写下概念名、决定、理由，以及第一条既往请求
5. 在工单上发一条评论，说明决定并提到这个 `.out-of-scope/` 文件
6. 用 `wontfix` 标签关闭工单

## 更新或删除超范围文件

如果维护者对之前某个被拒绝的概念改变了想法：

- 删除这个 `.out-of-scope/` 文件
- skill 不需要重开旧工单；它们是历史记录
- 触发这次重新考虑的新工单走正常分诊流程
