<!--
name: '数据：Claude API 参考 — TypeScript'
description: TypeScript SDK 参考，包括安装、客户端初始化、基本请求、思考和多轮对话
ccVersion: 2.1.83
-->
# 克劳德 API — TypeScript

## 安装```bash
npm install @anthropic-ai/sdk
```
## 客户端初始化```typescript
import Anthropic from "@anthropic-ai/sdk";

// Default (uses ANTHROPIC_API_KEY env var)
const client = new Anthropic();

// Explicit API key
const client = new Anthropic({ apiKey: "your-api-key" });
```
---

## 基本消息请求```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  messages: [{ role: "user", content: "What is the capital of France?" }],
});
// response.content is ContentBlock[] — a discriminated union. Narrow by .type
// before accessing .text (TypeScript will error on content[0].text without this).
for (const block of response.content) {
  if (block.type === "text") {
    console.log(block.text);
  }
}
```
---

## 系统提示```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  system:
    "You are a helpful coding assistant. Always provide examples in Python.",
  messages: [{ role: "user", content: "How do I read a JSON file?" }],
});
```
---

## 愿景（图像）

### 网址```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "url", url: "https://example.com/image.png" },
        },
        { type: "text", text: "Describe this image" },
      ],
    },
  ],
});
```
### Base64```typescript
import fs from "fs";

const imageData = fs.readFileSync("image.png").toString("base64");

const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "base64", media_type: "image/png", data: imageData },
        },
        { type: "text", text: "What's in this image?" },
      ],
    },
  ],
});
```
---

## 提示缓存

**缓存是前缀匹配** - 前缀中任何位置的任何字节更改都会使其后面的所有内容无效。有关放置模式、架构指南（冻结系统提示、确定性工具顺序、在何处放置易失性内容）以及静默无效器审核清单，请阅读 `shared/prompt-caching.md`。

### 自动缓存（推荐）

使用顶级 `cache_control` 自动缓存请求中的最后一个可缓存块：```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  cache_control: { type: "ephemeral" }, // auto-caches the last cacheable block
  system: "You are an expert on this large document...",
  messages: [{ role: "user", content: "Summarize the key points" }],
});
```
### 手动缓存控制

为了进行细粒度控制，请将 `cache_control` 添加到特定内容块：```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  system: [
    {
      type: "text",
      text: "You are an expert on this large document...",
      cache_control: { type: "ephemeral" }, // default TTL is 5 minutes
    },
  ],
  messages: [{ role: "user", content: "Summarize the key points" }],
});

// With explicit TTL (time-to-live)
const response2 = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  system: [
    {
      type: "text",
      text: "You are an expert on this large document...",
      cache_control: { type: "ephemeral", ttl: "1h" }, // 1 hour TTL
    },
  ],
  messages: [{ role: "user", content: "Summarize the key points" }],
});
```
### 验证缓存命中```typescript
console.log(response.usage.cache_creation_input_tokens); // tokens written to cache (~1.25x cost)
console.log(response.usage.cache_read_input_tokens);     // tokens served from cache (~0.1x cost)
console.log(response.usage.input_tokens);                // uncached tokens (full cost)
```
如果 `cache_read_input_tokens` 在重复的相同前缀请求中为零，则说明静默无效器正在工作 - `Date.now()` 或系统提示中的 UUID、非确定性密钥排序或变化的工具集。有关完整审核表，请参阅 `shared/prompt-caching.md`。

---

## 延伸思考

> **Opus 4.6 和 Sonnet 4.6：** 使用适应性思维。 `budget_tokens` 在 Opus 4.6 和 Sonnet 4.6 上均已弃用。
> **旧型号：** 使用 `thinking: {type: "enabled", budget_tokens: N}`（必须 < `max_tokens`，最小 1024）。```typescript
// Opus 4.6: adaptive thinking (recommended)
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  thinking: { type: "adaptive" },
  output_config: { effort: "high" }, // low | medium | high | max
  messages: [
    { role: "user", content: "Solve this math problem step by step..." },
  ],
});

for (const block of response.content) {
  if (block.type === "thinking") {
    console.log("Thinking:", block.thinking);
  } else if (block.type === "text") {
    console.log("Response:", block.text);
  }
}
```
---

## 错误处理

使用 SDK 的类型化异常类 — 永远不要检查字符串匹配的错误消息：```typescript
import Anthropic from "@anthropic-ai/sdk";

try {
  const response = await client.messages.create({...});
} catch (error) {
  if (error instanceof Anthropic.BadRequestError) {
    console.error("Bad request:", error.message);
  } else if (error instanceof Anthropic.AuthenticationError) {
    console.error("Invalid API key");
  } else if (error instanceof Anthropic.RateLimitError) {
    console.error("Rate limited - retry later");
  } else if (error instanceof Anthropic.APIError) {
    console.error(`API error ${error.status}:`, error.message);
  }
}
```
所有类都使用类型化 `status` 字段扩展 `Anthropic.APIError`。从最具体到最不具体进行检查。请参阅 [shared/error-codes.md](../../shared/error-codes.md) 以获取完整的错误代码参考。

---

## 多轮对话

API 是无状态的——每次发送完整的对话历史记录。使用 `Anthropic.MessageParam[]` 键入消息数组：```typescript
const messages: Anthropic.MessageParam[] = [
  { role: "user", content: "My name is Alice." },
  { role: "assistant", content: "Hello Alice! Nice to meet you." },
  { role: "user", content: "What's my name?" },
];

const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  messages: messages,
});
```
**规则：**

- 允许连续的相同角色消息 - API 将它们组合成一个回合
- 第一条消息必须是 `user`
- 对所有 API 数据结构使用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Message`、`Anthropic.Tool` 等）——不要重新定义等效接口

---

### 压缩（长对话）

> **Beta、Opus 4.6 和 Sonnet 4.6。** 当对话接近 200K 上下文窗口时，压缩会自动总结服务器端较早的上下文。 API返回`compaction`块；您必须在后续请求中将其传回 - 附加 `response.content`，而不仅仅是文本。```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const messages: Anthropic.Beta.BetaMessageParam[] = [];

async function chat(userMessage: string): Promise<string> {
  messages.push({ role: "user", content: userMessage });

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "{{OPUS_ID}}",
    max_tokens: 16000,
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }],
    },
  });

  // Append full content — compaction blocks must be preserved
  messages.push({ role: "assistant", content: response.content });

  const textBlock = response.content.find(
    (b): b is Anthropic.Beta.BetaTextBlock => b.type === "text",
  );
  return textBlock?.text ?? "";
}

// Compaction triggers automatically when context grows large
console.log(await chat("Help me build a Python web scraper"));
console.log(await chat("Add support for JavaScript-rendered pages"));
console.log(await chat("Now add rate limiting and error handling"));
```
---

## 停止原因

响应中的 `stop_reason` 字段指示模型停止生成的原因：

|价值|意义|
| ---------------- | --------------------------------------------------------------------------- |
| `end_turn` |克劳德很自然地完成了回应|
| `max_tokens` |达到 `max_tokens` 限制 — 增加限制或使用流媒体 |
| `stop_sequence` |按下自定义停止顺序 |
| `tool_use` |克劳德想要调用一个工具——执行它并继续 |
| `pause_turn` |模型已暂停并可以恢复（代理流）|
| `refusal` |出于安全原因，克劳德拒绝了——输出可能与模式不匹配 |

---

## 成本优化策略

### 1. 对重复上下文使用提示缓存```typescript
// Automatic caching (simplest — caches the last cacheable block)
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  cache_control: { type: "ephemeral" },
  system: largeDocumentText, // e.g., 50KB of context
  messages: [{ role: "user", content: "Summarize the key points" }],
});

// First request: full cost
// Subsequent requests: ~90% cheaper for cached portion
```
### 2. 在请求之前使用令牌计数```typescript
const countResponse = await client.messages.countTokens({
  model: "{{OPUS_ID}}",
  messages: messages,
  system: system,
});

const estimatedInputCost = countResponse.input_tokens * 0.000005; // $5/1M tokens
console.log(`Estimated input cost: $${estimatedInputCost.toFixed(4)}`);
```
