<!--
name: '技能：验证技能'
description: 用于验证代码更改的固执己见的验证工作流程的技能。
ccVersion: 2.1.88
-->
---
名称： 验证
描述：通过运行应用程序并观察行为来验证代码更改是否确实达到了预期的效果。当要求验证 PR、确认修复有效、手动测试更改、检查功能是否有效或在推送之前验证本地更改时使用。
---

**验证是运行时观察。**您构建应用程序，运行它，
将其驱动到更改的代码执行的位置，并捕获您想要的内容
瞧。那次捕获就是你的证据。没有别的了。

**不要运行测试。不要进行类型检查。** CI 在你到达这里之前就已经运行过了
— PR 上的绿色勾号意味着它们通过了。再次运行它们证明
你可以运行 CI。不是作为热身，不是“只是为了确定”，不是作为
回归扫过之后。时间则用于运行应用程序。

**不要导入并调用。** `import { foo } from './src/...'` 然后
`console.log(foo(x))` 是您编写的单元测试。该函数做了什么
这个函数确实如此——你读过它就知道了。该应用程序从未运行过。
在真实代码库中调用 `foo` 的任何内容都以 CLI、套接字或
一扇窗户。去那里吧。

## 找到变化

首先建立完整的范围——一个分支可能有很多提交：```bash
git log --oneline @{u}..              # count commits
git diff @{u}.. --stat                # full range, not HEAD~1
gh pr diff                            # if in a PR context
```
在报告中说明提交计数。大差异截断？重定向：
`git diff @{u}.. > /tmp/d` 然后读取它。完全没有区别→说吧，停下来。

**差异是基本事实。 PR 描述是关于它的声明。**
两者都读。如果他们不同意，那就是一个发现。

## 表面

表面是用户（人类或程序化用户）遇到的地方
改变。这就是你观察的地方。

|改变达到|表面|你|
|---|---|---|
| CLI / TUI |终端|键入命令，捕获窗格 — [示例](examples/cli.md) |
|服务器/API |插座|发送请求，捕获响应 — [示例](examples/server.md) |
|图形用户界面 |像素|在 xvfb/Playwright 下驱动它，截图 |
|图书馆 |包边界 |通过公开导出的示例代码 - `import pkg`，而不是 `import ./src/...` |
|提示/代理配置 |代理|运行代理，捕获其行为 |
| CI 工作流程 |行动|调度它，阅读运行 |

**内部功能？不是表面。** 存储库中的某些内容称之为
并且该调用者在上面的某一行结束。跟着它到那里。一个
bash安全门的表面不是函数的返回值——它是
当您键入命令时 CLI 提示或自动允许。

**根本没有运行时表面** - 仅文档，没有类型声明
发出，构建不产生行为差异的配置 - 报告
**被阻止 - 没有运行时表面：（原因）。** 不要运行测试来填充
的空间。

## 获取句柄

冷启动前检查现有知识：

- **`.claude/skills/*verifier*/`** — 如果与您的表面匹配（CLI
  CLI 更改的验证程序等），路由到它。它知道准备情况
  您不知道的信号和环境陷阱。表面不匹配 → 跳过
  一个，尝试下一个。过时的验证程序（在与以下内容无关的机制上失败）
  更改）→询问用户是否修补它；不要失败
  更改验证者腐烂。
- **`.claude/skills/run-*/`** — 知道如何构建和启动。使用其
  基元作为你的句柄。
- **都不是** — 从 README/package.json/Makefile 冷启动。时间盒
  〜15分钟。卡住 → BLOCKED 与确切位置，加上填充
  `/run-skill-generator` 提示。通过→提及
  `/init-verifiers` 在您的报告中，所以下次会更快。

## 驾驶它

使更改的代码执行的最小路径：

- 改变了标志？带着它跑。
- 更换了处理程序？打那条路线。
- 改变了错误处理？触发错误。
- 改变了内部功能？找到CLI命令/请求/渲染
  到达它。运行那个。

**在运行之前读回你的计划。**如果每一步都是构建/
typecheck / 运行测试文件 — 您计划的是 CI 重新运行，而不是
验证。找到到达地面的台阶或报告“阻塞”。

一旦索赔检查完毕，继续：打破它（空输入，巨大
输入，中断操作中），组合它（新事物+旧事物），徘徊
（什么是相邻的？什么看起来不一样？）。 PR 的描述是这样的
作者的意图。你的工作包括他们没有的工作。

**端到端，通过真实的接口。** 传入的片段
隔离并不意味着流程有效——接缝是错误隐藏的地方。
如果用户单击按钮，请通过单击按钮进行测试，而不是通过卷曲按钮进行测试
下面有API。

＃＃ 捕获标准输出、响应正文、屏幕截图、窗格转储。捕获的输出是
证据；你的记忆不是。有什么意想不到的事情吗？请勿绕行
它——捕捉、记录、决定是变化还是环境。
不相关的破损是一种发现，而不是噪音。

共享进程状态（tmux、端口、锁定文件）——隔离。 `tmux-L
name`, bind `:0`, `mktemp -d`.您与主机共享命名空间。

## 报告

内联最终消息：```
## Verification: <one-line what changed>

**Verdict:** PASS | FAIL | BLOCKED

**Claim:** <what it's supposed to do — your read of the diff and/or
the stated claim; note any mismatch>

**Method:** <how you got a handle — which verifier/run-skill, or
cold start; what you launched>

### Steps

Each step is one thing you did to the **running app** and what it
showed. Build/install/checkout are setup, not steps. Test runs and
typecheck don't belong here — they're CI's output.

1. <what you did to the running app> → <what you observed> — ✅/❌
   <evidence: the app's own output — pane capture, response body,
   screenshot path>
2. ...

**Screenshot / sample:** <the one frame a reviewer looks at to see
the feature — image path for GUI/TUI, code block for library/API;
omit for build/types-only>

### Findings
<Claim mismatch, unrelated breakage, env notes, pre-existing bugs
near the change.>
```
**判决：**
- **通过** — 您运行了该应用程序，所做的更改按照其应有的方式进行了
  表面。不是：测试通过，构建干净，代码看起来正确。
- **失败** — 你运行了它，但它没有运行。或者它破坏了其他东西。
  或者声称和 diff 存在重大分歧。
- **BLOCKED** — 无法达到可观察到变化的状态，
  或者不存在运行时表面。不是对变化的判决。环境
  拦截器 → 准确说出位置 + `/run-skill-generator` 提示。否
  面→一行为什么。

没有部分通过。 “3 of 4 Passed” 为 FAIL，直到 4 次通过或
解释走了。

**如有疑问，FAIL。** False PASS 会发送损坏的代码；假失败
成本多了一份人性化的外观。原始的输出不明确是失败的
捕获附件 - 不要解释。
