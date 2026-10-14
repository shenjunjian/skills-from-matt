---
name: domain-modeling
description: 构建并打磨项目的领域模型。当需要讨论代码库术语、编写或修改 GLOSSARY.md、或记录或修改 ADR 时使用。
---

# 领域建模

在设计的同时主动构建并打磨项目的领域模型。这是一门*主动*的纪律：推敲术语、编造极端场景、在术语和决定成形的当下就把它们写下来。（只是*阅读* `GLOSSARY.md` 取用词汇不算这个 Skill：那是一行习惯，任何 Skill 都能做。这个 Skill 针对的是你在改动模型，而不只是消费模型。）

## 文件结构

多数仓库只有一个上下文：

```
/
├── GLOSSARY.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

如果根目录存在 `GLOSSARY-MAP.md`，说明仓库有多个上下文。这份 map 指明每个上下文所在位置：

```
/
├── GLOSSARY-MAP.md
├── docs/
│   └── adr/                          ← system-wide decisions
├── src/
│   ├── ordering/
│   │   ├── GLOSSARY.md
│   │   └── docs/adr/                 ← context-specific decisions
│   └── billing/
│       ├── GLOSSARY.md
│       └── docs/adr/
```

文件按需创建：只在有内容可写时才创建。没有 `GLOSSARY.md` 就在第一个术语确定下来时建一份。没有 `docs/adr/` 就在需要第一份 ADR 时建。

## 会话进行中

### 拿术语表来对质

用户使用的术语与 `GLOSSARY.md` 里已有的语言冲突时，立刻指出。"你的术语表把 'cancellation' 定义成 X，但你要表达的意思似乎是 Y。到底是哪个？"

### 打磨含混的语言

用户用到模糊或含义过载的词时，提出一个精确的规范术语。"你说的是 'account'：指 Customer 还是 User？这是两回事。"

### 讨论具体场景

讨论领域关系时，用具体场景去做压力测试。编造能触及边界情况、迫使用户把概念之间的界线说清楚的场景。

### 与代码交叉印证

用户说出某个机制如何工作时，去核对代码是否同意。发现矛盾就摆出来："你的代码取消的是整张 Order，但你刚才说可以部分取消。哪个是对的？"

### 就地更新 GLOSSARY.md

一个术语确定下来，就当场更新 `GLOSSARY.md`。别攒着：在它发生的当下就记下来。格式见 [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md)。

`GLOSSARY.md` 应当完全不涉及实现细节。不要把 `GLOSSARY.md` 当 spec（规格说明）、草稿本，或存放实现决定的仓库。它就是一份术语表，仅此而已。

### 克制地给出 ADR

只在下面三条都成立时，才提议创建 ADR：

1. **难以回退**：日后反悔的代价不小
2. **脱离上下文会让人意外**：后来的人会想"为什么会这么做？"
3. **是真实取舍的结果**：确实存在别的方案，你出于具体理由选了一个

三条里缺任何一条，就不写 ADR。格式见 [ADR-FORMAT.md](./ADR-FORMAT.md)。
