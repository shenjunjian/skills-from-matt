# GLOSSARY.md 格式

## 结构

```md
# {Context Name}

{One or two sentence description of what this context is and why it exists.}

## Language

**Order**:
{A one or two sentence description of the term}
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account
```

## 规则

- **要有立场。**同一个概念存在多个说法时，选出最好的那一个，其余列进 `_Avoid_`。
- **释义要紧。**最多一两句话。定义它本身是什么，而不是它能做什么。
- **只收录本项目上下文特有的术语。**通用编程概念（超时、错误类型、工具类模式）不该进来，即便项目大量使用它们。加术语之前先问：这是本上下文独有的概念，还是通用编程概念？只有前者该收录。
- **出现自然的分组时，用子标题把术语归类。**如果所有术语都属于同一个内聚领域，平铺一份列表就行。

## 单上下文与多上下文的仓库

**单一上下文（多数仓库）：**仓库根目录放一份 `GLOSSARY.md`。

**多个上下文：**仓库根目录放一份 `GLOSSARY-MAP.md`，列出各个上下文、它们所在位置、以及它们之间的关系：

```md
# Glossary Map

## Contexts

- [Ordering](./src/ordering/GLOSSARY.md): receives and tracks customer orders
- [Billing](./src/billing/GLOSSARY.md): generates invoices and processes payments
- [Fulfillment](./src/fulfillment/GLOSSARY.md): manages warehouse picking and shipping

## Relationships

- **Ordering → Fulfillment**: Ordering emits `OrderPlaced` events; Fulfillment consumes them to start picking
- **Fulfillment → Billing**: Fulfillment emits `ShipmentDispatched` events; Billing consumes them to generate invoices
- **Ordering ↔ Billing**: Shared types for `CustomerId` and `Money`
```

Skill 会推断适用哪种结构：

- 如果存在 `GLOSSARY-MAP.md`，读它找出各个上下文
- 如果只有根目录的 `GLOSSARY.md`，就是单一上下文
- 如果两者都没有，在第一个术语确定下来时，按需创建根目录的 `GLOSSARY.md`

存在多个上下文时，推断当前话题属于哪一个。不清楚就问。
