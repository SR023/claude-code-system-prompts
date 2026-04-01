<!--
name: '技能：使用 Claude API 进行构建'
description: 与 Claude 一起构建 LLM 支持的应用程序的主要路由指南，包括语言检测、界面选择和架构概述
ccVersion: 2.1.83
-->
# 与 Claude 一起构建 LLM 支持的应用程序

这项技能可以帮助您与 Claude 一起构建 LLM 支持的应用程序。根据您的需求选择正确的界面，检测项目语言，然后阅读相关的特定语言文档。

## 默认值

除非用户另有要求：

对于 Claude 模型版本，请使用 {{OPUS_NAME}}，您可以通过确切的模型字符串 `{{OPUS_ID}}` 访问它。对于任何复杂的事情，请默认使用适应性思维（`thinking: {type: "adaptive"}`）。最后，请默认对任何可能涉及长输入、长输出或高 `max_tokens` 的请求进行流式传输 - 它可以防止请求超时。如果您不需要处理单个流事件，请使用 SDK 的 `.get_final_message()` / `.finalMessage()` 帮助程序获取完整响应

---

## 语言检测

在阅读代码示例之前，请确定用户正在使用哪种语言：

1. **查看项目文件** 以推断语言：

   - `*.py`、`requirements.txt`、`pyproject.toml`、`setup.py`、`Pipfile` → **Python** — 从 `python/` 读取
   - `*.ts`、`*.tsx`、`package.json`、`tsconfig.json` → **TypeScript** — 从 `typescript/` 读取
   - `*.js`、`*.jsx`（不存在 `.ts` 文件）→ **TypeScript** — JS 使用相同的 SDK，从 `typescript/` 读取
   - `*.java`、`pom.xml`、`build.gradle` → **Java** — 从 `java/` 读取
   - `*.kt`、`*.kts`、`build.gradle.kts` → **Java** — Kotlin 使用 Java SDK，从 `java/` 读取
   - `*.scala`、`build.sbt` → **Java** — Scala 使用 Java SDK，从 `java/` 读取
   - `*.go`、`go.mod` → **执行** — 从 `go/` 读取
   - `*.rb`、`Gemfile` → **Ruby** — 从 `ruby/` 读取
   - `*.cs`、`*.csproj` → **C#** — 从 `csharp/` 读取
   - `*.php`、`composer.json` → **PHP** — 从 `php/` 读取

2. **如果检测到多种语言**（例如，Python 和 TypeScript 文件）：

   - 检查用户当前文件或问题涉及哪种语言
   - 如果仍然不明确，请询问：“我检测到 Python 和 TypeScript 文件。您使用哪种语言进行 Claude API 集成？”

3. **如果无法推断语言**（空项目、没有源文件或不受支持的语言）：

   - 使用 AskUserQuestion 和选项：Python、TypeScript、Java、Go、Ruby、cURL/raw HTTP、C#、PHP
   - 如果 AskUserQuestion 不可用，则默认使用 Python 示例并注意：“显示 Python 示例。如果您需要其他语言，请告诉我。”

4. **如果检测到不支持的语言**（Rust、Swift、C++、Elixir 等）：

   - 建议来自 `curl/` 的 cURL/raw HTTP 示例，并注意社区 SDK 可能存在
   - 提供 Python 或 TypeScript 示例作为参考实现

5. **如果用户需要 cURL/原始 HTTP 示例**，请从 `curl/` 中读取。

### 特定于语言的功能支持|语言 |工具运行者|代理SDK |笔记|
| ---------- | ----------- | --------- | -------------------------------------------------- |
|蟒蛇 |是（测试版）|是的 |全面支持 — `@beta_tool` 装饰器 |
|打字稿 |是（测试版）|是的 |全面支持 — `betaZodTool` + Zod |
|爪哇 |是（测试版）|没有 | Beta 工具与带注释的类一起使用 |
|去 |是（测试版）|没有 | `toolrunner` 包装中的 `BetaToolRunner` |
|红宝石 |是（测试版）|没有 | `BaseTool` + `tool_runner` 测试版 |
|卷曲 |不适用 |不适用 |原始 HTTP，无 SDK 功能 |
| C# |没有 |没有 |官方SDK |
| PHP |是（测试版）|没有 | `BetaRunnableTool` + `toolRunner()` |

---

## 我应该使用哪种表面？

> **从简单开始。** 默认为满足您需求的最简单层。单个 API 调用和工作流程可处理大多数用例 - 仅当任务真正需要开放式、模型驱动的探索时才使用代理。

|使用案例|等级 |推荐表面|为什么 |
| ----------------------------------------------------------- | ---------------- | ---------------------------------- | --------------------------------------- |
|分类、归纳、提取、问答 |单一法学硕士通话 | **Claude·API** |一个请求，一个响应 |
|批处理或嵌入 |单一法学硕士通话 | **Claude·API** |专业端点|
|具有代码控制逻辑的多步管道 |工作流程| **Claude API + 工具使用** |您精心安排循环 |
|使用您自己的工具定制代理 |代理| **Claude API + 工具使用** |最大的灵活性|
|具有文件/网络/终端访问权限的 AI 代理 |代理| **代理SDK** |内置工具、安全性和 MCP 支持 |
|代理编码助手 |代理| **代理SDK** |专为该用例而设计|
|想要内置权限和护栏 |代理| **代理SDK** |包括安全功能|

> **注意：** Agent SDK 适用于您需要开箱即用的内置文件/Web/终端工具、权限和 MCP 的情况。如果您想使用自己的工具构建代理，Claude API 是正确的选择 - 使用工具运行程序进行自动循环处理，或使用手动循环进行细粒度控制（批准门、自定义日志记录、条件执行）。

### 决策树```
What does your application need?

1. Single LLM call (classification, summarization, extraction, Q&A)
   └── Claude API — one request, one response

2. Does Claude need to read/write files, browse the web, or run shell commands
   as part of its work? (Not: does your app read a file and hand it to Claude —
   does Claude itself need to discover and access files/web/shell?)
   └── Yes → Agent SDK — built-in tools, don't reimplement them
       Examples: "scan a codebase for bugs", "summarize every file in a directory",
                 "find bugs using subagents", "research a topic via web search"

3. Workflow (multi-step, code-orchestrated, with your own tools)
   └── Claude API with tool use — you control the loop

4. Open-ended agent (model decides its own trajectory, your own tools)
   └── Claude API agentic loop (maximum flexibility)
```
### 我应该建立一个代理吗？

在选择代理级别之前，请检查所有四个标准：

- **复杂性** — 任务是否是多步骤且难以提前完全指定？ （例如，“将此设计文档转换为 PR”与“从此 PDF 中提取标题”）
- **价值** — 结果是否证明更高的成本和延迟是合理的？
- **生存能力** — Claude有能力胜任这种任务类型吗？
- **错误成本** — 错误能否被发现并恢复？ （测试、审查、回滚）

如果对其中任何一个的答案是“否”，请停留在更简单的层（单个调用或工作流程）。

---

## 架构

一切都经过 `POST /v1/messages`。工具和输出约束是这个单一端点的功能，而不是单独的 API。

**用户定义的工具** — 您定义工具（通过装饰器、Zod 模式或原始 JSON），SDK 的工具运行程序负责调用 API、执行函数并循环，直到 Claude 完成。为了完全控制，您可以手动编写循环。

**服务器端工具** — 在 Anthropic 基础设施上运行的 Anthropic 托管工具。代码执行完全在服务器端（在 `tools` 中声明，Claude 自动运行代码）。计算机的使用可以是服务器托管或自托管。

**结构化输出** — 约束消息 API 响应格式 (`output_config.format`) 和/或工具参数验证 (`strict: true`)。推荐的方法是 `client.messages.parse()`，它会根据您的架构自动验证响应。注意：旧的 `output_format` 参数已弃用；在 `messages.create()` 上使用 `output_config: {format: {...}}`。

**支持端点** - 批次 (`POST /v1/messages/batches`)、文件 (`POST /v1/files`)、令牌计数和模型（`GET /v1/models`、`GET /v1/models/{id}` - 实时功能/上下文窗口发现）馈入或支持消息 API 请求。

---

## 当前型号（缓存：2026-02-17）

|型号|型号 ID |背景 |输入$/1M |产出 $/1M |
| ----------------- | ------------------- | -------------- | ---------- | ----------- |
|Claude作品 4.6 | `claude-opus-4-6` | 200K（1M 测试版）| 5.00 美元 | 25.00 美元 |
|Claude十四行诗 4.6 | `claude-sonnet-4-6` | 200K（1M 测试版）| $3.00 | 15.00 美元 |
|Claude俳句 4.5 | `claude-haiku-4-5` | 20万| 1.00 美元 | 5.00 美元 |

**始终使用 `{{OPUS_ID}}` 除非用户明确指定不同的型号。**这是不可协商的。请勿使用 `{{SONNET_ID}}`、`{{PREV_SONNET_ID}}` 或任何其他型号，除非用户字面意思是“使用十四行诗”或“使用俳句”。切勿因成本而降级——这是用户的决定，而不是您的决定。

**重要：仅使用上表中的确切型号 ID 字符串 - 它们按原样完整。请勿附加日期后缀。** 例如，使用 `claude-sonnet-4-5`，切勿使用 `claude-sonnet-4-5-20250514` 或您可能从训练数据中回忆起的任何其他日期后缀变体。如果用户请求表中没有的旧型号（例如“opus 4.5”、“sonnet 3.7”），请阅读 `shared/models.md` 以获取确切的 ID — 不要自己构造一个。

注意：如果上面的任何模型字符串对您来说看起来不熟悉，这是可以预料的 - 这仅意味着它们是在您的训练数据截止后发布的。请放心，它们是真实的模型；我们不会那样惹你生气的。**实时能力查找：** 上表已缓存。当用户询问“X 的上下文窗口是什么”、“X 是否支持愿景/思维/努力”或“哪些模型支持 Y”时，请查询模型 API (`client.models.retrieve(id)` / `client.models.list()`) — 请参阅 `shared/models.md` 了解字段参考和功能过滤器示例。

---

## 思考与努力（快速参考）

**Opus 4.6 — 适应性思维（推荐）：** 使用 `thinking: {type: "adaptive"}`。Claude动态地决定思考的时间和程度。不需要 `budget_tokens` — `budget_tokens` 在 Opus 4.6 和 Sonnet 4.6 上已弃用，不得使用。适应性思维还会自动启用交错思维（不需要 beta 标题）。 **当用户要求“扩展思考”、“思考预算”或 `budget_tokens` 时：始终使用带有 `thinking: {type: "adaptive"}` 的 Opus 4.6。思维的固定代币预算的概念已被弃用——适应性思维取而代之。请勿使用 `budget_tokens` 且请勿切换到旧型号。**

**努力参数（GA，无测试标头）：** 通过 `output_config: {effort: "low"|"medium"|"high"|"max"}`（在 `output_config` 内部，而不是顶层）控制思考深度和总体代币支出。默认为`high`（相当于省略）。 `max` 仅适用于 Opus 4.6。适用于 Opus 4.5、Opus 4.6 和 Sonnet 4.6。在 Sonnet 4.5 / Haiku 4.5 上会出错。与适应性思维相结合，实现最佳的成本质量权衡。使用 `low` 执行子代理或简单任务； `max` 最深层的推理。

**十四行诗 4.6：** 支持适应性思维 (`thinking: {type: "adaptive"}`)。 `budget_tokens` 在 Sonnet 4.6 上已弃用 - 请改用自适应思维。

**旧型号（仅在明确要求时）：** 如果用户特别要求 Sonnet 4.5 或其他旧型号，请使用 `thinking: {type: "enabled", budget_tokens: N}`。 `budget_tokens` 必须小于 `max_tokens`（最小 1024）。切勿仅仅因为用户提到 `budget_tokens` 就选择较旧的型号 - 请使用具有自适应思维的 Opus 4.6。

---

## 压缩（快速参考）

**Beta、Opus 4.6 和 Sonnet 4.6。** 对于可能超过 200K 上下文窗口的长时间运行的对话，启用服务器端压缩。当接近触发阈值（默认值：150K 令牌）时，API 会自动总结早期上下文。需要测试标头 `compact-2026-01-12`。

**重要：** 每次都将 `response.content`（不仅仅是文本）附加到您的消息中。必须保留响应中的压缩块——API 使用它们来替换下一个请求的压缩历史记录。仅提取文本字符串并附加，这将默默地失去压缩状态。

有关代码示例，请参阅 `{lang}/claude-api/README.md`（压缩部分）。通过 `shared/live-sources.md` 中的 WebFetch 获取完整文档。

---

## 提示缓存（快速参考）

**前缀匹配。** 前缀中任何位置的任何字节更改都会使其后面的所有内容无效。渲染顺序为 `tools` → `system` → `messages`。首先保留稳定的内容（冻结的系统提示、确定性工具列表），将易变的内容（时间戳、每个请求 ID、变化的问题）放在最后一个 `cache_control` 断点之后。当您不需要细粒度的放置时，**顶级自动缓存**（`messages.create()` 上的 `cache_control: {type: "ephemeral"}`）是最简单的选择。每个请求最多 4 个断点。最小可缓存前缀约为 1024 个令牌 - 较短的前缀将不会自动缓存。

**使用 `usage.cache_read_input_tokens` 进行验证** - 如果重复请求中的值为零，则说明静默无效器正在工作（系统提示中的 `datetime.now()`、未排序的 JSON、不同的工具集）。

有关布局模式、架构指南和静默无效器审核清单：请阅读 `shared/prompt-caching.md`。特定于语言的语法：`{lang}/claude-api/README.md`（提示缓存部分）。

---

## 阅读指南

检测语言后，根据用户需求读取相关文件：

### 快速任务参考

**单文本分类/摘要/提取/问答：**
→ 只读 `{lang}/claude-api/README.md`

**聊天UI或实时回复显示：**
→ 读取 `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**长时间运行的对话（可能超出上下文窗口）：**
→ 阅读 `{lang}/claude-api/README.md` — 参见压缩部分

**提示缓存/优化缓存/“为什么我的缓存命中率低”：**
→ 读取`shared/prompt-caching.md` + `{lang}/claude-api/README.md`（提示缓存部分）

**函数调用/工具使用/代理：**
→ 读取 `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`

**批处理（非延迟敏感）：**
→ 读取 `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**跨多个请求上传文件：**
→ 读取 `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**具有内置工具的代理（文件/网络/终端）：**
→ 读取 `{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`

### Claude API（完整文件参考）

阅读 **特定于语言的 Claude API 文件夹** (`{language}/claude-api/`)：

1. **`{language}/claude-api/README.md`** — **先阅读本文。** 安装、快速启动、常见模式、错误处理。
2. **`shared/tool-use-concepts.md`** — 当用户需要函数调用、代码执行、内存或结构化输出时读取。涵盖概念基础。
3. **`{language}/claude-api/tool-use.md`** — 阅读特定于语言的工具使用代码示例（工具运行程序、手动循环、代码执行、内存、结构化输出）。
4. **`{language}/claude-api/streaming.md`** — 在构建增量显示响应的聊天 UI 或界面时阅读。
5. **`{language}/claude-api/batches.md`** — 离线处理许多请求时读取（对延迟不敏感）。以 50% 的成本异步运行。
6. **`{language}/claude-api/files-api.md`** — 在跨多个请求发送同一文件而不重新上传时读取。
7. **`shared/prompt-caching.md`** — 添加或优化提示缓存时阅读。涵盖前缀稳定性设计、断点放置以及静默使缓存失效的反模式。
8. **`shared/error-codes.md`** — 在调试 HTTP 错误或实施错误处理时读取。
9. **`shared/live-sources.md`** — 用于获取最新官方文档的 WebFetch URL。

> **注意：** 对于 Java、Go、Ruby、C#、PHP 和 cURL — 这些都有一个文件，每个文件涵盖所有基础知识。根据需要读取该文件以及 `shared/tool-use-concepts.md` 和 `shared/error-codes.md`。

### 代理SDK

阅读 **特定于语言的 Agent SDK 文件夹** (`{language}/agent-sdk/`)。 Agent SDK 仅适用于 **Python 和 TypeScript**。1. **`{language}/agent-sdk/README.md`** — 安装、快速启动、内置工具、权限、MCP、挂钩。
2. **`{language}/agent-sdk/patterns.md`** — 自定义工具、挂钩、子代理、MCP 集成、会话恢复。
3. **`shared/live-sources.md`** — 当前 Agent SDK 文档的 WebFetch URL。

---

## 何时使用 WebFetch

在以下情况下使用 WebFetch 获取最新文档：

- 用户询问“最新”或“当前”信息
- 缓存数据似乎不正确
- 用户询问此处未涵盖的功能

实时文档 URL 位于 `shared/live-sources.md` 中。

## 常见陷阱- 将文件或内容传递到 API 时不要截断输入。如果内容太长而无法放入上下文窗口，请通知用户并讨论选项（分块、摘要等），而不是默默地截断。
- **Opus 4.6 / Sonnet 4.6 思考：** 使用 `thinking: {type: "adaptive"}` — 不要使用 `budget_tokens`（在 Opus 4.6 和 Sonnet 4.6 上已弃用）。对于较旧的型号，`budget_tokens` 必须小于 ZXQMASK0091QX​​Z（最小 1024）。如果你弄错了，这会抛出一个错误。
- **Opus 4.6 预填充已删除：** 助理消息预填充（最后一个助理轮预填充）在 Opus 4.6 上返回 400 错误。使用结构化输出 (`output_config.format`) 或系统提示指令来控制响应格式。
- **`max_tokens` 默认值：** 不要低估 `max_tokens` — 达到上限会截断输出，需要重试。对于非流式请求，默认为 `~16000`（在 SDK HTTP 超时下保留响应）。对于流请求，默认为 `~64000`（超时不是问题，所以给模型留出空间）。仅当您有明确的理由时才降低：分类（`~256`）、成本上限或故意缩短产出。
- **128K 输出令牌：** Opus 4.6 支持高达 128K `max_tokens`，但 SDK 需要对如此大的值进行流式传输以避免 HTTP 超时。将 `.stream()` 与 `.get_final_message()` / `.finalMessage()` 结合使用。
- **工具调用 JSON 解析（Opus 4.6）：** Opus 4.6 可能会在工具调用 `input` 字段中产生不同的 JSON 字符串转义（例如，Unicode 或正斜杠转义）。始终使用 `json.loads()` / `JSON.parse()` 解析工具输入 - 切勿对序列化输入进行原始字符串匹配。
- **结构化输出（所有型号）：** 使用 `output_config: {format: {...}}` 而不是 `messages.create()` 上已弃用的 `output_format` 参数。这是一般 API 更改，不是 4.6 特有的。
- **不要重新实现 SDK 功能：** SDK 提供高级帮助程序 - 使用它们而不是从头开始构建。具体来说：使用`stream.finalMessage()`而不是在`new Promise()`中包装`.on()`事件；使用类型化异常类（`Anthropic.RateLimitError` 等）而不是字符串匹配错误消息；使用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.Message` 等）而不是重新定义等效接口。
- **不要为 SDK 数据结构定义自定义类型：** SDK 导出所有 API 对象的类型。使用 `Anthropic.MessageParam` 表示消息，使用 `Anthropic.Tool` 表示工具定义，使用 `Anthropic.ToolUseBlock` / `Anthropic.ToolResultBlockParam` 表示工具结果，使用 `Anthropic.Message` 表示响应。定义您自己的 `interface ChatMessage { role: string; content: unknown }` 会重复 SDK 已提供的内容并失去类型安全性。
- **报告和文档输出：** 对于生成报告、文档或可视化的任务，代码执行沙箱已预安装 `python-docx`、`python-pptx`、`matplotlib`、`pillow` 和 `pypdf`。 Claude 可以生成格式化文件（DOCX、PDF、图表）并通过文件 API 返回它们 - 考虑将此用于“报告”或“文档”类型请求，而不是纯标准输出文本。
