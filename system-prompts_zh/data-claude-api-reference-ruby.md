<!--
name: '数据：Claude API 参考 — Ruby'
description: Ruby SDK 参考，包括安装、客户端初始化、基本请求、流式传输和 Beta 工具运行程序
ccVersion: 2.1.83
-->
# 克劳德 API — Ruby

> **注意：** Ruby SDK 支持 Claude API。工具运行程序可通过 `client.beta.messages.tool_runner()` 获得测试版。 Agent SDK 尚不可用于 Ruby。

＃＃ 安装```bash
gem install anthropic
```
## 客户端初始化```ruby
require "anthropic"

# Default (uses ANTHROPIC_API_KEY env var)
client = Anthropic::Client.new

# Explicit API key
client = Anthropic::Client.new(api_key: "your-api-key")
```
---

## 基本消息请求```ruby
message = client.messages.create(
  model: :"{{OPUS_ID}}",
  max_tokens: 16000,
  messages: [
    { role: "user", content: "What is the capital of France?" }
  ]
)
# content is an array of polymorphic block objects (TextBlock, ThinkingBlock,
# ToolUseBlock, ...). .type is a Symbol — compare with :text, not "text".
# .text raises NoMethodError on non-TextBlock entries.
message.content.each do |block|
  puts block.text if block.type == :text
end
```
---

## 流媒体```ruby
stream = client.messages.stream(
  model: :"{{OPUS_ID}}",
  max_tokens: 64000,
  messages: [{ role: "user", content: "Write a haiku" }]
)

stream.text.each { |text| print(text) }
```
---

## 工具使用

Ruby SDK 支持通过原始 JSON 模式定义使用工具，并提供用于自动工具执行的测试版工具运行程序。

### 工具运行程序（测试版）```ruby
class GetWeatherInput < Anthropic::BaseModel
  required :location, String, doc: "City and state, e.g. San Francisco, CA"
end

class GetWeather < Anthropic::BaseTool
  doc "Get the current weather for a location"

  input_schema GetWeatherInput

  def call(input)
    "The weather in #{input.location} is sunny and 72°F."
  end
end

client.beta.messages.tool_runner(
  model: :"{{OPUS_ID}}",
  max_tokens: 16000,
  tools: [GetWeather.new],
  messages: [{ role: "user", content: "What's the weather in San Francisco?" }]
).each_message do |message|
  puts message.content
end
```
### 手动循环

有关工具定义格式和代理循环模式，请参阅[共享工具使用概念](../shared/tool-use-concepts.md)。

---

## 提示缓存

`system_:`（尾随下划线 - 避免遮蔽 `Kernel#system`）采用文本块数组；在最后一个块上设置 `cache_control`。普通哈希通过 `OrHash` 类型别名工作。有关放置模式和静默无效器审核清单，请参阅 `shared/prompt-caching.md`。```ruby
message = client.messages.create(
  model: :"{{OPUS_ID}}",
  max_tokens: 16000,
  system_: [
    { type: "text", text: long_system_prompt, cache_control: { type: "ephemeral" } }
  ],
  messages: [{ role: "user", content: "Summarize the key points" }]
)
```
对于 1 小时 TTL：`cache_control: { type: "ephemeral", ttl: "1h" }`。 `messages.create` 上还有一个顶级 `cache_control:`，它自动放置在最后一个可缓存块上。

通过 `message.usage.cache_creation_input_tokens` / `message.usage.cache_read_input_tokens` 验证命中。
