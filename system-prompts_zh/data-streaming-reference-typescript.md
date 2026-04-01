<!--
name: '数据：流媒体参考 — TypeScript'
description: TypeScript 流媒体参考，包括基本流媒体和处理不同的内容类型
ccVersion: 2.1.78
-->
# 流媒体 — TypeScript

## 快速入门```typescript
const stream = client.messages.stream({
  model: "{{OPUS_ID}}",
  max_tokens: 64000,
  messages: [{ role: "user", content: "Write a story" }],
});

for await (const event of stream) {
  if (
    event.type === "content_block_delta" &&
    event.delta.type === "text_delta"
  ) {
    process.stdout.write(event.delta.text);
  }
}
```
---

## 处理不同的内容类型

> **Opus 4.6：** 使用 `thinking: {type: "adaptive"}`。在旧型号上，请改用 `thinking: {type: "enabled", budget_tokens: N}`。```typescript
const stream = client.messages.stream({
  model: "{{OPUS_ID}}",
  max_tokens: 64000,
  thinking: { type: "adaptive" },
  messages: [{ role: "user", content: "Analyze this problem" }],
});

for await (const event of stream) {
  switch (event.type) {
    case "content_block_start":
      switch (event.content_block.type) {
        case "thinking":
          console.log("\n[Thinking...]");
          break;
        case "text":
          console.log("\n[Response:]");
          break;
      }
      break;
    case "content_block_delta":
      switch (event.delta.type) {
        case "thinking_delta":
          process.stdout.write(event.delta.thinking);
          break;
        case "text_delta":
          process.stdout.write(event.delta.text);
          break;
      }
      break;
  }
}
```
---

## 使用工具进行流式传输 (Tool Runner)

使用带有 `stream: true` 的工具运行器。外循环迭代工具运行器迭代（消息），内循环处理流事件：```typescript
import Anthropic from "@anthropic-ai/sdk";
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";

const client = new Anthropic();

const getWeather = betaZodTool({
  name: "get_weather",
  description: "Get current weather for a location",
  inputSchema: z.object({
    location: z.string().describe("City and state, e.g., San Francisco, CA"),
  }),
  run: async ({ location }) => `72°F and sunny in ${location}`,
});

const runner = client.beta.messages.toolRunner({
  model: "{{OPUS_ID}}",
  max_tokens: 64000,
  tools: [getWeather],
  messages: [
    { role: "user", content: "What's the weather in Paris and London?" },
  ],
  stream: true,
});

// Outer loop: each tool runner iteration
for await (const messageStream of runner) {
  // Inner loop: stream events for this iteration
  for await (const event of messageStream) {
    switch (event.type) {
      case "content_block_delta":
        switch (event.delta.type) {
          case "text_delta":
            process.stdout.write(event.delta.text);
            break;
          case "input_json_delta":
            // Tool input being streamed
            break;
        }
        break;
    }
  }
}
```
---

## 获取最终消息```typescript
const stream = client.messages.stream({
  model: "{{OPUS_ID}}",
  max_tokens: 64000,
  messages: [{ role: "user", content: "Hello" }],
});

for await (const event of stream) {
  // Process events...
}

const finalMessage = await stream.finalMessage();
console.log(`Tokens used: ${finalMessage.usage.output_tokens}`);
```
---

## 流事件类型

|事件类型 |描述 |当它发生时 |
| -------------------- | ------------------------ | | --------------------------------- |
| `message_start` |包含消息元数据 |一次在开始 |
| `content_block_start` |新内容块开始 |当 text/tool_use 块开始时 |
| `content_block_delta` |增量内容更新 |对于每个令牌/块|
| `content_block_stop` |内容块完成 |当一个块完成时 |
| `message_delta` |消息级更新 |包含`stop_reason`，用法|
| `message_stop` |留言完毕 |一次在最后|

## 最佳实践

1. **始终刷新输出** — 使用 `process.stdout.write()` 立即显示
2. **处理部分响应** — 如果流中断，您的内容可能不完整
3. **跟踪代币使用情况** — `message_delta` 事件包含使用信息
4. **使用 `finalMessage()`** — 即使在流式传输时也能获取完整的 `Anthropic.Message` 对象。不要将 `.on()` 事件包装在 `new Promise()` 中 — `finalMessage()` 在内部处理所有完成/错误/中止状态
5. **Web UI 的缓冲区** — 考虑在渲染之前缓冲一些令牌，以避免过多的 DOM 更新
6. **使用 `stream.on("text", ...)` 进行增量** — `text` 事件仅提供增量字符串，比手动过滤 `content_block_delta` 事件更简单
7. **对于具有流式传输的代理循环** — 请参阅 tool-use.md 中的 [Streaming Manual Loop](./tool-use.md#streaming-manual-loop) 部分，了解如何将 `stream()` + `finalMessage()` 与工具使用循环结合起来

## 原始 SSE 格式

如果使用原始 HTTP（不是 SDK），流将返回服务器发送的事件：```
event: message_start
data: {"type":"message_start","message":{"id":"msg_...","type":"message",...}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn"},"usage":{"output_tokens":12}}

event: message_stop
data: {"type":"message_stop"}
```
