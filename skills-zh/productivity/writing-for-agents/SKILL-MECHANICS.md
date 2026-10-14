# Skill 机制

[`writing-for-agents`](SKILL.md) 的技能专属分支：文档是一个 Skill 时会变什么（frontmatter、调用方式的选择，以及路由 Skill）。其余关于怎么写的东西，全在 `SKILL.md` 这份通用参考里。

## 调用方式

两种选择，在两种负担之间做交换：

- **模型调用（model-invoked）** 的 Skill 保留 `description`，于是 agent 能自主触发它，别的 Skill 也能指向它。你照样可以手打它的名字：模型调用始终 _包含_ 人工触达；description 只会给 agent 加上发现能力，绝不夺走人的那一份。description 是这个 Skill 的顶层上下文指针，被迫一直挂着：以永久的上下文负担换可被发现性。一个内容全是参考材料的模型调用 Skill，同时是共享参考材料的容身处：别的 Skill 可以调用它，于是几个 Skill 都要用的参考材料就待在同一处。机制：省略 `disable-model-invocation`，写一段面向模型的 description，把触发分支装进去（`SKILL.md` 里的指针写法规则完全适用）。
- **人工调用（user-invoked）** 的 Skill 把 description 从 agent 的触达范围里剥掉：只有人打出它的名字才能调用，别的 Skill 也不行。零上下文负担，但花的是认知负担：你就是那个索引，得记住它存在。机制：设 `disable-model-invocation: true`；`description` 转而面向人：一行摘要，删掉触发清单。

只有当 agent 必须自己找到这个 Skill，或者别的 Skill 必须找到它时，才选模型调用。若它全靠手动触发，就做成人工调用，一分上下文负担都不付。

两个人工调用 Skill 都要用的共享参考材料，哪个里都放不了：没有 description，谁也触发不了谁。把它推到 Skill 体系之外的一个普通文件里：任何 Skill 都能指向的外部参考材料。

## 按调用方式拆分

按调用方式下这一刀（按顺序下刀那一节在 `SKILL.md`）：当你有一个独立的先行词（leading word）该独自触发它（一个你在 prompt 里真的会用的触发词），或者别的 Skill 必须够到它，就拆出一个模型调用 Skill。新的 description 会常驻上下文，你要为它付上下文负担，所以这份独立触达能力得值这个价。

## 路由 Skill

当人工调用的 Skill 多到超出你记得住的地步，这堆积压的认知负担靠**路由 Skill**来解：一个人工调用的 Skill，把其余那些点名列出，并写明每一个什么时候去找，于是人只需要记住一个 Skill，而不是许多。它只能提示，永远触发不了它们：人工调用的 Skill 没有 description，所以除了人没有别的东西够得到它们。
