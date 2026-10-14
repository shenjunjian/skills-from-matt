# HTML 报告格式

架构评审渲染成一个自包含的 HTML 文件，放在操作系统临时目录里。Tailwind 和 Mermaid 都从 CDN 引入。图形化的示意图交给 Mermaid 处理更稳；偏编辑风格的视觉内容（质量图、剖面图）用手写的 div 和行内 SVG。两者混用：什么都塞给 Mermaid，做出来的东西会开始显得千篇一律。

## 骨架

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Architecture review for {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly:
         dashed seam lines, hand-drawn-feeling arrow heads, etc. */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## 页眉

仓库名、日期，加一份紧凑的图例：实心方块 = module（模块），虚线 = seam（接缝），红色箭头 = 泄漏，粗边深色方块 = deep module（深模块）。不要引言段落。直接进候选项。

## 候选项卡片

分量压在示意图上。文字克制、平实，直接使用 `/codebase-design` Skill 里的术语表词汇，不必铺垫。

每个候选项一个 `<article>`：

- **标题**：短，点明这次加深（例如"收拢 Order intake 这条流水线"）。
- **徽章行**：推荐强度（`Strong` = 翠绿，`Worth exploring` = 琥珀，`Speculative` = 石板灰），再加上一个依赖类别标签（`in-process`、`local-substitutable`、`ports & adapters`、`mock`）。
- **文件**：等宽字体列表，`font-mono text-sm`。
- **Before / After 示意图**：核心。两栏并排。式样见下。
- **问题**：一句话。痛在哪里。
- **方案**：一句话。会怎么改。
- **收益**：条目式，每条不超过 6 个词。例如"测试只打一个接口""定价逻辑不再外泄""删掉 4 个浅包装"。
- **ADR 提示框**（如适用）：琥珀色底框里的一行。

不要成段的解释。如果一张示意图非要有成段文字才看得懂，那就重画这张图。

## 示意图式样

挑贴合该候选项的那种式样。混着用。别让每张图都长一个样。多样本身就是目的。

### Mermaid graph（处理依赖与调用流的主力）

要点是"X 调 Y 调 Z，看看这团乱麻"时，用 Mermaid 的 `flowchart` 或 `graph`。外面套一张 Tailwind 样式过的卡片，别让它像是空降进来的。用 classDef 上色：泄漏的边标红，deep module 标深色。时序图很适合表达"之前：6 次往返；之后：1 次"。

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### 手写方块加箭头（Mermaid 的自动布局跟你较劲时）

模块用带边框和标签的 `<div>`。箭头用行内 SVG 的 `<line>` 或 `<path>`，绝对定位在相对容器之上。当你希望"之后"那张图看上去就是一个粗边框的 deep module、内部元素淡化处理时，就选这种，因为 Mermaid 渲染不出那种分量感。

### 剖面图（适合表现层层都浅）

用堆叠的横条（`h-12 border-l-4`）展示一次调用穿过的各个层。之前：6 层薄条，每层都没干什么事。之后：一条厚带，标上收拢后的职责。

### 质量图（适合表现"接口宽得跟实现一样"）

每个模块两个矩形：一个代表接口表面积，一个代表实现。之前：接口矩形几乎跟实现矩形一样高（浅）。之后：接口矩形很矮，实现矩形很高（深）。

### 调用图收拢

之前：一棵函数调用树，画成层层嵌套的方块。之后：同一棵树收拢成一个方块，如今变成内部的调用以淡色显示在方块里面。

## 风格指引

- 走编辑风格，别做成企业仪表盘。留白充足。标题可以用衬线（`font-serif` 与 stone/slate 配色搭配得很好）。
- 用色克制：一个主色（翠绿或靛蓝），外加红色表示泄漏、琥珀色表示警告。
- 示意图高度保持在 320px 左右，这样 before/after 并排摆放时不需要滚动。
- 示意图内部的模块标签用 `text-xs uppercase tracking-wider`，看上去像原理图，而不是 UI。
- 脚本只有 Tailwind CDN 和 Mermaid 的 ESM 引入这两处。报告其余部分是静态的：没有应用代码，除了 Mermaid 自己的渲染之外没有交互。

## 首选推荐小节

一张更大的卡片。候选项名字，一句为什么选它，再加一个指向该卡片的锚点链接。就这些。

## 语气

用平实的英文，简洁，但架构上的名词和动词一律直接取自 `/codebase-design` Skill。简洁不是放任漂移的借口。

**只用这些词：**module、interface、implementation、depth、deep、shallow、seam、adapter、leverage、locality。

**绝不替换成：**component、service、unit（指 module 时）· API、signature（指 interface 时）· boundary（指 seam 时）· layer、wrapper（指 module 时）。

**合风格的句式：**

- "Order intake module 很浅：interface 几乎等同于 implementation。"
- "Pricing 越过 seam 外泄。"
- "加深：一个 interface，一个测试落点。"
- "两个 adapter 撑得起这个 seam：生产走 HTTP，测试用内存实现。"

**收益条目**用术语表里的词点出所得：*"locality：缺陷集中到一个 module 里"*、*"leverage：一个 interface，N 处调用点"*、*"interface 收窄，implementation 把包装层吸收进去"*。不要写*"更好维护"*或*"代码更干净"*：这些词不在术语表里，站不住脚。

不打折扣，不清嗓子开场，不写"值得注意的是……"。一句话能写成条目的，就写成条目。一个条目能删的，就删。一个词不在 `/codebase-design` 的术语表里，先从表里挑一个能用的，再考虑造新词。
