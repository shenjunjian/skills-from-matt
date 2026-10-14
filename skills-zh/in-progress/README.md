# In Progress（进行中）

Beta。这些 Skill 有意公开：试用它们，然后告诉我哪里坏了。在它们毕业进入稳定分组之前，插件和顶层 README 都不包含它们，它们也没有文档页，并且可能毫无预警地改变或消失。

插件不会给你这些。单独装一个：

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

- **[loop-me](./loop-me/SKILL.md)**：把自己拷问到吐出可实现的工作流 spec（规格说明），跨多次会话进行，把当前目录当作有状态的 workspace（工作区）。由用户调用。
- **[writing-beats](./writing-beats/SKILL.md)**：把一篇文章塑造成一段节拍的旅程，走"自己选冒险路线"的路子。挑一个起始节拍，只写那个节拍，再转到下一个，直到文章自然收尾。
- **[writing-fragments](./writing-fragments/SKILL.md)**：一场拷问式会话，把你身上挖出碎片（各种不成一体的文字块），追加进同一个文档，作为将来成文的原材料。
- **[writing-shape](./writing-shape/SKILL.md)**：拿一个装着原材料的 markdown 文件，一段一段把它塑造成文章，每一步就格式取舍讲道理。
- **[claude-handoff](./claude-handoff/SKILL.md)**：把当前会话交给一个新的后台 agent，让它立刻接手工作，通过 `claude --bg` 把交接摘要喂给它。由用户调用。
- **[setup-ts-deep-modules](./setup-ts-deep-modules/SKILL.md)**：把一个 TypeScript 仓库接上 dependency-cruiser，让每个包都是一个 deep module（深模块）：实现藏在子目录里，只能通过它的入口文件抵达，测试也通过这些入口来考验它。由用户调用。
