<!--
name: '数据：Claude API 参考 — Python'
description: Python SDK参考包括安装、客户端初始化、基本请求、思考和多轮对话
ccVersion: 2.1.83
-->
# Claude API — Python

## 安装```bash
pip install anthropic
```
## 客户端初始化```python
import anthropic

# Default (uses ANTHROPIC_API_KEY env var)
client = anthropic.Anthropic()

# Explicit API key
client = anthropic.Anthropic(api_key="your-api-key")

# Async client
async_client = anthropic.AsyncAnthropic()
```
---

## 基本消息请求```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    messages=[
        {"role": "user", "content": "What is the capital of France?"}
    ]
)
# response.content is a list of content block objects (TextBlock, ThinkingBlock,
# ToolUseBlock, ...). Check .type before accessing .text.
for block in response.content:
    if block.type == "text":
        print(block.text)
```
---

## 系统提示```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    system="You are a helpful coding assistant. Always provide examples in Python.",
    messages=[{"role": "user", "content": "How do I read a JSON file?"}]
)
```
---

## 愿景（图像）

### Base64```python
import base64

with open("image.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": image_data
                }
            },
            {"type": "text", "text": "What's in this image?"}
        ]
    }]
)
```
### 网址```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "url",
                    "url": "https://example.com/image.png"
                }
            },
            {"type": "text", "text": "Describe this image"}
        ]
    }]
)
```
---

## 提示缓存

缓存大型上下文以降低成本（节省高达 90%）。 **缓存是前缀匹配** - 前缀中任何位置的任何字节更改都会使其后面的所有内容无效。有关放置模式、架构指南（冻结系统提示、确定性工具顺序、在何处放置易失性内容）以及静默无效器审核清单，请阅读 `shared/prompt-caching.md`。

### 自动缓存（推荐）

使用顶级 `cache_control` 自动缓存请求中的最后一个可缓存块 - 无需注释各个内容块：```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    cache_control={"type": "ephemeral"},  # auto-caches the last cacheable block
    system="You are an expert on this large document...",
    messages=[{"role": "user", "content": "Summarize the key points"}]
)
```
### 手动缓存控制

为了进行细粒度控制，请将 `cache_control` 添加到特定内容块：```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    system=[{
        "type": "text",
        "text": "You are an expert on this large document...",
        "cache_control": {"type": "ephemeral"}  # default TTL is 5 minutes
    }],
    messages=[{"role": "user", "content": "Summarize the key points"}]
)

# With explicit TTL (time-to-live)
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    system=[{
        "type": "text",
        "text": "You are an expert on this large document...",
        "cache_control": {"type": "ephemeral", "ttl": "1h"}  # 1 hour TTL
    }],
    messages=[{"role": "user", "content": "Summarize the key points"}]
)
```
### 验证缓存命中```python
print(response.usage.cache_creation_input_tokens)  # tokens written to cache (~1.25x cost)
print(response.usage.cache_read_input_tokens)      # tokens served from cache (~0.1x cost)
print(response.usage.input_tokens)                 # uncached tokens (full cost)
```
如果 `cache_read_input_tokens` 在重复的相同前缀请求中为零，则说明静默无效器正在工作 - `datetime.now()` 或系统提示中的 UUID、未排序的 `json.dumps()` 或不同的工具集。有关完整审核表，请参阅 `shared/prompt-caching.md`。

---

## 延伸思考

> **Opus 4.6 和 Sonnet 4.6：** 使用适应性思维。 `budget_tokens` 在 Opus 4.6 和 Sonnet 4.6 上均已弃用。
> **旧型号：** 使用 `thinking: {type: "enabled", budget_tokens: N}`（必须 < `max_tokens`，最小 1024）。```python
# Opus 4.6: adaptive thinking (recommended)
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # low | medium | high | max
    messages=[{"role": "user", "content": "Solve this step by step..."}]
)

# Access thinking and response
for block in response.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking}")
    elif block.type == "text":
        print(f"Response: {block.text}")
```
---

## 错误处理```python
import anthropic

try:
    response = client.messages.create(...)
except anthropic.BadRequestError as e:
    print(f"Bad request: {e.message}")
except anthropic.AuthenticationError:
    print("Invalid API key")
except anthropic.PermissionDeniedError:
    print("API key lacks required permissions")
except anthropic.NotFoundError:
    print("Invalid model or endpoint")
except anthropic.RateLimitError as e:
    retry_after = int(e.response.headers.get("retry-after", "60"))
    print(f"Rate limited. Retry after {retry_after}s.")
except anthropic.APIStatusError as e:
    if e.status_code >= 500:
        print(f"Server error ({e.status_code}). Retry later.")
    else:
        print(f"API error: {e.message}")
except anthropic.APIConnectionError:
    print("Network error. Check internet connection.")
```
---

## 多轮对话

API 是无状态的——每次发送完整的对话历史记录。```python
class ConversationManager:
    """Manage multi-turn conversations with the Claude API."""

    def __init__(self, client: anthropic.Anthropic, model: str, system: str = None):
        self.client = client
        self.model = model
        self.system = system
        self.messages = []

    def send(self, user_message: str, **kwargs) -> str:
        """Send a message and get a response."""
        self.messages.append({"role": "user", "content": user_message})

        response = self.client.messages.create(
            model=self.model,
            max_tokens=kwargs.get("max_tokens", 16000),
            system=self.system,
            messages=self.messages,
            **kwargs
        )

        assistant_message = next(
            (b.text for b in response.content if b.type == "text"), ""
        )
        self.messages.append({"role": "assistant", "content": assistant_message})

        return assistant_message

# Usage
conversation = ConversationManager(
    client=anthropic.Anthropic(),
    model="{{OPUS_ID}}",
    system="You are a helpful assistant."
)

response1 = conversation.send("My name is Alice.")
response2 = conversation.send("What's my name?")  # Claude remembers "Alice"
```
**规则：**

- 消息必须在 `user` 和 `assistant` 之间交替
- 第一条消息必须是 `user`

---

### 压缩（长对话）

> **Beta、Opus 4.6 和 Sonnet 4.6。** 当对话接近 200K 上下文窗口时，压缩会自动总结服务器端较早的上下文。 API返回`compaction`块；您必须在后续请求中将其传回 - 附加 `response.content`，而不仅仅是文本。```python
import anthropic

client = anthropic.Anthropic()
messages = []

def chat(user_message: str) -> str:
    messages.append({"role": "user", "content": user_message})

    response = client.beta.messages.create(
        betas=["compact-2026-01-12"],
        model="{{OPUS_ID}}",
        max_tokens=16000,
        messages=messages,
        context_management={
            "edits": [{"type": "compact_20260112"}]
        }
    )

    # Append full content — compaction blocks must be preserved
    messages.append({"role": "assistant", "content": response.content})

    return next(block.text for block in response.content if block.type == "text")

# Compaction triggers automatically when context grows large
print(chat("Help me build a Python web scraper"))
print(chat("Add support for JavaScript-rendered pages"))
print(chat("Now add rate limiting and error handling"))
```
---

## 停止原因

响应中的 `stop_reason` 字段指示模型停止生成的原因：

|价值|意义|
|--------|---------|
| `end_turn` |Claude很自然地完成了回应|
| `max_tokens` |达到 `max_tokens` 限制 — 增加限制或使用流媒体 |
| `stop_sequence` |按下自定义停止顺序 |
| `tool_use` |Claude想要调用一个工具——执行它并继续 |
| `pause_turn` |模型已暂停并可以恢复（代理流）|
| `refusal` |Claude出于安全原因拒绝 - 输出可能与您的模式不匹配 |

---

## 成本优化策略

### 1. 对重复上下文使用提示缓存```python
# Automatic caching (simplest — caches the last cacheable block)
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    cache_control={"type": "ephemeral"},
    system=large_document_text,  # e.g., 50KB of context
    messages=[{"role": "user", "content": "Summarize the key points"}]
)

# First request: full cost
# Subsequent requests: ~90% cheaper for cached portion
```
### 2. 选择正确的型号```python
# Default to Opus for most tasks
response = client.messages.create(
    model="{{OPUS_ID}}",  # $5.00/$25.00 per 1M tokens
    max_tokens=16000,
    messages=[{"role": "user", "content": "Explain quantum computing"}]
)

# Use Sonnet for high-volume production workloads
standard_response = client.messages.create(
    model="{{SONNET_ID}}",  # $3.00/$15.00 per 1M tokens
    max_tokens=16000,
    messages=[{"role": "user", "content": "Summarize this document"}]
)

# Use Haiku only for simple, speed-critical tasks
simple_response = client.messages.create(
    model="{{HAIKU_ID}}",  # $1.00/$5.00 per 1M tokens
    max_tokens=256,
    messages=[{"role": "user", "content": "Classify this as positive or negative"}]
)
```
### 3. 在请求之前使用令牌计数```python
count_response = client.messages.count_tokens(
    model="{{OPUS_ID}}",
    messages=messages,
    system=system
)

estimated_input_cost = count_response.input_tokens * 0.000005  # $5/1M tokens
print(f"Estimated input cost: ${estimated_input_cost:.4f}")
```
---

## 使用指数退避重试

> **注意：** Anthropic SDK 使用指数退避自动重试速率限制 (429) 和服务器错误 (5xx)。您可以使用 `max_retries` 进行配置（默认值：2）。仅当您需要的行为超出 SDK 提供的范围时，才实施自定义重试逻辑。```python
import time
import random
import anthropic

def call_with_retry(
    client: anthropic.Anthropic,
    max_retries: int = 5,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    **kwargs
):
    """Call the API with exponential backoff retry."""
    last_exception = None

    for attempt in range(max_retries):
        try:
            return client.messages.create(**kwargs)
        except anthropic.RateLimitError as e:
            last_exception = e
        except anthropic.APIStatusError as e:
            if e.status_code >= 500:
                last_exception = e
            else:
                raise  # Client errors (4xx except 429) should not be retried

        delay = min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
        print(f"Retry {attempt + 1}/{max_retries} after {delay:.1f}s")
        time.sleep(delay)

    raise last_exception
```
