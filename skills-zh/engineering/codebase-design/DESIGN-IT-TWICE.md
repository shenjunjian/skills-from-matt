# 同一接口设计两次

当用户想为某个选定的深化候选项探索不同的 interface 时，使用这套并行 sub-agent 模式。出处是 "Design It Twice"（Ousterhout）：你的第一个想法多半不是最好的。

使用 [SKILL.md](SKILL.md) 里的词汇：**module**、**interface**、**seam**、**adapter**、**leverage**。

## 流程

### 1. 界定问题空间

在派生 sub-agent 之前，先为选定候选项写一份面向用户的问题空间说明：

- 任何新的 interface 都必须满足的约束
- 它会依赖哪些依赖项，各自属于哪一类（见 [DEEPENING.md](DEEPENING.md)）
- 一份粗略的示意代码草稿，用来把约束落到实处。它不是提案，只是一种让约束变得具体的手段

把这份说明展示给用户，随后立刻进入第 2 步。用户阅读和思考的同时，sub-agent 们在并行工作。

### 2. 派生 sub-agent

并行派生 3 个以上 sub-agent。每个都必须产出一个**截然不同**的 interface，用于那个深化后的 module。

给每个 sub-agent 单独一份技术简报（文件路径、耦合细节、来自 [DEEPENING.md](DEEPENING.md) 的依赖类别、seam 后面是什么）。这份简报独立于第 1 步那份面向用户的问题空间说明。给每个 agent 一个不同的设计约束：

- Agent 1："精简 interface：入口点最多控制在 1 到 3 个。让每个入口点的 leverage 最大化。"
- Agent 2："最大化灵活性：支持尽可能多的用例与扩展。"
- Agent 3："为最常见的调用方优化：让默认用法变得不费力。"
- Agent 4（适用时）："围绕 ports & adapters 来设计跨 seam 的依赖。"

简报里要同时包含 [SKILL.md](SKILL.md) 的词汇和 GLOSSARY.md 的词汇，这样每个 sub-agent 命名事物时都能与架构语言和项目的领域语言保持一致。

每个 sub-agent 输出：

1. interface（类型、方法、参数，以及不变量、调用顺序、错误模式）
2. 用法示例，展示调用方如何使用它
3. 实现把什么藏在了 seam 后面
4. 依赖策略与 adapter（见 [DEEPENING.md](DEEPENING.md)）
5. 取舍：leverage 在哪里厚实，在哪里单薄

### 3. 呈现与比较

依次呈现各个设计，让用户逐个消化，然后用文字把它们放在一起比较。从 **depth**（interface 处的 leverage）、**locality**（变更集中在哪里）和 **seam 的位置** 三个维度来对比。

比较之后给出你自己的推荐：你认为哪个设计最强，理由是什么。如果不同设计里的某些元素组合起来效果好，就提出一个混合方案。要有立场：用户要的是一句明确的判断，不是一份菜单。
