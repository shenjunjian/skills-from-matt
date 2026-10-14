---
name: codebase-design
description: 用于设计深层 module 的共享词汇。当用户想设计或改进某个 module 的 interface、寻找深化的机会、决定 seam 该放在哪里、让代码更容易测试或更容易被 AI 导航，或者另一个 skill 需要这套深层模块词汇时使用。
---

# 代码库设计

设计 **deep module（深层模块）**：interface 小而行为多，放在干净的 seam 上，并且能通过那个 interface 来测试。凡是在设计或重构代码的地方，都用这套语言和这些原则。目标是让调用方获得 leverage，让维护者获得 locality，让所有人都能测试。

## Glossary（术语表）

严格使用这些术语：不要用 "component"、"service"、"API" 或 "boundary" 来替代。语言一致本身就是全部意义所在。

**Module**：任何同时具备 interface 和实现的东西。刻意不绑定规模：一个函数、一个类、一个 package，或一段跨层的内容都可以。_避免_：unit、component、service。

**Interface**：调用方为了正确使用这个 module 必须知道的一切：类型签名，还包括不变量、调用顺序约束、错误模式、必需的配置，以及性能特征。_避免_：API、signature（都太窄，它们只指类型层面的表面）。

**Implementation**：module 内部的东西，它的代码本体。与 **Adapter** 不同：一个东西可以是很小的 adapter 配很大的 implementation（Postgres 仓储），也可以是很大的 adapter 配很小的 implementation（内存里的假实现）。当话题是 seam 时用 "adapter"，其他情况用 "implementation"。

**Depth**：interface 处的 leverage。调用方（或测试）每学一份 interface，能驱动多少行为。小的 interface 后面藏着大量行为，这个 module 就是 **deep**；interface 的复杂度几乎和实现一样高，它就是 **shallow**。

**Seam** _(Michael Feathers)_：一个让你无需在该处编辑就能改变行为的位置；module 的 interface 所在的*位置*。seam 放在哪里本身是一个独立的设计决策，与它后面放什么无关。_避免_：boundary（它被 DDD 的 bounded context 占用了）。

**Adapter**：在 seam 处满足某个 interface 的具体东西。它描述*角色*（填哪个槽），不描述实质（里面装了什么）。

**Leverage**：调用方从 depth 那里得到的东西。学到的每一份 interface 换来更多能力。一份实现能在 N 个调用点和 M 个测试上反复回收成本。

**Locality**：维护者从 depth 那里得到的东西。变更、缺陷、知识和验证集中在一个地方，而不是摊到各个调用方身上。改一处，处处已改。

## 深层与浅层

**Deep module** = 小 interface + 大量 implementation：

```
┌─────────────────────┐
│   Small Interface   │  ← Few methods, simple params
├─────────────────────┤
│                     │
│  Deep Implementation│  ← Complex logic hidden
│                     │
└─────────────────────┘
```

**Shallow module** = 大 interface + 少量 implementation（要避免）：

```
┌─────────────────────────────────┐
│       Large Interface           │  ← Many methods, complex params
├─────────────────────────────────┤
│  Thin Implementation            │  ← Just passes through
└─────────────────────────────────┘
```

设计 interface 时，问自己：

- 我能减少方法数量吗？
- 我能简化参数吗？
- 我能把更多复杂度藏到里面去吗？

## 原则

- **depth 是 interface 的性质，不是实现的性质。** 一个深层 module 内部可以由小的、可 mock、可替换的部分组成；它们只是不属于 interface。一个 module 可以有 **internal seam**（对其实现私有，供自己的测试使用），也可以有位于 interface 上的 **external seam**。
- **删除测试。** 设想把这个 module 删掉。如果复杂度随之消失，那它只是一个透传层。如果复杂度在 N 个调用方身上重新冒出来，那它确实在挣自己的饭钱。
- **interface 就是测试面。** 调用方和测试跨过同一个 seam。如果你想在 interface *之外*去测，这个 module 的形状多半就错了。
- **一个 adapter 意味着假设的 seam，两个 adapter 才是真实的。** 除非确实有东西在 seam 两端变化，否则不要引入 seam。

## 为可测试性而设计

好的 interface 让测试变得自然：

1. **接收依赖，不要自己创建依赖。**

   ```typescript
   // Testable
   function processOrder(order, paymentGateway) {}

   // Hard to test
   function processOrder(order) {
     const gateway = new StripeGateway();
   }
   ```

2. **返回结果，不要产生副作用。**

   ```typescript
   // Testable
   function calculateDiscount(cart): Discount {}

   // Hard to test
   function applyDiscount(cart): void {
     cart.total -= discount;
   }
   ```

3. **表面面积要小。** 方法少，需要的测试就少。参数少，测试的准备工作就简单。

## 相互关系

- 一个 **Module** 有且只有一个 **Interface**（它呈现给调用方和测试的那个表面）。
- **Depth** 是 **Module** 的一个性质，以它的 **Interface** 为参照来衡量。
- **Seam** 是 **Module** 的 **Interface** 所在之处。
- **Adapter** 位于 **Seam** 处，并满足那个 **Interface**。
- **Depth** 为调用方带来 **Leverage**，为维护者带来 **Locality**。

## 被否决的表述方式

- **把 depth 定义为 implementation 行数与 interface 行数之比**（Ousterhout）：这会奖励往实现里塞东西。我们用的是 depth-as-leverage。
- **把 "interface" 当作 TypeScript 的 `interface` 关键字或一个类的公开方法**：太窄。这里的 interface 包含调用方必须知道的每一条事实。
- **"Boundary"**：被 DDD 的 bounded context 占用了。要说 **seam** 或 **interface**。

## 深入下去

- **按依赖条件深化一团模块**，见 [DEEPENING.md](DEEPENING.md)：依赖分类、seam 纪律，以及"替换而不叠加"的测试策略。
- **探索备选 interface**，见 [DESIGN-IT-TWICE.md](DESIGN-IT-TWICE.md)：并行派生若干 sub-agent，把 interface 设计成几种截然不同的样子，再从 depth、locality 和 seam 位置三方面比较。
