<!--
name: '数据：HTTP 错误代码参考'
description: Claude API返回的HTTP错误码常见原因及处理策略参考
ccVersion: 2.1.73
-->
# HTTP 错误代码参考

此文件记录了 Claude API 返回的 HTTP 错误代码、常见原因以及处理方法。有关特定于语言的错误处理示例，请参阅 `python/` 或 `typescript/` 文件夹。

## 错误代码摘要

|代码|错误类型 |可重试 |共同原因|
| ---- | ----------------------- | --------- | ------------------------------------------------ |
| 400 | `invalid_request_error` |没有 |请求格式或参数无效 |
| 401 | 401 `authentication_error` |没有 | API 密钥无效或丢失 |
| 403 | 403 `permission_error` |没有 | API 密钥缺少权限 |
| 404 | 404 `not_found_error` |没有 |端点或模型 ID 无效 |
| 413 | 413 `request_too_large` |没有 |请求超出大小限制 |
| 429 | 429 `rate_limit_error` |是的 |请求过多 |
| 500 | 500 `api_error` |是的 |人为服务问题 |
| 529 | 529 `overloaded_error` |是的 | API 暂时超载 |

## 详细错误信息

### 400 错误请求

**原因：**

- 请求正文中的 JSON 格式错误
- 缺少必需的参数（`model`、`max_tokens`、`messages`）
- 无效的参数类型（例如，需要整数的字符串）
- 空消息数组
- 消息不交替用户/助理

**错误示例：**```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate between \"user\" and \"assistant\""
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```
**修复：** 发送前验证请求结构。检查：

- `model` 是有效的型号 ID
- `max_tokens` 是正整数
- `messages` 数组非空且正确交替

---

### 401 未经授权

**原因：**

- 缺少 `x-api-key` 标头或 `Authorization` 标头
- API 密钥格式无效
- 撤销或删除 API 密钥

**修复：** 确保 `ANTHROPIC_API_KEY` 环境变量设置正确。

---

### 403 禁止

**原因：**

- API 密钥无权访问所请求的模型
- 组织级别的限制
- 尝试在没有测试版访问权限的情况下访问测试版功能

**修复：** 在控制台中检查您的 API 密钥权限。您可能需要不同的 API 密钥或请求访问特定功能。

---

### 404 未找到

**原因：**

- 型号 ID 中的拼写错误（例如，`claude-sonnet-4.6` 而不是 `claude-sonnet-4-6`）
- 使用已弃用的模型 ID
- API 端点无效

**修复：** 使用模型文档中的准确模型 ID。您可以使用别名（例如，`{{OPUS_ID}}`）。

---

### 413 请求太大

**原因：**

- 请求正文超出最大大小
- 输入中的标记过多
- 图像数据太大

**修复：** 减少输入大小 - 截断对话历史记录、压缩/调整图像大小或将大文档拆分为块。

---

### 400 验证错误

一些 400 错误与参数验证特别相关：

- `max_tokens` 超出型号限制
- `temperature` 值无效（必须为 0.0-1.0）
- 扩展思维中的`budget_tokens` >= `max_tokens`
- 无效的工具定义架构

**扩展思维的常见错误：**```
# Wrong: budget_tokens must be < max_tokens
thinking: budget_tokens=10000, max_tokens=1000  → Error!

# Correct
thinking: budget_tokens=10000, max_tokens=16000
```
---

### 429 费率有限

**原因：**

- 超出每分钟请求数 (RPM)
- 超出每分钟令牌数 (TPM)
- 每日超出代币数量 (TPD)

**要检查的标题：**

- `retry-after`：重试前等待的秒数
- `x-ratelimit-limit-*`：你的极限
- `x-ratelimit-remaining-*`：剩余配额

**修复：** Anthropic SDK 使用指数退避自动重试 429 和 5xx 错误（默认值：`max_retries=2`）。有关自定义重试行为，请参阅特定于语言的错误处理示例。

---

### 500 内部服务器错误

**原因：**

- 临时人类服务问题
- API 处理中的错误

**修复：** 使用指数退避重试。如果持续存在，请检查 [status.anthropic.com](https://status.anthropic.com)。

---

### 529 超载

**原因：**

- API需求高
- 服务能力达到

**修复：** 使用指数退避重试。考虑使用不同的模型（俳句通常负载较少），随着时间的推移分散请求，或实现请求排队。

---

## 常见错误和修复

|错误|错误 |修复|
| ------------------------------------------- | ---------------- | ------------------------------------------------------- |
| `budget_tokens` >= `max_tokens` | 400 |确保 `budget_tokens` < `max_tokens` |
|型号 ID 中的拼写错误 | 404 | 404使用有效的型号 ID，如 `{{OPUS_ID}}` |
|第一条消息是 `assistant` | 400 |第一条消息必须是 `user` |
|连续的相同角色消息| 400 |备用 `user` 和 `assistant` |
|代码中的 API 密钥 | 401（泄露密钥）|使用环境变量|
|自定义重试需求| 429/5xx | 429/5xx SDK自动重试；使用 `max_retries` 进行定制 |

## SDK 中的类型异常

**始终使用 SDK 的类型化异常类**，而不是使用字符串匹配检查错误消息。每个 HTTP 错误代码都映射到一个特定的异常类：

| HTTP 代码 | TypeScript 类 | Python 类 |
| --------- | --------------------------------- | --------------------------------- |
| 400 | `Anthropic.BadRequestError` | `anthropic.BadRequestError` |
| 401 | 401 `Anthropic.AuthenticationError` | `anthropic.AuthenticationError` |
| 403 | 403 `Anthropic.PermissionDeniedError` | `anthropic.PermissionDeniedError` |
| 404 | 404 `Anthropic.NotFoundError` | `anthropic.NotFoundError` |
| 429 | 429 `Anthropic.RateLimitError` | `anthropic.RateLimitError` |
| 500+ | `Anthropic.InternalServerError` | `anthropic.InternalServerError` |
|任何 | `Anthropic.APIError` | `anthropic.APIError` |```typescript
// ✅ Correct: use typed exceptions
try {
  const response = await client.messages.create({...});
} catch (error) {
  if (error instanceof Anthropic.RateLimitError) {
    // Handle rate limiting
  } else if (error instanceof Anthropic.APIError) {
    console.error(`API error ${error.status}:`, error.message);
  }
}

// ❌ Wrong: don't check error messages with string matching
try {
  const response = await client.messages.create({...});
} catch (error) {
  const msg = error instanceof Error ? error.message : String(error);
  if (msg.includes("429") || msg.includes("rate_limit")) { ... }
}
```
所有异常类都扩展 `Anthropic.APIError`，它具有 `status` 属性。使用 `instanceof` 从最具体到最不具体进行检查（例如，在 `APIError` 之前检查 `RateLimitError`）。
