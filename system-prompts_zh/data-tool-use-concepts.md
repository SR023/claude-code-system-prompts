<!--
name: '数据：工具使用概念'
description: Claude API 工具使用的概念基础，包括工具定义、工具选择和最佳实践
ccVersion: 2.1.83
-->
# 工具使用概念

该文件涵盖了 Claude API 工具使用的概念基础。有关特定于语言的代码示例，请参阅 `python/`、`typescript/` 或其他语言文件夹。

## 用户定义的工具

### 工具定义结构

> **注意：** 使用 Tool Runner（测试版）时，工具架构会根据函数签名 (Python)、Zod 架构 (TypeScript)、带注释的类 (Java)、`jsonschema` 结构标记 (Go) 或 `BaseTool` 子类 (Ruby) 自动生成。下面的原始 JSON 模式格式适用于手动方法（包括 PHP 的 `BetaRunnableTool`，它将运行闭包包装在手写模式周围）或没有工具运行程序支持的 SDK。

每个工具都需要名称、描述和 JSON 架构作为其输入：```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City and state, e.g., San Francisco, CA"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit"
      }
    },
    "required": ["location"]
  }
}
```
**工具定义的最佳实践：**

- 使用清晰的描述性名称（例如，`get_weather`、`search_database`、`send_email`）
- 编写详细描述 - Claude使用这些来决定何时使用该工具
- 包括每个属性的描述
- 对于具有一组固定值的参数使用 `enum`
- 在`required`中标记真正需要的参数；将其他选项设置为默认值

---

### 工具选择选项

控制Claude何时使用工具：

|价值|行为 |
| --------------------------------- | -------------------------------------------------------- |
| `{"type": "auto"}` |Claude决定是否使用工具（默认）|
| `{"type": "any"}` |Claude必须使用至少一种工具 |
| `{"type": "tool", "name": "..."}` |Claude必须使用指定工具 |
| `{"type": "none"}` |Claude不能使用工具 |

任何 `tool_choice` 值还可以包含 `"disable_parallel_tool_use": true`，以强制 Claude 每个响应最多使用一个工具。默认情况下，Claude 可以在单个响应中请求多个工具调用。

---

### 工具运行器与手动循环

**工具运行程序（推荐）：** SDK 的工具运行程序自动处理代理循环 - 它调用 API、检测工具使用请求、执行工具函数、将结果反馈给 Claude，并重复直到 Claude 停止调用工具。适用于 Python、TypeScript、Java、Go、Ruby 和 PHP SDK（测试版）。 Python SDK 还提供 MCP 转换帮助程序 (`anthropic.lib.tools.mcp`)，用于转换 MCP 工具、提示和资源以与工具运行程序一起使用 - 有关详细信息，请参阅 `python/claude-api/tool-use.md`。

**手动代理循环：** 当您需要对循环进行细粒度控制时使用（例如，自定义日志记录、条件工具执行、人机交互批准）。循环直到`stop_reason == "end_turn"`，始终附加完整的`response.content`以保留tool_use块，并确保每个`tool_result`包含匹配的`tool_use_id`。

**服务器端工具的停止原因：** 使用服务器端工具（代码执行、网络搜索等）时，API 会运行服务器端采样循环。如果此循环达到 10 次迭代的默认限制，则响应将为 `stop_reason: "pause_turn"`。要继续，请重新发送用户消息和助手响应并发出另一个 API 请求 - 服务器将从中断处恢复。不要添加额外的用户消息，例如“继续”。 — API 检测到尾部 `server_tool_use` 块并知道自动恢复。```python
# Handle pause_turn in your agentic loop
if response.stop_reason == "pause_turn":
    messages = [
        {"role": "user", "content": user_query},
        {"role": "assistant", "content": response.content},
    ]
    # Make another API request — server resumes automatically
    response = client.messages.create(
        model="{{OPUS_ID}}", messages=messages, tools=tools
    )
```
设置 `max_continuations` 限制（例如 5）以防止无限循环。有关完整指南，请参阅：`https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons`

> **安全性：** 只要 Claude 请求，工具运行程序就会自动执行您的工具功能。对于具有副作用的工具（发送电子邮件、修改数据库、金融交易），请验证工具功能中的输入，并考虑要求确认破坏性操作。如果您需要在每次工具执行之前进行人机交互批准，请使用手动代理循环。

---

### 处理工具结果

当 Claude 使用工具时，响应包含 `tool_use` 块。您必须：

1. 使用提供的输入执行该工具
2. 将结果通过`tool_result`消息发回
3. 继续对话

**工具结果中的错误处理：** 当工具执行失败时，设置 `"is_error": true` 并提供信息性错误消息。Claude通常会承认错误，然后尝试不同的方法或要求澄清。

**多个工具调用：** Claude 可以在单个响应中请求多个工具。在继续之前处理所有这些问题 - 在一条 `user` 消息中发送回所有结果。

---

## 服务器端工具：代码执行

代码执行工具让 Claude 在安全的沙盒容器中运行代码。与用户定义的工具不同，服务器端工具在 Anthropic 的基础设施上运行 - 您无需在客户端执行任何操作。只需包含工具定义，Claude 即可处理剩下的事情。

### 关键事实

- 在隔离容器中运行（1 个 CPU、5 GiB RAM、5 GiB 磁盘）
- 无法访问互联网（完全沙盒）
- 预装数据科学库的 Python 3.11
- 容器可持续保存 30 天，并且可以跨请求重复使用
- 与网络搜索/网络获取工具一起使用时免费；每个组织每月免费使用 1,550 小时后 0.05 美元/小时

### 工具定义

该工具不需要模式 - 只需在 `tools` 数组中声明它：```json
{
  "type": "code_execution_20260120",
  "name": "code_execution"
}
```
Claude 自动获得对 `bash_code_execution`（运行 shell 命令）和 `text_editor_code_execution`（创建/查看/编辑文件）的访问权限。

### 预安装的 Python 库

- **数据科学**：pandas、numpy、scipy、scikit-learn、statsmodels
- **可视化**：matplotlib、seaborn
- **文件处理**：openpyxl、xlsxwriter、pillow、pypdf、pdfplumber、python-docx、python-pptx
- **数学**：sympy、mpmath
- **实用程序**：tqdm、python-dateutil、pytz、sqlite3

可以在运行时通过 `pip install` 安装其他软件包。

### 支持上传的文件类型

|类型 |扩展 |
| ------ | ---------------------------------- |
|数据| CSV、Excel (.xlsx/.xls)、JSON、XML |
|图片 | JPEG、PNG、GIF、WebP |
|文字| .txt、.md、.py、.js 等 |

### 容器重复利用

跨请求重用容器来维护状态（文件、已安装的包、变量）。从第一个响应中提取 `container_id` 并将其传递给后续请求。

### 响应结构

响应包含交错文本和工具结果块：

- `text` — Claude的解释
- `server_tool_use` — Claude在做什么
- `bash_code_execution_tool_result` — 代码执行输出（检查 `return_code` 是否成功/失败）
- `text_editor_code_execution_tool_result` — 文件操作结果

> **安全性：** 在将下载的文件写入磁盘之前，始终使用 `os.path.basename()` / `path.basename()` 清理文件名，以防止路径遍历攻击。将文件写入专用输出目录。

---

## 服务器端工具：Web 搜索和 Web Fetch

Web 搜索和 Web 获取让 Claude 可以搜索 Web 并检索页面内容。它们在服务器端运行 - 只需包含工具定义，Claude 就会自动处理查询、获取和结果处理。

### 工具定义```json
[
  { "type": "web_search_20260209", "name": "web_search" },
  { "type": "web_fetch_20260209", "name": "web_fetch" }
]
```
### 动态过滤（Opus 4.6 / Sonnet 4.6）

`web_search_20260209` 和 `web_fetch_20260209` 版本支持**动态过滤** - Claude 编写并执行代码以在搜索结果到达上下文窗口之前对其进行过滤，从而提高准确性和令牌效率。动态过滤内置于这些工具版本中并自动激活；您不需要单独声明 `code_execution` 工具或传递任何 beta 标头。```json
{
  "tools": [
    { "type": "web_search_20260209", "name": "web_search" },
    { "type": "web_fetch_20260209", "name": "web_fetch" }
  ]
}
```
不带动态过滤，之前的`web_search_20250305`版本也可用。

> **注意：** 仅当您的应用程序需要独立于 Web 搜索来执行代码以实现其自身目的（数据分析、文件处理、可视化）时，才包含独立的 `code_execution` 工具。将其与 `_20260209` Web 工具一起创建会创建第二个执行环境，这可能会混淆模型。

---

## 服务器端工具：编程工具调用

编程工具调用让 Claude 在代码中执行复杂的多工具工作流程，将中间结果保留在上下文窗口之外。 Claude 编写直接调用您的工具的代码，从而减少多步骤操作的令牌使用量。

如需完整文档，请使用 WebFetch：

- 网址：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling`

---

## 服务器端工具：工具搜索

工具搜索工具使 Claude 能够从大型库中动态发现工具，而无需将所有定义加载到上下文窗口中。当您有很多工具但只有少数工具与任何给定查询相关时很有用。

如需完整文档，请使用 WebFetch：

- 网址：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool`

---

## 工具使用示例

您可以直接在工具定义中提供示例工具调用，以演示使用模式并减少参数错误。这有助于 Claude 了解如何正确格式化工具输入，特别是对于具有复杂架构的工具。

如需完整文档，请使用 WebFetch：

- 网址：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use`

---

## 服务器端工具：计算机使用

计算机的使用让 Claude 能够与桌面环境（屏幕截图、鼠标、键盘）进行交互。它可以是人为托管的（服务器端，如代码执行）或自托管（您提供环境并在客户端执行操作）。

如需完整文档，请使用 WebFetch：

- 网址：`https://platform.claude.com/docs/en/agents-and-tools/computer-use/overview`

---

## 客户端工具：内存

记忆工具使Claude能够通过记忆文件目录存储和检索对话中的信息。 Claude 可以创建、读取、更新和删除会话之间保留的文件。

### 关键事实

- 客户端工具 — 您可以通过实施来控制存储
- 支持命令：`view`、`create`、`str_replace`、`insert`、`delete`、`rename`
- 对 `/memories` 目录中的文件进行操作
- Python、TypeScript 和 Java SDK 提供了用于实现内存后端的帮助器类/函数

> **安全性：**切勿将 API 密钥、密码、令牌或其他机密存储在内存文件中。请谨慎对待个人身份信息 (PII) — 在保留用户数据之前检查数据隐私法规（GDPR、CCPA）。参考实现没有内置访问控制；在多用户系统中，在工具处理程序中实现每用户内存目录和身份验证。

有关完整的实现示例，请使用 WebFetch：

- 文档：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md`

---

## 结构化输出

结构化输出限制 Claude 的响应遵循特定的 JSON 模式，从而保证有效、可解析的输出。这不是一个单独的工具 - 它增强了 Messages API 响应格式和/或工具参数验证。

有两个功能可用：- **JSON 输出** (`output_config.format`)：控制 Claude 的响应格式
- **严格工具使用** (`strict: true`)：保证有效的工具参数模式

**支持的型号：** {{OPUS_NAME}}、{{SONNET_NAME}} 和 {{HAIKU_NAME}}。旧模型（Claude Opus 4.5、Claude Opus 4.1）也支持结构化输出。

> **推荐：** 使用 `client.messages.parse()`，它会根据您的架构自动验证响应。直接使用`messages.create()`时，使用`output_config: {format: {...}}`。 `output_format` 便利参数也被一些 SDK 方法（例如 `.parse()`）接受，但 `output_config.format` 是规范的 API 级参数。

### JSON 架构限制

**支持：**

- 基本类型：对象、数组、字符串、整数、数字、布尔值、null
- `enum`、`const`、`anyOf`、`allOf`、`$ref`/`$def`
- 字符串格式：`date-time`、`time`、`date`、`duration`、`email`、`hostname`、`uri`、`ipv4`、 `ipv6`、`uuid`
- `additionalProperties: false`（所有对象都需要）

**不支持：**

- 递归模式
- 数值约束（`minimum`、`maximum`、`multipleOf`）
- 字符串约束（`minLength`、`maxLength`）
- 复杂的数组约束
- `additionalProperties` 设置为 `false` 以外的任何值

Python 和 TypeScript SDK 通过从发送到 API 的架构中删除不支持的约束并在客户端验证它们来自动处理不支持的约束。

### 重要提示

- **首次请求延迟**：新模式会产生一次性编译成本。具有相同架构的后续请求将使用 24 小时缓存。
- **拒绝**：如果 Claude 出于安全原因拒绝 (`stop_reason: "refusal"`)，则输出可能与您的架构不匹配。
- **令牌限制**：如果是 `stop_reason: "max_tokens"`，输出可能不完整。增加`max_tokens`。
- **不兼容**：引用（返回 400 错误）、消息预填充。
- **适用于**：批量 API、流式传输、令牌计数、扩展思维。

---

## 有效使用工具的技巧

1. **提供详细描述**：Claude 严重依赖描述来理解何时以及如何使用工具
2. **使用特定工具名称**：`get_current_weather` 优于 `weather`
3. **验证输入**：执行前始终验证工具输入
4. **优雅地处理错误**：返回信息丰富的错误消息，以便 Claude 能够适应
5. **限制工具数量**：太多的工具可能会混淆模型 - 保持集合的重点
6. **测试工具交互**：验证Claude在各种场景下正确使用工具

详细的工具使用文档，请使用WebFetch：

- 网址：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview`
