<!--
name: '数据：Agent SDK 参考 — TypeScript'
description: TypeScript Agent SDK 参考，包括安装、快速启动、自定义工具和挂钩
ccVersion: 2.1.83
-->
# 代理 SDK — TypeScript

Claude Agent SDK 提供了一个更高级别的接口，用于构建具有内置工具、安全功能和代理功能的 AI 代理。

＃＃ 安装```bash
npm install @anthropic-ai/claude-agent-sdk
```
---

## 快速入门```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Explain this codebase",
  options: { allowedTools: ["Read", "Glob", "Grep"] },
})) {
  if ("result" in message) {
    console.log(message.result);
  }
}
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

## 权限系统```typescript
for await (const message of query({
  prompt: "Refactor the authentication module",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits",
  },
})) {
  if ("result" in message) console.log(message.result);
}
```
权限模式：

- `"default"`：危险操作提示
- `"plan"`：仅计划，无执行
- `"acceptEdits"`：自动接受文件编辑
- `"dontAsk"`：不提示 — **拒绝**任何未预先批准的内容（不是自动批准模式）
- `"bypassPermissions"`：跳过所有提示（选项中需要`allowDangerouslySkipPermissions: true`）

---

## MCP（模型上下文协议）支持```typescript
for await (const message of query({
  prompt: "Open example.com and describe what you see",
  options: {
    mcpServers: {
      playwright: { command: "npx", args: ["@playwright/mcp@latest"] },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```
### 进程内 MCP 工具

您可以使用 `tool()` 和 `createSdkMcpServer` 定义在进程中运行的自定义工具：```typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const myTool = tool("my-tool", "Description", { input: z.string() }, async (args) => {
  return { content: [{ type: "text", text: "result" }] };
});

const server = createSdkMcpServer({ name: "my-server", tools: [myTool] });

// Pass to query
for await (const message of query({
  prompt: "Use my-tool to do something",
  options: { mcpServers: { myServer: server } },
})) {
  if ("result" in message) console.log(message.result);
}
```
---

## 钩子```typescript
import { query, HookCallback } from "@anthropic-ai/claude-agent-sdk";
import { appendFileSync } from "fs";

const logFileChange: HookCallback = async (input) => {
  const filePath = (input as any).tool_input?.file_path ?? "unknown";
  appendFileSync(
    "./audit.log",
    `${new Date().toISOString()}: modified ${filePath}\n`,
  );
  return {};
};

for await (const message of query({
  prompt: "Refactor utils.py to improve readability",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits",
    hooks: {
      PostToolUse: [{ matcher: "Edit|Write", hooks: [logFileChange] }],
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```
工具生命周期事件（`PreToolUse`、`PostToolUse`、`PostToolUseFailure`）的挂钩事件输入包括 `agent_id` 和 `agent_type` 字段，允许挂钩识别哪个代理（主代理或子代理）触发了工具调用。

可用的挂钩事件：`PreToolUse`、`PostToolUse`、`PostToolUseFailure`、`Notification`、`UserPromptSubmit`、`SessionStart`、`SessionEnd`、`Stop`、 `SubagentStart`、`SubagentStop`、`PreCompact`、`PermissionRequest`、`Setup`、`TeammateIdle`、`TaskCompleted`、`ConfigChange`、`Elicitation`、 `ElicitationResult`、`WorktreeCreate`、`WorktreeRemove`、`InstructionsLoaded`

---

## 常用选项

`query()` 采用顶级 `prompt` （字符串）和 `options` 对象：```typescript
query({ prompt: "...", options: { ... } })
```
|选项|类型 |描述 |
| ----------------------------------- | ------ | -------------------------------------------------------------------------------------- |
| `cwd` |字符串|文件操作的工作目录 |
| `allowedTools` |数组|代理可以使用的工具（例如，`["Read", "Edit", "Bash"]`）|
| `tools` |数组\|预设|提供内置工具（`string[]` 或 `{type:'preset', preset:'claude_code'}`）|
| `disallowedTools` |数组|明确禁止的工具 |
| `permissionMode` |字符串|如何处理权限提示|
| `allowDangerouslySkipPermissions` |布尔 |必须是 `true` 才能使用 `permissionMode: "bypassPermissions"` |
| `mcpServers` |对象|要连接的 MCP 服务器 |
| `hooks` |对象|用于自定义行为的挂钩 |
| `systemPrompt` |字符串\|预设|自定义系统提示符（`string`或`{type:'preset', preset:'claude_code', append?:string}`）|
| `maxTurns` |数量 |停止前最大代理转弯次数 |
| `maxBudgetUsd` |数量 |查询的最大预算（美元）|
| `model` |字符串|型号 ID（默认：由 CLI 确定）|
| `agents` |对象|子代理定义 (`Record<string, AgentDefinition>`) |
| `outputFormat` |对象|结构化输出模式|
| `thinking` |对象|思维/推理控制 |
| `betas` |数组|要启用的 Beta 功能（例如 `["context-1m-2025-08-07"]`）|
| `settingSources` |数组|要加载的设置（例如，`["project"]`）。默认值：无（无 CLAUDE.md 文件）|
| `env` |对象|为会话设置的环境变量 |
| `agentProgressSummaries` |布尔 |启用人工智能定期生成 `task_progress` 事件的进度摘要 |

---

## 子代理```typescript
for await (const message of query({
  prompt: "Use the code-reviewer agent to review this codebase",
  options: {
    allowedTools: ["Read", "Glob", "Grep", "Agent"],
    agents: {
      "code-reviewer": {
        description: "Expert code reviewer for quality and security reviews.",
        prompt: "Analyze code quality and suggest improvements.",
        tools: ["Read", "Glob", "Grep"],
        // Optional: skills, mcpServers for subagent customization
      },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```
---

## 消息类型```typescript
for await (const message of query({
  prompt: "Find TODO comments",
  options: { allowedTools: ["Read", "Glob", "Grep"] },
})) {
  if ("result" in message) {
    console.log(message.result);
    console.log(`Stop reason: ${message.stop_reason}`); // e.g., "end_turn", "tool_use", "max_tokens"
  } else if (message.type === "system" && message.subtype === "init") {
    const sessionId = message.session_id; // Capture for resuming later
  }
}
```
还为子代理操作发出与任务相关的系统消息：
- `task_started` — 注册子代理任务时发出
- `task_progress` — 实时进度更新，包括累积使用指标、工具计数和持续时间（通过 `summary` 字段启用 `agentProgressSummaries` 选项以定期由 AI 生成摘要）
- `task_notification` — 任务完成通知（包括用于与原始工具调用关联的 `tool_use_id`）

---

## 会话历史记录

检索过去的会话数据：```typescript
import { listSessions, getSessionMessages, getSessionInfo } from "@anthropic-ai/claude-agent-sdk";

// List all past sessions (supports pagination via limit/offset)
const sessions = await listSessions({ limit: 20, offset: 0 });
for (const session of sessions) {
  console.log(`${session.sessionId}: ${session.cwd} (tag: ${session.tag})`);
}

// Get metadata for a single session
const sessionId = sessions[0]?.sessionId;
const info = await getSessionInfo(sessionId);
console.log(info.tag, info.createdAt);

// Get messages from a specific session (supports pagination via limit/offset)
const messages = await getSessionMessages(sessionId, { limit: 50, offset: 0 });
for (const msg of messages) {
  console.log(msg);
}
```
### 会话突变

重命名、标记或分叉会话：```typescript
import { renameSession, tagSession, forkSession } from "@anthropic-ai/claude-agent-sdk";

// Rename a session
await renameSession(sessionId, "My refactoring session");

// Tag a session
await tagSession(sessionId, "experiment");

// Clear a tag
await tagSession(sessionId, null);

// Fork a session — branch a conversation from a specific point
const { sessionId: forkedId } = await forkSession(sessionId);
```
---

## MCP 服务器管理

在运行时管理正在运行的查询的 MCP 服务器：```typescript
// Reconnect a disconnected MCP server
await queryHandle.reconnectMcpServer("my-server");

// Toggle an MCP server on/off
await queryHandle.toggleMcpServer("my-server", false);  // (name, enabled) — both required

// Get status of ALL configured MCP servers — returns an ARRAY
const statuses: McpServerStatus[] = await queryHandle.mcpServerStatus();
for (const s of statuses) {
  console.log(s.name, s.scope, s.tools.length, s.error);
}
```
---

## 最佳实践

1. **始终指定 allowedTools** — 明确列出代理可以使用哪些工具
2. **设置工作目录** — 文件操作始终指定`cwd`
3. **使用适当的权限模式** — 从 `"default"` 开始，仅在需要时升级
4. **处理所有消息类型** — 检查 `result` 属性以获取代理输出
5. **限制 maxTurns** — 通过合理的限制防止代理失控
