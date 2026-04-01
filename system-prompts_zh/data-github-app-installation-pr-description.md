<!--
name: '资料：GitHub App安装PR说明'
description: 安装 Claude Code GitHub 应用程序集成时 PR 描述的模板
ccVersion: 2.0.14
-->
## \uD83E\uDD16 安装 Claude Code GitHub 应用程序

此 PR 添加了一个 GitHub Actions 工作流程，使 Claude Code 能够集成到我们的存储库中。

### 什么是Claude码？

[Claude Code](https://claude.com/claude-code) 是一种人工智能编码代理，可以帮助：
- 错误修复和改进  
- 文档更新
- 实施新功能
- 代码审查和建议
- 编写测试
- 还有更多！

### 它是如何工作的

合并此 PR 后，我们将能够通过在拉取请求或问题评论中提及 @claude 来与 Claude 进行交互。
一旦触发工作流程，Claude 将分析评论和周围上下文，并在 GitHub 操作中执行请求。

### 重要提示

- **此工作流程只有在合并此 PR 后才会生效**
- **@claude 提及在合并完成后才起作用**
- 每当 PR 或问题评论中提到 Claude 时，工作流程就会自动运行
- Claude 可以访问整个 PR 或问题上下文，包括文件、差异和以前的评论

### 安全

- 我们的 Anthropic API 密钥作为 GitHub Actions 秘密安全存储
- 只有对存储库具有写入权限的用户才能触发工作流程
- 所有 Claude 运行都存储在 GitHub Actions 运行历史记录中
- Claude 的默认工具仅限于读取/写入文件以及通过创建注释、分支和提交与我们的存储库交互。
- 我们可以通过将更多允许的工具添加到工作流程文件中来添加它们，例如：```
allowed_tools: Bash(npm install),Bash(npm run build),Bash(npm run lint),Bash(npm run test)
```
[Claude 代码操作存储库](https://github.com/anthropics/claude-code-action) 中有更多信息。

合并此 PR 后，让我们尝试在任何 PR 的评论中提及 @claude 以开始！
