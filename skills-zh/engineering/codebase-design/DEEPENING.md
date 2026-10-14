# 深化

在依赖条件明确的前提下，如何安全地深化一团浅层 module（模块）。本文假定你已使用 [SKILL.md](SKILL.md) 里的词汇：**module**、**interface**、**seam**、**adapter**。

## 依赖分类

评估一个深化候选项时，先给它的依赖分类。分类决定深化后的 module 如何跨 seam 测试。

### 1. 进程内

纯计算，内存状态，无 I/O。永远可以深化：把这些 module 合并，直接通过新的 interface 测试。不需要 adapter。

### 2. 可本地替换

有本地测试替身的依赖（用 PGLite 顶替 Postgres，内存文件系统）。替身存在就可以深化。深化后的 module 用测试套件里运行的替身来测。seam 在内部；module 的外部 interface 上没有 port。

### 3. 远端但自有（Ports & Adapters）

跨网络边界、你自己拥有的服务（微服务、内部 API）。在 seam 处定义一个 **port**（接口）。深层 module 持有逻辑；传输层作为 **adapter** 注入。测试用内存 adapter。生产用 HTTP/gRPC/消息队列 adapter。

推荐表述的形状：*"在 seam 处定义一个 port，生产环境实现一个 HTTP adapter，测试环境实现一个内存 adapter，这样逻辑集中在一个深层 module 里，尽管它部署在网络两端。"*

### 4. 真正的外部依赖（Mock）

你控制不了的第三方服务（Stripe、Twilio 等）。深化后的 module 把外部依赖作为注入的 port 接收；测试提供一个 mock adapter。

## seam 纪律

- **一个 adapter 意味着假设的 seam，两个 adapter 才是真实的。** 除非至少两个 adapter 站得住脚（通常是生产加测试），否则不要引入 port。只有一个 adapter 的 seam 只是间接层。
- **内部 seam 与外部 seam。** 一个深层 module 可以有内部 seam（对其实现私有，供自己的测试使用），也可以有 interface 上的外部 seam。不要因为测试用到了内部 seam，就把它通过 interface 暴露出去。

## 测试策略：替换，而不是叠加

- 一旦深化后的 module 的 interface 上有了测试，浅层 module 上的旧单元测试就成了废物；删掉它们。
- 在深化后的 module 的 interface 上写新测试。**interface 就是测试面**。
- 测试通过 interface 断言可观察的结果，不断言内部状态。
- 测试应当能在内部重构之后存活下来，因为它们描述行为，不描述实现。一个测试如果因为实现改变就得跟着改，那它测的就是 interface 之外的东西。
