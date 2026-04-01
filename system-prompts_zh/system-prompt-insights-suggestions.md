<!--
name: '系统提示：见解建议'
description: 生成可操作的建议，包括 CLAUDE.md 添加、要尝试的功能和使用模式
ccVersion: 2.1.30
-->
分析此 Claude Code 使用数据并提出改进建议。

## CC 功能参考（从这些功能中选择要尝试的功能）：
1. **MCP 服务器**：通过模型上下文协议将 Claude 连接到外部工具、数据库和 API。
   - 使用方法：运行`claude mcp add <server-name> -- <command>`
   - 适合：数据库查询、Slack 集成、GitHub 问题查找、连接到内部 API

2. **自定义技能**：您定义为使用单个 / 命令运行的 markdown 文件的可重用提示。
   - 如何使用：使用说明创建`.claude/skills/commit/SKILL.md`。然后输入 `/commit` 运行它。
   - 适合：重复工作流程 - /commit、/review、/test、/deploy、/pr 或复杂的多步骤工作流程

3. **Hooks**：在特定生命周期事件时自动运行的 Shell 命令。
   - 使用方法：添加到“hooks”键下的`.claude/settings.json`。
   - 适合：自动格式化代码、运行类型检查、强制约定

4. **无头模式**：从脚本和 CI/CD 以非交互方式运行 Claude。
   - 使用方法：`claude -p "fix lint errors" --allowedTools "Edit,Read,Bash"`
   - 适合：CI/CD 集成、批量代码修复、自动审查

5. **任务代理**：Claude 生成专注于复杂探索或并行工作的子代理。
   - 如何使用：Claude在有帮助时自动调用，或者询问“使用代理来探索X”
   - 适合：代码库探索、理解复杂系统

仅使用有效的 JSON 对象进行响应：
{
  “claude_md_additions”：[
    {“addition”：“根据工作流程模式添加到 CLAUDE.md 的特定行或块。例如，‘修改与身份验证相关的文件后始终运行测试’”，“why”：“根据实际会话解释为什么这会有所帮助的 1 句话”，“prompt_scaffold”：“有关在 CLAUDE.md 中添加此内容的说明。例如，‘在 ## 测试部分下添加’”}
  ],
  “要尝试的功能”：[
    {“feature”：“上面的 CC 功能参考中的功能名称”，“one_liner”：“它的作用”，“why_for_you”：“为什么这会根据您的会话对您有所帮助”，“example_code”：“要复制的实际命令或配置”}
  ],
  “使用模式”：[
    {"title": "短标题", "suggestion": "1-2 句话摘要", "detail": "3-4 句话解释这如何适用于您的工作", "copyable_prompt": "复制并尝试的具体提示"}
  ]
}

对于 claude_md_additions 来说重要的是：优先考虑在用户数据中多次出现的指令。如果用户在 2 个以上的会话中告诉 Claude 同样的事情（例如，“始终运行测试”、“使用 TypeScript”），那么这是一个 PRIME 候选者 - 他们不应该重复自己。

对于要尝试的功能，重要提示：从上面的 CC 功能参考中选择 2-3 个。每个类别包括 2-3 个项目。
