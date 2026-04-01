<!--
name: '数据：Claude API 参考 — cURL'
description: 用于与 cURL 或原始 HTTP 一起使用的 Claude API 的原始 API 参考
ccVersion: 2.1.83
-->
# Claude API — cURL / 原始 HTTP

当用户需要原始 HTTP 请求或使用没有官方 SDK 的语言时，请使用这些示例。

＃＃ 设置```bash
export ANTHROPIC_API_KEY="your-api-key"
```
---

## 基本消息请求```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "messages": [
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }'
```
### 解析响应

使用 `jq` 从 JSON 响应中提取字段。请勿使用 `grep`/`sed` —
JSON 字符串可以包含任何字符，正则表达式解析将在引号处中断，
转义符或多行内容。```bash
# Capture the response, then extract fields
response=$(curl -s https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{"model":"{{OPUS_ID}}","max_tokens":16000,"messages":[{"role":"user","content":"Hello"}]}')

# Print the first text block (-r strips the JSON quotes)
echo "$response" | jq -r '.content[0].text'

# Read usage fields
input_tokens=$(echo "$response" | jq -r '.usage.input_tokens')
output_tokens=$(echo "$response" | jq -r '.usage.output_tokens')

# Read stop reason (for tool-use loops)
stop_reason=$(echo "$response" | jq -r '.stop_reason')

# Extract all text blocks (content is an array; filter to type=="text")
echo "$response" | jq -r '.content[] | select(.type == "text") | .text'
```
---

## 流媒体（SSE）```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 64000,
    "stream": true,
    "messages": [{"role": "user", "content": "Write a haiku"}]
  }'
```
响应是服务器发送的事件流：```
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
---

## 工具使用```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "tools": [{
      "name": "get_weather",
      "description": "Get current weather for a location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "City name"}
        },
        "required": ["location"]
      }
    }],
    "messages": [{"role": "user", "content": "What is the weather in Paris?"}]
  }'
```
当 Claude 用 `tool_use` 块响应时，将结果发回：```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "tools": [{
      "name": "get_weather",
      "description": "Get current weather for a location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "City name"}
        },
        "required": ["location"]
      }
    }],
    "messages": [
      {"role": "user", "content": "What is the weather in Paris?"},
      {"role": "assistant", "content": [
        {"type": "text", "text": "Let me check the weather."},
        {"type": "tool_use", "id": "toolu_abc123", "name": "get_weather", "input": {"location": "Paris"}}
      ]},
      {"role": "user", "content": [
        {"type": "tool_result", "tool_use_id": "toolu_abc123", "content": "72°F and sunny"}
      ]}
    ]
  }'
```
---

## 提示缓存

将 `cache_control` 放在稳定前缀的最后一个块上。请参阅 `shared/prompt-caching.md` 了解放置模式和静默无效器审核清单。```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "system": [
      {"type": "text", "text": "<large shared prompt...>", "cache_control": {"type": "ephemeral"}}
    ],
    "messages": [{"role": "user", "content": "Summarize the key points"}]
  }'
```
对于 1 小时 TTL：`"cache_control": {"type": "ephemeral", "ttl": "1h"}`。请求正文中的顶级 `"cache_control"` 自动放置在最后一个可缓存块上。通过响应 `usage.cache_creation_input_tokens` / `usage.cache_read_input_tokens` 字段验证命中。

---

## 延伸思考

> **Opus 4.6 和 Sonnet 4.6：** 使用适应性思维。 `budget_tokens` 在 Opus 4.6 和 Sonnet 4.6 上均已弃用。
> **旧型号：** 将 `"type": "enabled"` 与 `"budget_tokens": N` 结合使用（必须 < `max_tokens`，最小 1024）。```bash
# Opus 4.6: adaptive thinking (recommended)
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "thinking": {
      "type": "adaptive"
    },
    "output_config": {
      "effort": "high"
    },
    "messages": [{"role": "user", "content": "Solve this step by step..."}]
  }'
```
---

## 必需的标头

|标题|价值|描述 |
| ------------------- | ------------------ | -------------------------- |
| `Content-Type` | `application/json` |必填|
| `x-api-key` |您的 API 密钥 |认证|
| `anthropic-version` | `2023-06-01` | API版本 |
| `anthropic-beta` | Beta 功能 ID |测试版功能所需 |
