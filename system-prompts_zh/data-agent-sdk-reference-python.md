<!--
name: '数据：Agent SDK 参考 — Python'
description: Python Agent SDK 参考，包括安装、快速启动、通过 MCP 的自定义工具和挂钩
ccVersion: 2.1.83
-->
# 代理 SDK — Python

Claude Agent SDK 提供了一个更高级别的接口，用于构建具有内置工具、安全功能和代理功能的 AI 代理。

＃＃ 安装```bash
pip install claude-agent-sdk
```
---

## 快速入门```python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Explain this codebase",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
```
---

## 内置工具

|工具|描述 |
| --------- | ------------------------------------------------ |
|阅读 |读取工作区中的文件 |
|写 |创建新文件 |
|编辑|对现有文件进行精确编辑 |
|猛击 |执行 shell 命令 |
|全球|按模式查找文件 |
|查询 |按内容搜索文件 |
|网络搜索 |搜索网络获取信息 |
| Web获取|获取并分析网页 |
|向用户提问 |向用户询问澄清问题 |
|代理|生成子代理 |

---

## 主要接口

### `query()` — 简单的一次性使用

`query()` 函数是运行代理的最简单方法。它返回消息的异步迭代器。```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Explain this codebase",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    if isinstance(message, ResultMessage):
        print(message.result)
```
### `ClaudeSDKClient` — 完全控制

`ClaudeSDKClient` 提供对代理生命周期的完全控制。当您需要自定义工具、挂钩、流或中断执行的能力时，请使用它。```python
import anyio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, AssistantMessage, TextBlock

async def main():
    options = ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
    async with ClaudeSDKClient(options=options) as client:
        await client.query("Explain this codebase")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)

anyio.run(main)
```
`ClaudeSDKClient` 支持：

- **上下文管理器** (`async with`) 用于自动资源清理
- **`client.query(prompt)`** 向代理发送提示
- **`receive_response()`** 用于流式传输消息直至完成
- **`interrupt()`** 停止代理执行任务中
- **自定义工具所需**（通过 SDK MCP 服务器）

---

## 权限系统```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Refactor the authentication module",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Write"],
        permission_mode="acceptEdits"  # Auto-accept file edits
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```
权限模式：

- `"default"`：危险操作提示
- `"plan"`：仅计划，无执行
- `"acceptEdits"`：自动接受文件编辑
- `"bypassPermissions"`：跳过所有提示（谨慎使用）

---

## MCP（模型上下文协议）支持```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Open example.com and describe what you see",
    options=ClaudeAgentOptions(
        mcp_servers={
            "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```
---

## 钩子

使用回调函数通过钩子自定义代理行为：```python
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage

async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get('tool_input', {}).get('file_path', 'unknown')
    print(f"Modified: {file_path}")
    return {}

async for message in query(
    prompt="Refactor utils.py",
    options=ClaudeAgentOptions(
        permission_mode="acceptEdits",
        hooks={
            "PostToolUse": [HookMatcher(matcher="Edit|Write", hooks=[log_file_change])]
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```
工具生命周期事件（`PreToolUse`、`PostToolUse`、`PostToolUseFailure`）的挂钩回调输入包括 `agent_id` 和 `agent_type` 字段，允许挂钩识别哪个代理（主代理或子代理）触发了工具调用。

可用的挂钩事件：`PreToolUse`、`PostToolUse`、`PostToolUseFailure`、`UserPromptSubmit`、`Stop`、`SubagentStop`、`PreCompact`、`Notification`、 `SubagentStart`、`PermissionRequest`

---

## 常用选项

`query()` 采用顶级 `prompt` （字符串）和 `options` 对象 (`ClaudeAgentOptions`)：```python
async for message in query(prompt="...", options=ClaudeAgentOptions(...)):
```
|选项|类型 |描述 |
| ----------------------------------- | ------ | -------------------------------------------------------------------------------------- |
| `cwd` |字符串|文件操作的工作目录 |
| `allowed_tools` |列表 |代理可以使用的工具（例如，`["Read", "Edit", "Bash"]`）|
| `tools` |列表 |提供内置工具（限制默认设置）|
| `disallowed_tools` |列表 |明确禁止的工具 |
| `permission_mode` |字符串|如何处理权限提示|
| `mcp_servers` |字典 |要连接的 MCP 服务器 |
| `hooks` |字典 |用于自定义行为的挂钩 |
| `system_prompt` |字符串|自定义系统提示|
| `max_turns` |整数 |停止前最大代理转弯次数 |
| `max_budget_usd` |浮动|查询的最大预算（美元）|
| `model` |字符串|型号 ID（默认：由 CLI 确定）|
| `agents` |字典 |子代理定义 (`dict[str, AgentDefinition]`) |
| `output_format` |字典 |结构化输出模式|
| `thinking` |字典 |思维/推理控制 |
| `betas` |列表 |要启用的 Beta 功能（例如 `["context-1m-2025-08-07"]`）|
| `setting_sources` |列表 |要加载的设置（例如，`["project"]`）。默认值：无（无 CLAUDE.md 文件）|
| `env` |字典 |为会话设置的环境变量 |

---

## 消息类型```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage

async for message in query(
    prompt="Find TODO comments",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    if isinstance(message, ResultMessage):
        print(message.result)
        print(f"Stop reason: {message.stop_reason}")  # e.g., "end_turn", "max_turns"
    elif isinstance(message, SystemMessage) and message.subtype == "init":
        session_id = message.data.get("session_id")  # Capture for resuming later
```
`AssistantMessage` 包含每轮 `usage` 数据（与 Anthropic API 使用形状匹配的字典），用于跟踪成本：```python
from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage

async for message in query(prompt="...", options=ClaudeAgentOptions()):
    if isinstance(message, AssistantMessage) and message.usage:
        print(f"Input: {message.usage['input_tokens']}, Output: {message.usage['output_tokens']}")
```
在处理子代理任务事件时，类型化任务消息子类可提供更好的类型安全性：
- `TaskStartedMessage` — 注册子代理任务时发出
- `TaskProgressMessage` — 具有累积使用指标的实时进度更新
- `TaskNotificationMessage` — 任务完成通知

当速率限制状态转换时（例如，从 `allowed` 到 `allowed_warning` 或 `rejected`），会发出 `RateLimitEvent`。用它来警告用户或优雅地退出：```python
from claude_agent_sdk import query, ClaudeAgentOptions, RateLimitEvent

async for message in query(prompt="...", options=ClaudeAgentOptions()):
    if isinstance(message, RateLimitEvent):
        print(f"Rate limit status: {message.rate_limit_info.status}")
        if message.rate_limit_info.resets_at:
            print(f"Resets at: {message.rate_limit_info.resets_at}")
```
---

## 子代理```python
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

async for message in query(
    prompt="Use the code-reviewer agent to review this codebase",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Glob", "Grep", "Agent"],
        agents={
            "code-reviewer": AgentDefinition(
                description="Expert code reviewer for quality and security reviews.",
                prompt="Analyze code quality and suggest improvements.",
                tools=["Read", "Glob", "Grep"]
            )
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
```
---

## 错误处理```python
from claude_agent_sdk import query, ClaudeAgentOptions, CLINotFoundError, CLIConnectionError, ResultMessage

try:
    async for message in query(
        prompt="...",
        options=ClaudeAgentOptions(allowed_tools=["Read"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)
except CLINotFoundError:
    print("Claude Code CLI not found. Install with: pip install claude-agent-sdk")
except CLIConnectionError as e:
    print(f"Connection error: {e}")
```
---

## 会话历史记录

使用顶级函数检索过去的会话数据：```python
from claude_agent_sdk import list_sessions, get_session_messages

# List all past sessions (sync function — no await)
sessions = list_sessions()
for session in sessions:
    print(f"{session.session_id}: {session.cwd}")

# Get messages from a specific session (sync function — no await)
messages = get_session_messages(session_id="...")
for msg in messages:
    print(msg)
```
### 会话突变

重命名或标记会话（同步功能 - 无需等待）：```python
from claude_agent_sdk import rename_session, tag_session

# Rename a session
rename_session(session_id="...", title="My refactoring session")

# Tag a session (tags are Unicode-sanitized automatically)
tag_session(session_id="...", tag="experiment")

# Clear a tag
tag_session(session_id="...", tag=None)

# Optionally scope to a specific project directory
rename_session(session_id="...", title="New title", directory="/path/to/project")
```
---

## MCP 服务器管理

使用 `ClaudeSDKClient` 在运行时管理 MCP 服务器：```python
async with ClaudeSDKClient(options=options) as client:
    # Reconnect a disconnected MCP server
    await client.reconnect_mcp_server("my-server")

    # Toggle an MCP server on/off
    await client.toggle_mcp_server("my-server", enabled=False)

    # Get status of all MCP servers
    status = await client.get_mcp_status()  # returns McpStatusResponse
```
---

## 最佳实践

1. **始终指定 allowed_tools** — 显式列出代理可以使用哪些工具
2. **设置工作目录** — 文件操作始终指定`cwd`
3. **使用适当的权限模式** — 从 `"default"` 开始，仅在需要时升级
4. **处理所有消息类型** — 检查 `ResultMessage` 以获取代理输出
5. **限制 max_turns** — 通过合理的限制防止代理失控
