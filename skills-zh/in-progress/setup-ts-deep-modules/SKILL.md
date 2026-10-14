---
name: setup-ts-deep-modules
description: 把一个 TypeScript 仓库接上 dependency-cruiser，让每个包都是一个 deep module（深模块）：实现藏在子目录里，只能通过它的入口文件抵达。由用户调用。
disable-model-invocation: true
---

# Setup TS Deep Modules（配置 TS 深模块）

让这个仓库里的每个包都成为**deep module**（深模块）：小而窄的接口后面藏着大量行为。一个包的公开面是它的**入口点**（包根目录上的那些文件），子目录里的一切都是隐藏的。本 Skill 装上 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 以及那些把入口点变成唯一通路的规则，然后证明这些规则真的会咬人。

关于词汇（deep module、interface、seam、depth），调用 Skill 工具、传入 "codebase-design"，并在全文使用它那套语言。

## The shape this enforces（它强制的形状）

```
src/packages/
  <name>/
    index.ts        ← an entry point (public). Import this from outside.
    client.ts       ← another entry point. Packages may expose SEVERAL.
    lib/            ← implementation: hidden from outside, free to import each other.
    tests/          ← co-located tests + fixtures (a subfolder, so private).
```

公开面是包的**根目录文件**，不是某个钦定的 `index.ts`。按约定实现放在 `lib/`、测试放在 `tests/`，于是每个包都是同样的两目录形状。规则本身则是通用的：*任何*子目录里的*任何*东西都是私有的，所以你永远不需要为了加一个目录去改配置。

四条规则，全部 `error`：

1. **入口点边界**：包外的代码（应用代码或另一个包）只能导入那个包的入口点（它的根目录文件），绝不能导入它子目录里的任何东西。
2. **包内自由**：一个包自己的文件之间随便互相导入。
3. **测试走入口点**：`<pkg>/tests/` 下的文件可以导入任何包的入口点，以及它们自己 `tests/` 里的 fixture，但绝不能导入任何包子目录的内部（连自己的也不行）。跨包的集成测试没问题，深层导入不行。
4. **禁止环**：不允许依赖环。

**要入口点，不要 barrel。** 因为公开面是*每一个*根目录文件，一个包就能暴露好几个小而专的入口点（`index.ts`、`client.ts`、`server.ts`），而不是把所有东西都从一个巨大的 `index.ts` 里漏出去。把整棵子树重新导出的 barrel 文件（桶文件）不推荐；入口点保持小，实现藏进子目录。

分层（哪些包可以依赖哪些包）是*另一个*问题，本 skill 在配置里留下一段带注释的桩，交给这个仓库自己去填。

## Steps（步骤）

### 1. 探测环境

- **包管理器**：`pnpm-lock.yaml` → pnpm，`yarn.lock` → yarn，`bun.lockb` → bun，否则 npm。下面每条命令都用它（`pnpm`/`yarn`/`npm run`/`bunx`）。
- **包根目录**：如果有 `src/`，就用 `src/packages`，否则用 `packages`。如果仓库已经有另一个明显的约定，先和用户确认这个选择。
- **已有配置**：查看有没有 `.dependency-cruiser.*` 文件。如果有，**不要**覆盖它：把那四条规则和 options 合并进去，并告诉用户你加了什么。

**完成标准：** 包管理器、包根目录、以及是否已有配置，三件事都清楚了。

### 2. 安装 dependency-cruiser

用探测出的包管理器，把 `dependency-cruiser` 装成 devDependency。

**完成标准：** `dependency-cruiser` 出现在 `devDependencies` 里。

### 3. 写配置

把 [`dependency-cruiser.config.cjs`](./dependency-cruiser.config.cjs) 复制到仓库根目录，命名为 `.dependency-cruiser.cjs`。把 `PACKAGES_ROOT` 设成第 1 步探测到的根目录。这些规则基于路径深度、不区分扩展名，所以别处不需要再改。

**完成标准：** `.dependency-cruiser.cjs` 存在，`PACKAGES_ROOT` 正确，四条 forbidden 规则都在。

### 4. 接进检查

- 加一个 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。
- 把它并进仓库的总检查命令，就是那个已经在跑 typecheck 的命令（比如 `check` / `ci` / `validate` 脚本）。**不要**动 `tsconfig`，也不要加路径别名。
- 如果没有总命令，就单独加 `lint:boundaries`，并告诉用户把它放进 CI。

**完成标准：** `lint:boundaries` 存在，并且和 typecheck 在同一条命令里跑起来。

### 5. 搭出示例包

创建一个纳入版本管理的 `<packages-root>/example/`，作为照着抄的模板：

- `index.ts` 是一个入口点。只导出一个函数，它委托给某个内部文件（这样这个包的*深*是看得见的，而不是一个透传的壳）。
- `lib/impl.ts`：一个位于**子目录**里的内部文件，被 `index.ts` 导入，从外部不可抵达。
- `tests/example.test.ts` 只导入 `../index`（一个入口点），并对那个公开函数做断言。

告诉用户这是个起步模板，可以抄，也可以删。

**完成标准：** 示例包存在，通过一个根目录入口点暴露它的行为，并把 `impl` 藏在子目录里。

### 6. 证明规则真的会咬人

这是整个 skill 的完成判据：一份碰上违规却不报错的配置毫无价值。

1. 跑 `lint:boundaries`。在干净的示例上它必须**通过**。
2. 临时往 `tests/example.test.ts` 里加一个深层导入（例如 `import { thing } from "../lib/impl"`）。再跑一次 `lint:boundaries`；它必须**失败**，报 `tests-through-entrypoints`。
3. 撤销那个深层导入。再跑一次，它必须**通过**。

**完成标准：** 你亲眼看到了一次通过、一次深层导入导致的失败、再一次通过。如果第 2 步没有失败，说明规则没接对，先修好再收工。

### 7. 把这个约定写进文档

在包目录里**就地**写一个 `README.md`（`<packages-root>/README.md`，紧挨着它管辖的那些包），内容涵盖：`src/packages/<name>/` 的布局（入口点在根目录，`lib/` 放实现，`tests/` 放测试）、"只能通过一个包的入口点（它的根目录文件）来导入"，以及怎么运行 `lint:boundaries`。**明确不推荐 barrel 文件**：暴露好几个小而专的入口点，而不是通过一个 index 把整棵子树重新导出。篇幅就是那段照着抄的目录片段，加上四条规则各一段。

然后从仓库的 agent 指令文件里给它加一个**上下文指针**（有 `CLAUDE.md` 就用它，否则用 `AGENTS.md`，两个都没有就创建 `AGENTS.md`）。一行就够，例如 `Packages are deep modules: see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` 正是这一行让 agent 自己去发现边界规则，而不是撞上它。

**完成标准：** `<packages-root>/README.md` 存在并且不推荐 barrel，仓库的 `CLAUDE.md`/`AGENTS.md` 链接到了它。

## Notes（备注）

- 配置里的 `$1` 反向引用（dependency-cruiser 的分组匹配）就是让一个包能触到它自己的内部、而外人不能的关键。别把它们拍平成一堆按包分开的规则。
- 公开还是私有由**深度**决定：包的根目录文件是入口点；任何子目录里的东西都是私有的。约定的子目录是 `lib/`（实现）和 `tests/`，但规则并没有把它们写死：任何子目录都是私有的，所以新增一个目录不需要改配置。加入口点就是加一个根目录文件（不用 barrel）。
- 包是**平铺**的：根目录下只有一层直接子目录。一个包自己的内部可以嵌套多深都行；但一个包不能包含另一个包。
- 用 `.cjs`（不是 `.js`），这样配置的 `module.exports` 在 `"type": "module"` 的仓库里也能用。
