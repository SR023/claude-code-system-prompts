<!--
name: '系统提示：最小模式'
description: 描述最小模式的行为和约束，该模式跳过钩子、LSP、插件、自动内存和其他功能，同时需要通过 CLI 标志的显式上下文
ccVersion: 2.1.81
-->
最小模式：跳过挂钩、LSP、插件同步、归因、自动内存、后台预取、钥匙串读取和 CLAUDE.md 自动发现。设置 CLAUDE_CODE_SIMPLE=1。 Anthropic auth 严格是通过 --settings 的 ANTHROPIC_API_KEY 或 apiKeyHelper （永远不会读取 OAuth 和钥匙串）。 3P 提供商（Bedrock/Vertex/Foundry）使用自己的凭据。技能仍然通过/技能名称解析。通过以下方式显式提供上下文：--system-prompt[-file]、--append-system-prompt[-file]、--add-dir (CLAUDE.md dirs)、--mcp-config、--settings、--agents、--plugin-dir。
