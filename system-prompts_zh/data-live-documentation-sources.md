<!--
name: '数据：实时文档来源'
description: WebFetch URL，用于从官方来源获取当前的 Claude API 和 Agent SDK 文档
ccVersion: 2.1.63
-->
# 实时文档源

此文件包含用于从 platform.claude.com 和 Agent SDK 存储库获取当前信息的 WebFetch URL。当用户需要自上次更新缓存内容以来可能已更改的最新数据时，请使用这些数据。

## 何时使用 WebFetch

- 用户明确要求“最新”或“当前”信息
- 缓存数据似乎不正确
- 用户询问缓存内容中未涵盖的功能
- 用户需要特定的API详细信息或示例

## Claude API 文档 URL

### 型号和定价

|主题 |网址 |提取提示|
| ---------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
|型号概览 | `https://platform.claude.com/docs/en/about-claude/models/overview.md` | “提取所有 Claude 模型的当前模型 ID、上下文窗口和定价”|
|定价| `https://platform.claude.com/docs/en/pricing.md` | “提取当前每百万代币的输入和输出定价” |

### 核心特性

|主题 |网址 |提取提示|
| ----------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
|延伸思考| `https://platform.claude.com/docs/en/build-with-claude/extended-thinking.md` | “提取扩展思维参数、budget_tokens 要求和使用示例” |
|适应性思维| `https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking.md` | “提取适应性思维设置、努力水平和 {{OPUS_NAME}} 使用示例” |
|努力参数| `https://platform.claude.com/docs/en/build-with-claude/effort.md` | “提取努力水平、成本质量权衡以及与思维的互动” |
|工具使用| `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview.md` | “提取工具定义架构、tool_choice 选项和处理工具结果”|
|流媒体| `https://platform.claude.com/docs/en/build-with-claude/streaming.md` | “提取流事件类型、SDK 示例和最佳实践”|
|提示缓存 | `https://platform.claude.com/docs/en/build-with-claude/prompt-caching.md` | “提取cache_control 的用法、定价优势和实施示例”|

### 媒体和文件

|主题 |网址 |提取提示|
| ----------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
|愿景 | `https://platform.claude.com/docs/en/build-with-claude/vision.md` | “提取支持的图像格式、大小限制和代码示例”|
| PDF 支持 | `https://platform.claude.com/docs/en/build-with-claude/pdf-support.md` | “提取 PDF 处理功能、限制和示例”|

### API操作|主题 |网址 |提取提示|
| ---------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
|批处理| `https://platform.claude.com/docs/en/build-with-claude/batch-processing.md` | “提取批量 API 端点、请求格式和轮询结果” |
|文件 API | `https://platform.claude.com/docs/en/build-with-claude/files.md` | “提取文件上传、下载和消息中的引用，包括支持的类型和测试标头” |
|令牌计数 | `https://platform.claude.com/docs/en/build-with-claude/token-counting.md` | “提取令牌计数 API 用法和示例”|
|速率限制 | `https://platform.claude.com/docs/en/api/rate-limits.md` | “按级别和型号提取当前速率限制”|
|错误 | `https://platform.claude.com/docs/en/api/errors.md` | “提取 HTTP 错误代码、含义和重试指南”|

### 工具

|主题 |网址 |提取提示|
| -------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
|代码执行 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool.md` | “提取代码执行工具设置、文件上传、容器重用和响应处理”|
|电脑使用 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use.md` | “提取计算机使用工具设置、功能和实施示例”|

### 高级功能

|主题 |网址 |提取提示|
| ------------------ | ------------------------------------------------------------------------------------------ | --------------------------------------------------- |
|结构化输出 | `https://platform.claude.com/docs/en/build-with-claude/structured-outputs.md` | “提取output_config.format 的用法和架构实施”|
|压实| `https://platform.claude.com/docs/en/build-with-claude/compaction.md` | “提取压缩设置、触发配置和压缩流”|
|引文| `https://platform.claude.com/docs/en/build-with-claude/citations.md` | 《摘录引文格式及实现》|
|上下文窗口 | `https://platform.claude.com/docs/en/build-with-claude/context-windows.md` | “提取上下文窗口大小和令牌管理” |

---

## Claude API SDK 存储库| SDK |网址 |描述 |
| ---------- | -------------------------------------------------------------------- | ------------------------------------------ |
|蟒蛇 | `https://github.com/anthropics/anthropic-sdk-python` | `anthropic` pip包源码 |
|打字稿 | `https://github.com/anthropics/anthropic-sdk-typescript` | `@anthropic-ai/sdk` npm 源 |
|爪哇 | `https://github.com/anthropics/anthropic-sdk-java` | `anthropic-java` Maven 源码 |
|去 | `https://github.com/anthropics/anthropic-sdk-go` | Go 模块源码 |
|红宝石 | `https://github.com/anthropics/anthropic-sdk-ruby` | `anthropic` 宝石来源 |
| C# | `https://github.com/anthropics/anthropic-sdk-csharp` | NuGet 包源码 |
| PHP | `https://github.com/anthropics/anthropic-sdk-php` | Composer包源码|

---

## Agent SDK 文档 URL

### 核心文档

|主题 |网址 |提取提示|
| -------------------- | ----------------------------------------------------------- | --------------------------------------------------------------------------- |
|代理SDK概述| `https://platform.claude.com/docs/en/agent-sdk.md` | “提取 Agent SDK 概述、主要功能和用例”|
|代理 SDK Python | `https://github.com/anthropics/claude-agent-sdk-python` | “提取Python SDK安装、导入和基本用法”|
|代理 SDK TypeScript | `https://github.com/anthropics/claude-agent-sdk-typescript` | “Extract TypeScript SDK 安装、导入和基本用法”|

### SDK 参考（GitHub 自述文件）

|主题 |网址 |提取提示|
| -------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| Python SDK | `https://raw.githubusercontent.com/anthropics/claude-agent-sdk-python/main/README.md` | “提取 Python SDK API 参考、类和方法”|
| TypeScript SDK | `https://raw.githubusercontent.com/anthropics/claude-agent-sdk-typescript/main/README.md` | “提取 TypeScript SDK API 参考、类型和函数” |

### npm/PyPI 包

|套餐 |网址 |描述 |
| ----------------------------------- | -------------------------------------------------------------------------- | ---------------------------------- |
|克劳德代理 SDK (Python) | `https://pypi.org/project/claude-agent-sdk/` | PyPI 上的 Python 包 |
| @anthropic-ai/claude-agent-sdk (TS) | `https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk` | npm 上的 TypeScript 包 |

### GitHub 存储库

|资源 |网址 |描述 |
| -------------- | ----------------------------------------------------------- | ----------------------------------- |
| Python SDK | `https://github.com/anthropics/claude-agent-sdk-python` | Python包源码 |
| TypeScript SDK | `https://github.com/anthropics/claude-agent-sdk-typescript` | TypeScript/Node.js 包源码 |
| MCP 服务器 | `https://github.com/modelcontextprotocol` |官方 MCP 服务器实现 |

---

## 后备策略

如果 WebFetch 失败（网络问题、URL 更改）：1. 使用特定于语言的文件中的缓存内容（注意缓存日期）
2. 告知用户数据可能已过时
3. 建议他们直接检查 platform.claude.com 或 GitHub 存储库
