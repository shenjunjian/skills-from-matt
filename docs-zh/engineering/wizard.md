## 它做什么

`wizard` 生成一个交互式 bash 脚本，带着一个人一步一步走完一件手工流程：接第三方服务、跑一次性的迁移、把一个项目从状态 A 挪到状态 B。它逐个打开 URL，说该点哪里、该复制什么，接住返回的东西，写进 `.env` 文件和 GitHub Actions 的 secrets。

[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 写这个脚本，从不运行它。运行的是你自己，在你自己的机器上。一个 wizard 是一个程序，不是一份说明清单。脚本按顺序跑每一步，并保留你输入的值。你的那部分是点击、粘贴、按回车。

## 什么时候找它

你可以输入 `/wizard`，agent 也可以自己去找它。当它撞上一个你非做不可的步骤（一个它造不出来的 key、一个它点不了的 dashboard），它会给你写一个 wizard，而不是把这些步骤写进对话里，一滚就看不见。

当下一次挡在你前面的事是一趟 dashboard 之旅时，就找它：

| 情形 | wizard 做什么 |
| --- | --- |
| 新来的开发要配好六个服务，应用才能启动 | 按顺序打开每个 dashboard，接住那些 key，写进 `.env` 和 CI |
| 一次性的迁移需要按特定顺序拨几个开关 | 把不可逆的步骤排好，放在确认闸门后面 |
| 一个项目要一次性从状态 A 挪到状态 B | 带着走完这次转换，并报告它没能做到的部分 |
| 你正准备把这些步骤写进一个 README | 改写成可执行的那一版，某步过期时会显眼地失败 |

别为了*决定*要构建什么去找它；那件事的工具是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 和 [to-spec](https://aihero.dev/skills-to-spec)。

## 前置条件

生成时没有前置条件。它写出的 wizard 跑在 bash 上，当某个阶段要设 GitHub secret 或 variable 时会用 `gh`。如果 `gh` 缺失或者没登录，这一趟不会失败。那个阶段显示一条警告，结尾的摘要告诉你哪些要手工设。

## 阶段

一个 **stage**（阶段）是一块屏幕上的一项专注任务。脚本在每个 stage 之间清屏，所以一个超出屏幕的 stage 会把滚过去的那部分丢掉。你按依赖顺序写各个 stage，并设置 `TOTAL_STAGES`，它驱动进度显示。

这个 [Skill](https://www.aihero.dev/ai-coding-dictionary/skill)（技能）在写任何脚本之前先把流程圈定。它读仓库而不是凭空问：`.env*`、`docker-compose*`、框架配置，以及 `.github/workflows/` 里每一处 `secrets.*` / `vars.*` 引用。每一处这样的引用都是 wizard 必须产出的一个值。然后它把排好序的 stage 清单给你确认。只有你确认之后，它才把每个 stage 映射到人要走的那条确切路径（"Dashboard → Developers → API keys → Reveal test key → copy"）。在当前界面它不知道的地方，它问你或者去查文档，而不是凭空发明点击步骤。

对每一个要接住的价值，圈定阶段决定它去哪里：

| 去处 | 什么时候 |
| --- | --- |
| 只进 `.env` | 本地开发要它，CI 不要 |
| GitHub secret | CI 读它，而且它敏感 |
| GitHub variable | CI 读它，而且是公开的 |
| `.env` 和 secret 都写 | 本地开发和 CI 都要它 |
| 哪都不去 | 这个 stage 是纯动作：拨一个开关、升一个套餐 |

## 模板已经把交互解决了

[模板](https://github.com/mattpocock/skills/blob/main/skills/engineering/wizard/template.sh)自带整套体验：进度与剩余时间、确认闸门、跨平台的 URL 打开（含 WSL）、secret 的隐藏输入、幂等的 `.env` upsert、`gh secret` / `gh variable` 写入，以及一份把它不得不跳过的一切都列出来的收尾摘要。`STAGES` 标记以上全部是固定库，每个 wizard 里一字不差，永不手工编辑。因为库从来不变，每个 wizard 的行为都一样。每个 wizard 唯一的工作是圈定流程并写出它的各个 stage。

写一个 wizard 的 agent 不会端到端跑它，因为脚本会打开浏览器并等待人输入。它改成静态检查：`bash -n`、有 `shellcheck` 就用，再加一条追踪，确认每个值都落到圈定时说的那个地方，每个 `set_secret` 的名字都对应 CI 里一处真实的 `secrets.*` 引用。所以第一次运行是你的事，而那次运行就是测试。

## 默认是一次性的

| 你手上是什么 | 拿这个脚本怎么办 |
| --- | --- |
| 一次性的迁移、个人的环境搭建、一段你永远不会重复的转换 | 存到一个 scratch 或 `scripts/` 路径，跑掉它，删掉 |
| 下一个接手这个仓库的人也会需要的搭建路径 | 提交它并从 README 里链过去，这样他们跑脚本，而不是重新问一个 agent |

## 常见问题

**我的 API key 会进入模型的上下文吗？**

不会。agent 写一个脚本，它不跑脚本。你自己跑那个脚本，它用隐藏的终端输入接住 key，直接写进 `.env` 或 `gh secret`。wizard 是一个 CLI（命令行工具），模型没有跟它连上。这条只对 wizard 在运行时接住的值成立。如果你在圈定流程时把一个 key 粘进对话，那它就像任何别的一段粘贴文本一样在[上下文](https://www.aihero.dev/ai-coding-dictionary/context)里。

**我能退回去改一个我打错的值吗？**

运行中不能。没有后退按钮。各个 stage 向前走，第 3 步答错了意味着 Ctrl-C 重新跑。重跑很便宜，因为任何已经写进 `.env` 的值都会作为默认值还给你，所以你在答对的那些 stage 上直接按回车，只重打错的那个。发布那一周有用户提过这个，事情还开着："很喜欢！只有一件事，有没有办法退回去改已经输入的内容？"

**它知道我前面已经配好什么了吗？**

一部分，比发布时的反应假设的要少。它在提问之前先读仓库（你的 `.env` 文件、`docker-compose`、框架配置、CI 里的 `secrets.*` 引用），所以它圈定的是缺失的那些值，不像 README 那样从零开始。它不做的事是去查那个第三方服务。如果一个 key 已经在你的 `.env` 里，wizard 把它作为默认还给你，按回车就保留；如果你早先建好了 Stripe 账户但从没存过 key，wizard 还是会把你送到 dashboard 上去取。

**它在流程里坐在哪个位置，在 grilling 和 spec 之后吗？**

没什么特定位置。它是独立项，不是链上的一步。常见的猜法是 `/grill-with-docs → /to-spec → /wizard`，这个顺序没问题，但触发它的是冒出来的一件手工流程，而那可能发生在任何时刻：开工之前、构建中间、或者上线很久之后。它也能当发现工具用。圈定过程会在你正式投入之前找到一个任务藏起来的前置条件，比如那三个你根本没想到的 API key。

**它在 Claude Code 外面能用吗？**

脚本能。它是纯 bash，无论由哪个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（运行环境）生成，跑起来都一样。Skill 本身是 model-invoked（由模型调用）的，所以它到处都在列表里：在 Claude Code 里输入 `/wizard`，在 Codex 里输入 `$wizard`，或者直接描述你卡住的那次搭建。由模型调用这一点也让它避开了 [#693](https://github.com/mattpocock/skills/issues/693)，那个 bug 里 Claude 的桌面端和网页应用会把*用户调用*的 Skill 从 [model](https://www.aihero.dev/ai-coding-dictionary/model)（模型）的列表里省掉，并报成未安装。

**它以前不是用户调用的吗？**

是。现在它由模型调用，所以 agent 撞上你非做不可的步骤时会自己去找它。你以前能做的事没有一件失效。由模型调用让 agent 和你一样能唤起这个 Skill，所以 `/wizard` 的表现跟以前一样。这个改动修掉一种失败。以前，一个在构建中途撞上凭据墙的 agent 会把六个编号步骤倒进对话里让你照着手工做。现在它写一个 wizard。

**它以前在 `in-progress/` 里。现在在哪？**

`engineering/`，从 v1.2 起。它从 beta 桶里转正了，现在随插件一起发布，所以它跟其余转正的那一批一起来，不需要单独安装。转正时它的行为没变。

## 有效的话应该是这样

- 在任何脚本存在之前，你看到的是一份排好序的 stage 清单，每个 stage 会产出哪些值，并且要你确认。
- 每个 URL 都在向你要那一页上的值之前先打开。你不会被打断去粘贴一个从没让你去取的东西。
- Secret 是盲打的。没有任何敏感内容回声到你的滚动历史里。
- 每个 stage 装得下一屏。你还需要看的东西没有被滚走。
- Ctrl-C 再重跑能接上你停下的地方，把已经存好的值作为默认提供出来。
- 最后一屏列出它写了什么，另外单独列出它做不到、你必须手工完成的部分。

## 它在整体中的位置

`wizard` 是一个随时可以拿起来用的独立项，坐在自动化停下、必须由人来点击的那条线上。它最近的邻居是 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills)，因为两者都是为了把一个仓库推进到可用状态。那一个配置这套 Skill，`wizard` 则为其余一切生成一条搭建路径。它也跟 [implement](https://aihero.dev/skills-implement) 配对：当一次构建落地的功能需要凭据或一次手工切换时，wizard 覆盖那些只有人能做的步骤。拿不准此刻哪个 Skill 合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 给你路由。
