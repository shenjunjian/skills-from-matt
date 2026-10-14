# MISSION.md 格式

`MISSION.md` 放在工作区根目录。它记录用户学这个主题的_理由_。每一个教学决策（接下来教什么、呈现哪些资源、怎么设计练习）都应当能追溯到这份文档。

## 模板

```md
# Mission: {Topic}

## Why
{1-3 sentences. The concrete real-world goal the user is chasing. What changes in their life or work when they have this skill? Avoid abstract framings like "to understand X"; push for the underlying outcome.}

## Success looks like
- {A specific, observable thing the user will be able to do}
- {Another specific thing}
- {…}

## Constraints
- {Time, budget, prior commitments, learning preferences, anything that bounds the approach}

## Out of scope
- {Adjacent topics the user explicitly does not want to chase right now, protecting the zone of proximal development}
```

## 规则

- **一个工作区一个 mission（学习使命）。** 用户想学两件不相关的事，那就是两个工作区。
- **具体优先于抽象。** "十月之前跑完半个马拉松" 胜过 "变健康"。"给团队交付一个 Rust CLI" 胜过 "学 Rust"。
- **遇到含糊要顶回去。** 用户说不清为什么，就先访谈他，再动笔。一个糟糕的 mission 比没有 mission 更糟。
- **现实变了就修订。** mission 会变。用户的目的一旦转移，就更新这份文件：不要让一个过时的 mission 继续引导后续会话。
- **保持简短。** `MISSION.md` 超过一屏，它就不再是指南针，而变成了一份计划。
