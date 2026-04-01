<!--
name: '资料：克劳德模型目录'
description: 当前和旧版 Claude 模型的目录，包含准确的模型 ID、别名、上下文窗口和定价
ccVersion: 2.1.79
-->
# 克劳德模型目录

**仅使用此文件中列出的准确模型 ID。**切勿猜测或构造模型 ID — 不正确的 ID 将导致 API 错误。尽可能使用别名。有关最新信息，请 WebFetch `shared/live-sources.md` 中的模型概述 URL，或直接查询模型 API（请参阅下面的编程模型发现）。

## 程序化模型发现

对于**实时**功能数据 - 上下文窗口、最大输出标记、功能支持（思维、愿景、努力、结构化输出等） - 查询模型 API，而不是依赖下面的缓存表。当用户询问“X 的上下文窗口是什么”、“模型 X 支持愿景/思维/努力吗”、“哪些模型支持功能 Y”或者想要在运行时按功能选择模型时，请使用此选项。```python
m = client.models.retrieve("claude-opus-4-6")
m.id                 # "claude-opus-4-6"
m.display_name       # "Claude Opus 4.6"
m.max_input_tokens   # context window (int)
m.max_tokens         # max output tokens (int)

# capabilities is an untyped nested dict — bracket access, check ["supported"] at the leaf
caps = m.capabilities
caps["image_input"]["supported"]                       # vision
caps["thinking"]["types"]["adaptive"]["supported"]     # adaptive thinking
caps["effort"]["max"]["supported"]                     # effort: max (also low/medium/high)
caps["structured_outputs"]["supported"]
caps["context_management"]["compact_20260112"]["supported"]

# filter across all models — iterate the page object directly (auto-paginates); do NOT use .data
[m for m in client.models.list()
 if m.capabilities["thinking"]["types"]["adaptive"]["supported"]
 and m.max_input_tokens >= 200_000]
```
顶级字段（`id`、`display_name`、`max_input_tokens`、`max_tokens`）是类型属性。 `capabilities` 是一个字典 - 使用括号访问，而不是属性访问。 API 返回每个叶子上带有 `supported: true/false` 的每个模型的完整功能树，因此支架链在没有 `.get()` 防护装置的情况下是安全的。 TypeScript SDK：相同的方法名称，也在迭代时自动分页。

### 原始 HTTP```bash
curl https://api.anthropic.com/v1/models/claude-opus-4-6 \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01"
```

```json
{
  "id": "claude-opus-4-6",
  "display_name": "Claude Opus 4.6",
  "max_input_tokens": 1000000,
  "max_tokens": 128000,
  "capabilities": {
    "image_input": {"supported": true},
    "structured_outputs": {"supported": true},
    "thinking": {"supported": true, "types": {"enabled": {"supported": true}, "adaptive": {"supported": true}}},
    "effort": {"supported": true, "low": {"supported": true}, …, "max": {"supported": true}},
    …
  }
}
```
## 当前型号（推荐）

|友好名称|别名（使用这个）|完整身份证件 |背景 |最大输出|状态 |
|--------------------|---------------------|----------------------------------------|----------------|------------------------|--------|
|克劳德作品 4.6 | `claude-opus-4-6` | — | 200K（1M 测试版）| 128K |活跃|
|克劳德十四行诗 4.6 | `claude-sonnet-4-6` | - | 200K（1M 测试版）| 64K |活跃|
|克劳德俳句 4.5 | `claude-haiku-4-5` | `claude-haiku-4-5-20251001` | 20万| 64K |活跃|

### 型号说明

- **Claude Opus 4.6** — 我们用于构建代理和编码的最智能模型。支持自适应思维（推荐），最大 128K 输出令牌（需要流式传输才能实现大输出）。 1M 上下文窗口可通过 `context-1m-2025-08-07` 标头在测试版中使用。
- **Claude Sonnet 4.6** — 我们速度与智慧的最佳结合。支持适应性思维（推荐）。 1M 上下文窗口可通过 `context-1m-2025-08-07` 标头在测试版中使用。最大 64K 输出令牌。
- **Claude Haiku 4.5** — 适用于简单任务的最快且最具成本效益的模型。

## 旧模型（仍然有效）

|友好名称|别名（使用这个）|完整身份证件 |状态 |
|--------------------|---------------------|----------------------------------------|--------|
|克劳德作品 4.5 | `claude-opus-4-5` | `claude-opus-4-5-20251101` |活跃|
|克劳德作品 4.1 | `claude-opus-4-1` | `claude-opus-4-1-20250805` |活跃|
|克劳德十四行诗 4.5 | `claude-sonnet-4-5` | `claude-sonnet-4-5-20250929` |活跃|
|克劳德十四行诗 4 | `claude-sonnet-4-0` | `claude-sonnet-4-20250514` |活跃|
|克劳德作品 4 | `claude-opus-4-0` | `claude-opus-4-20250514` |活跃|

## 已弃用的模型（即将退役）

|友好名称|别名（使用这个）|完整身份证件 |状态 |退休|
|--------------------|---------------------|----------------------------------------|------------------------|------------------------|
|克劳德俳句 3 | — | `claude-3-haiku-20240307` |已弃用 | 2026 年 4 月 19 日 |

## 退役模型（不再可用）

|友好名称|完整身份证件 |退休 |
|--------------------|--------------------------------------------|-------------|
|克劳德十四行诗 3.7 | `claude-3-7-sonnet-20250219` | 2026 年 2 月 19 日 |
|克劳德俳句 3.5 | `claude-3-5-haiku-20241022` | 2026 年 2 月 19 日 |
|克劳德作品 3 | `claude-3-opus-20240229` | 2026 年 1 月 5 日 |
|克劳德十四行诗 3.5 | `claude-3-5-sonnet-20241022` | 2025 年 10 月 28 日 |
|克劳德十四行诗 3.5 | `claude-3-5-sonnet-20240620` | 2025 年 10 月 28 日 |
|克劳德十四行诗 3 | `claude-3-sonnet-20240229` | 2025 年 7 月 21 日 |
|克劳德2.1 | `claude-2.1` | 2025 年 7 月 21 日 |
|克劳德2.0 | `claude-2.0` | 2025 年 7 月 21 日 |

## 解决用户请求

当用户按名称请求模型时，请使用此表查找正确的模型 ID：|用户说... |使用此型号 ID |
|--------------------------------------------------------|--------------------------------|
| “作品”、“最强大”| `claude-opus-4-6` |
| “作品 4.6”| `claude-opus-4-6` |
| “作品 4.5”| `claude-opus-4-5` |
| “作品 4.1”| `claude-opus-4-1` |
| “作品 4”、“作品 4.0”| `claude-opus-4-0` |
| “十四行诗”、“平衡”| `claude-sonnet-4-6` |
| “十四行诗 4.6”| `claude-sonnet-4-6` |
| “十四行诗 4.5”| `claude-sonnet-4-5` |
| “十四行诗 4”、“十四行诗 4.0” | `claude-sonnet-4-0` |
| “十四行诗 3.7” |已退休 — 建议 `claude-sonnet-4-5` |
| “十四行诗 3.5”|已退休 — 建议 `claude-sonnet-4-5` |
| “俳句”、“快”、“便宜”| `claude-haiku-4-5` |
| “俳句 4.5”| `claude-haiku-4-5` |
| “俳句 3.5”|已退休 — 建议 `claude-haiku-4-5` |
| 《俳句 3》|已弃用 — 建议 `claude-haiku-4-5` |
