<!--
name: '数据：流参考 — Python'
description: Python 流参考，包括同步/异步流和处理不同的内容类型
ccVersion: 2.1.78
-->
# 流媒体 — Python

## 快速入门```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```
### 异步```python
async with async_client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```
---

## 处理不同的内容类型

克劳德可能会返回文本、思维块或工具使用。妥善处理每一项：

> **Opus 4.6：** 使用 `thinking: {type: "adaptive"}`。在旧型号上，请改用 `thinking: {type: "enabled", budget_tokens: N}`。```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "Analyze this problem"}]
) as stream:
    for event in stream:
        if event.type == "content_block_start":
            if event.content_block.type == "thinking":
                print("\n[Thinking...]")
            elif event.content_block.type == "text":
                print("\n[Response:]")

        elif event.type == "content_block_delta":
            if event.delta.type == "thinking_delta":
                print(event.delta.thinking, end="", flush=True)
            elif event.delta.type == "text_delta":
                print(event.delta.text, end="", flush=True)
```
---

## 使用工具进行流式传输

Python 工具运行程序当前返回完整消息。如果您需要使用工具进行每个令牌流式传输，请在手动循环中对单个 API 调用使用流式传输：```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    tools=tools,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    response = stream.get_final_message()
    # Continue with tool execution if response.stop_reason == "tool_use"
```
---

## 获取最终消息```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Hello"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    # Get full message after streaming
    final_message = stream.get_final_message()
    print(f"\n\nTokens used: {final_message.usage.output_tokens}")
```
---

## 带有进度更新的流媒体```python
def stream_with_progress(client, **kwargs):
    """Stream a response with progress updates."""
    total_tokens = 0
    content_parts = []

    with client.messages.stream(**kwargs) as stream:
        for event in stream:
            if event.type == "content_block_delta":
                if event.delta.type == "text_delta":
                    text = event.delta.text
                    content_parts.append(text)
                    print(text, end="", flush=True)

            elif event.type == "message_delta":
                if event.usage and event.usage.output_tokens is not None:
                    total_tokens = event.usage.output_tokens

        final_message = stream.get_final_message()

    print(f"\n\n[Tokens used: {total_tokens}]")
    return "".join(content_parts)
```
---

## 流中的错误处理```python
try:
    with client.messages.stream(
        model="{{OPUS_ID}}",
        max_tokens=64000,
        messages=[{"role": "user", "content": "Write a story"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
except anthropic.APIConnectionError:
    print("\nConnection lost. Please retry.")
except anthropic.RateLimitError:
    print("\nRate limited. Please wait and retry.")
except anthropic.APIStatusError as e:
    print(f"\nAPI error: {e.status_code}")
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

1. **始终刷新输出** — 使用 `flush=True` 立即显示令牌
2. **处理部分响应** — 如果流中断，您的内容可能不完整
3. **跟踪代币使用情况** — `message_delta` 事件包含使用信息
4. **使用超时** — 为您的应用程序设置适当的超时
5. **默认流式传输** — 使用 `.get_final_message()` 即使在流式传输时也能获得完整响应，为您提供超时保护，无需处理单个事件
