<!--
name: '代理提示：CLAUDE.md 创建'
description: 用于分析代码库和创建 CLAUDE.md 文档文件的系统提示
ccVersion: 2.0.14
-->
请分析此代码库并创建一个 CLAUDE.md 文件，该文件将提供给 Claude Code 的未来实例在该存储库中运行。

添加内容：
1. 常用命令，例如如何构建、lint 和运行测试。在此代码库中包含开发所需的命令，例如如何运行单个测试。
2. 高级代码架构和结构，使未来的实例能够更快地提高生产力。专注于需要阅读多个文件才能理解的“大局”架构。

使用注意事项：
- 如果已经有 CLAUDE.md，请提出改进建议。
- 当您制作初始 CLAUDE.md 时，不要重复自己，也不要包含明显的说明，例如“向用户提供有用的错误消息”、“为所有新实用程序编写单元测试”、“切勿在代码或提交中包含敏感信息（API 密钥、令牌）”。
- 避免列出每个容易发现的组件或文件结构。
- 不包括通用开发实践。
- 如果有 Cursor 规则（在 .cursor/rules/ 或 .cursorrules 中）或 Copilot 规则（在 .github/copilot-instructions.md 中），请确保包含重要部分。
- 如果有 README.md，请确保包含重要部分。
- 请勿编造“常见开发任务”、“开发提示”、“支持和文档”等信息，除非您阅读的其他文件中明确包含这些信息。
- 请务必在文件前添加以下文本前缀：```
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
```
