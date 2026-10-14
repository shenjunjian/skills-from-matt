# 领域文档

工程类 skill 在探索代码库时，应当如何消费本仓库的领域文档。

## 探索之前，先读这些

- 仓库根目录的 **`GLOSSARY.md`**，或者
- 若存在，则读仓库根目录的 **`GLOSSARY-MAP.md`**：它指向每个上下文各自的一份 `GLOSSARY.md`。读取与主题相关的每一份。
- **`docs/adr/`**：阅读涉及你即将改动区域的 ADR。多上下文仓库还要检查 `src/<context>/docs/adr/` 里的上下文级决策。

这些文件里有任何一个不存在，**就安静地继续**。不要指出它缺失，也不要主动提议先创建。`/domain-modeling` skill（通过 `/grill-with-docs` 和 `/improve-codebase-architecture` 进入）会在术语或决策真正定下来时按需创建它们。

## 文件结构

单上下文仓库（多数仓库）：

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多上下文仓库（根目录存在 `GLOSSARY-MAP.md`）：

```
/
├── GLOSSARY-MAP.md
├── docs/adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── GLOSSARY.md
    │   └── docs/adr/                  ← context-specific decisions
    └── billing/
        ├── GLOSSARY.md
        └── docs/adr/
```

## 使用词汇表里的术语

当你的产出要给某个领域概念命名（工单标题、重构提案、假设、测试名），使用 `GLOSSARY.md` 中定义的术语。不要滑向词汇表明确回避的同义词。

如果你需要的概念还没进词汇表，这是一个信号：要么你在发明项目并不使用的说法（重新想想），要么确实存在缺口（记下来交给 `/domain-modeling`）。

## 标记 ADR 冲突

如果你的产出与某份现有 ADR 相矛盾，明确把它说出来，而不是悄悄推翻：

> _与 ADR-0007（event-sourced orders）矛盾，但值得重新讨论，因为……_
