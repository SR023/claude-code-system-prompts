
# Claude Code系统提示

[![Mentioned in Awesome Claude Code](ZXQMASK0016QXZ)](https://github.com/hesreallyhim/awesome-claude-code)

> [!重要]
> **新（2026 年 1 月 23 日）：我们已将所有 Claude Code 的约 40 个系统提醒添加到此列表中 - 请参阅 [System Reminders](#system-reminders)。**

该存储库包含截至 **[Claude Code v2.1.89](ZXQMASK0018QXZ)（2026 年 3 月 31 日）所有 Claude Code 的各种系统提示及其相关令牌计数的最新列表。** 它还包含自 v2.0.14 以来 138 个版本的系统提示的 [**CHANGELOG.md**](./CHANGELOG.md)。  来自 [<img src="ZXQMASK0019QXZ width="15"> **Piebald.**](ZXQMASK0020QXZ) 背后的团队

**此存储库会在每次 Claude 代码发布后几分钟内更新。  请参阅 [changelog](./CHANGELOG.md)，并在 X 上关注 [@PiebaldAI](ZXQMASK0021QXZ)，以获取每个版本中系统提示更改的摘要。**

> [!注意]
> ⭐ **给这个存储库加注星标**，以获得有关新 Claude 代码版本的通知。  对于每个新的 Claude Code 版本，我们都会在 GitHub 上创建一个版本，该版本将通知所有已为存储库加注星标的用户。

---

为什么会有多个“系统提示”？

**Claude Code 的系统提示符不仅仅是一个字符串。**

相反，有：
- 根据环境和各种配置有条件地添加大部分。
- `Write`、`Bash` 和 `TodoWrite` 等内置工具的描述，有些相当大。
- 针对探索和计划等内置代理的单独系统提示。
- 众多人工智能驱动的实用功能，例如对话压缩、`CLAUDE.md`生成、会话标题生成等，具有自己的系统提示。

结果是 110 多个字符串在一个非常大的压缩 JS 文件中不断变化和移动。

> [!提示]
> 想要在您自己的 Claude Code 安装中**修改系统提示符的特定部分**？  **使用[tweakcc](ZXQMASK0022QXZ)。**它—
> - 允许您将系统提示符的各个部分自定义为 markdown 文件，然后
> - 使用它们修补基于 npm 或本机（二进制）Claude Code 安装，并且
> - 当您和 Anthropic 对同一提示文件的修改存在冲突时，提供差异和冲突管理。

## 提取

该存储库包含使用脚本从最新 npm 版本的 Claude Code 中提取的系统提示。  由于它们是直接从 Claude Code 的编译源代码中提取的，因此保证它们正是 Claude Code 使用的内容。  如果您使用 [tweakcc](ZXQMASK0023QXZ) 自定义系统提示，它的工作方式类似 - 它会修补本地安装中与提取到此存储库中完全相同的字符串。

## 提示请注意，某些提示包含插值位，例如内​​置工具名称引用、可用子代理列表以及各种其他特定于上下文的变量，因此特定 Claude 代码会话中的实际计数会略有不同，但可能不会超过 ±20 个令牌。

### 代理提示

子代理和公用事业。

#### 子代理

- [Agent Prompt: Explore](./system-prompts_zh/agent-prompt-explore.md) (**494** tks) - Explore 子代理的系统提示。
- [Agent Prompt: Plan mode (enhanced)](./system-prompts_zh/agent-prompt-plan-mode-enhanced.md) (**636** tks) - 增强了计划子代理的提示。

#### 创作助理

- [Agent Prompt: Agent creation architect](./system-prompts_zh/agent-prompt-agent-creation-architect.md) (**1110** tks) - 创建具有详细规格的自定义 AI 代理的系统提示。
- [Agent Prompt: CLAUDE.md creation](./system-prompts_zh/agent-prompt-claudemd-creation.md) (**384** tks) - 用于分析代码库和创建 CLAUDE.md 文档文件的系统提示。
- [Agent Prompt: Status line setup](./system-prompts_zh/agent-prompt-status-line-setup.md) (**1999** tks) - 配置状态行显示的状态行设置代理的系统提示。

#### 斜线命令

- [Agent Prompt: /batch slash command](./system-prompts_zh/agent-prompt-batch-slash-command.md) (**1106** tks) - 在代码库中编排大型、可并行更改的说明。
- [Agent Prompt: /pr-comments slash command](./system-prompts_zh/agent-prompt-pr-comments-slash-command.md) (**402** tks) - 获取和显示 GitHub PR 评论的系统提示。
- [Agent Prompt: /review-pr slash command](./system-prompts_zh/agent-prompt-review-pr-slash-command.md) (**211** tks) - 通过代码分析查看 GitHub 拉取请求的系统提示。
- [Agent Prompt: /schedule slash command](./system-prompts_zh/agent-prompt-schedule-slash-command.md) (**2468** tks) - 指导用户通过 Anthropic 云 API 在 cron 触发器上计划、更新、列出或运行远程 Claude Code 代理。
- [Agent Prompt: /security-review slash command](./system-prompts_zh/agent-prompt-security-review-slash-command.md) (**2607** tks) - 全面的安全审查提示，用于分析代码更改，重点关注可利用的漏洞。

#### 实用程序- [Agent Prompt: Agent Hook](./system-prompts_zh/agent-prompt-agent-hook.md) (**133** tks) - 提示输入“代理挂钩”。
- [Agent Prompt: Auto mode rule reviewer](./system-prompts_zh/agent-prompt-auto-mode-rule-reviewer.md) (**257** tks) - 审查和批评用户定义的自动模式分类器规则的清晰度、完整性、冲突和可操作性。
- [Agent Prompt: Bash command description writer](./system-prompts_zh/agent-prompt-bash-command-description-writer.md) (**207** tks) - 为 bash 命令生成清晰、简洁的主动语态命令描述的说明。
- [Agent Prompt: Bash command prefix detection](./system-prompts_zh/agent-prompt-bash-command-prefix-detection.md) (**823** tks) - 用于检测命令前缀和命令注入的系统提示。
- [Agent Prompt: Claude guide agent](./system-prompts_zh/agent-prompt-claude-guide-agent.md) (**734** tks) - claude-guide Agent 的系统提示，帮助用户有效理解和使用 Claude Code、Claude Agent SDK 和 Claude API。
- [Agent Prompt: Coding session title generator](./system-prompts_zh/agent-prompt-coding-session-title-generator.md) (**181** tks) - 生成编码会话的标题。
- [Agent Prompt: Conversation summarization](./system-prompts_zh/agent-prompt-conversation-summarization.md) (**1121** tks) - 创建详细对话摘要的系统提示。
- [Agent Prompt: Determine which memory files to attach](./system-prompts_zh/agent-prompt-determine-which-memory-files-to-attach.md) (**218** tks) - 用于确定为主代理附加哪些内存文件的代理。
- [Agent Prompt: Dream memory consolidation](./system-prompts_zh/agent-prompt-dream-memory-consolidation.md) (**727** tks) - 指示代理执行多阶段内存整合过程 - 面向现有内存，从日志和记录中收集最新信号，将更新合并到主题文件中，并修剪索引。
- [Agent Prompt: General purpose](./system-prompts_zh/agent-prompt-general-purpose.md) (**285** tks) - 通用子代理的系统提示，该子代理在代码库中搜索、分析和编辑代码，同时向调用者简明地报告结果。
- [Agent Prompt: Hook condition evaluator](./system-prompts_zh/agent-prompt-hook-condition-evaluator.md) (**78** tks) - 用于评估Claude Code中的挂钩条件的系统提示。
- [Agent Prompt: Prompt Suggestion Generator v2](./system-prompts_zh/agent-prompt-prompt-suggestion-generator-v2.md) (**296** tks) - 用于为 Claude Code 生成提示建议的 V2 指令。
- [Agent Prompt: Quick PR creation](./system-prompts_zh/agent-prompt-quick-pr-creation.md) (**806** tks) - 简化了使用预填充上下文创建提交和拉取请求的提示。
- [Agent Prompt: Quick git commit](./system-prompts_zh/agent-prompt-quick-git-commit.md) (**510** tks) - 简化了使用预填充上下文创建单个 git 提交的提示。
- [Agent Prompt: Recent Message Summarization](./system-prompts_zh/agent-prompt-recent-message-summarization.md) (**724** tks) - 用于总结最近消息的代理提示。
- [Agent Prompt: Security monitor for autonomous agent actions (first part)](./system-prompts_zh/agent-prompt-security-monitor-for-autonomous-agent-actions-first-part.md) (**2726** tks) - 指示 Claude 充当安全监视器，根据阻止/允许规则评估自主编码代理操作，以防止提示注入、范围蔓延和意外损坏。
- [Agent Prompt: Security monitor for autonomous agent actions (second part)](./system-prompts_zh/agent-prompt-security-monitor-for-autonomous-agent-actions-second-part.md) (**3150** tks) - 定义环境上下文、阻止规则并允许例外，以控制代理可以执行或不可以执行哪些工具操作。
- [Agent Prompt: Session Search Assistant](./system-prompts_zh/agent-prompt-session-search-assistant.md) (**439** tks) - 会话搜索助手的代理提示，该助手根据用户查询和元数据查找相关会话。
- [Agent Prompt: Session memory update instructions](./system-prompts_zh/agent-prompt-session-memory-update-instructions.md) (**756** tks) - 在对话期间更新会话内存文件的说明。
- [Agent Prompt: Session title and branch generation](./system-prompts_zh/agent-prompt-session-title-and-branch-generation.md) (**307** tks) - 用于生成简洁会话标题和 git 分支名称的代理。
- [Agent Prompt: Update Magic Docs](./system-prompts_zh/agent-prompt-update-magic-docs.md) (**718** tks) - 提示输入 magic-docs 代理。
- [Agent Prompt: Verification specialist](./system-prompts_zh/agent-prompt-verification-specialist.md) (**2866** tks) - 系统提示验证​​子代理，该子代理通过运行构建、测试套件、linter 和对抗性探针来对抗性地测试实现，然后发出 PASS/FAIL/PARTIAL 判决。
- [Agent Prompt: WebFetch summarizer](./system-prompts_zh/agent-prompt-webfetch-summarizer.md) (**189** tks) - 提示代理汇总来自主模型的 WebFetch 的详细输出。
- [Agent Prompt: Worker fork execution](./system-prompts_zh/agent-prompt-worker-fork-execution.md) (**404** tks) - 分叉工作子代理的系统提示，该子代理直接执行指令而不生成更多子代理，然后报告结构化结果。

＃＃＃ 数据Claude Code 中嵌入的各种模板文件的内容。- [Data: Agent SDK patterns — Python](./system-prompts_zh/data-agent-sdk-patterns-python.md) (**2656** tks) - Python Agent SDK 模式，包括自定义工具、挂钩、子代理、MCP 集成和会话恢复。
- [Data: Agent SDK patterns — TypeScript](./system-prompts_zh/data-agent-sdk-patterns-typescript.md) (**1529** tks) - TypeScript Agent SDK 模式，包括基本代理、挂钩、子代理和 MCP 集成。
- [Data: Agent SDK reference — Python](./system-prompts_zh/data-agent-sdk-reference-python.md) (**3299** tks) - Python Agent SDK 参考，包括安装、快速启动、通过 MCP 的自定义工具和挂钩。
- [Data: Agent SDK reference — TypeScript](./system-prompts_zh/data-agent-sdk-reference-typescript.md) (**2943** tks) - TypeScript Agent SDK 参考，包括安装、快速启动、自定义工具和挂钩。
- [Data: Claude API reference — C#](./system-prompts_zh/data-claude-api-reference-c.md) (**4341** tks) - C# SDK 参考，包括安装、客户端初始化、基本请求、流式传输和工具使用。
- [Data: Claude API reference — Go](./system-prompts_zh/data-claude-api-reference-go.md) (**4294** tks) - Go SDK 参考。
- [Data: Claude API reference — Java](./system-prompts_zh/data-claude-api-reference-java.md) (**4506** tks) - Java SDK 参考，包括安装、客户端初始化、基本请求、流式传输和 Beta 工具使用。
- [Data: Claude API reference — PHP](./system-prompts_zh/data-claude-api-reference-php.md) (**3486** tks) - PHP SDK 参考。
- [Data: Claude API reference — Python](./system-prompts_zh/data-claude-api-reference-python.md) (**3549** tks) - Python SDK参考，包括安装、客户端初始化、基本请求、思考和多轮对话。
- [Data: Claude API reference — Ruby](./system-prompts_zh/data-claude-api-reference-ruby.md) (**923** tks) - Ruby SDK 参考，包括安装、客户端初始化、基本请求、流式传输和 Beta 工具运行程序。
- [Data: Claude API reference — TypeScript](./system-prompts_zh/data-claude-api-reference-typescript.md) (**2881** tks) - TypeScript SDK 参考，包括安装、客户端初始化、基本请求、思考和多轮对话。
- [Data: Claude API reference — cURL](./system-prompts_zh/data-claude-api-reference-curl.md) (**2174** tks) - Claude API 的原始 API 参考，用于与 cURL 或原始 HTTP 一起使用。
- [Data: Claude model catalog](./system-prompts_zh/data-claude-model-catalog.md) (**2295** tks) - 当前和旧版 Claude 模型的目录，包含准确的模型 ID、别名、上下文窗口和定价。
- [Data: Files API reference — Python](./system-prompts_zh/data-files-api-reference-python.md) (**1334** tks) - Python 文件 API 参考，包括文件上传、列表、删除和消息中的使用。
- [Data: Files API reference — TypeScript](./system-prompts_zh/data-files-api-reference-typescript.md) (**797** tks) - TypeScript 文件 API 参考，包括文件上传、列表、删除和消息中的使用。
- [Data: GitHub Actions workflow for @claude mentions](./system-prompts_zh/data-github-actions-workflow-for-claude-mentions.md) (**527** tks) - 用于通过 @claude 提及触发 Claude 代码的 GitHub Actions 工作流程模板。
- [Data: GitHub App installation PR description](./system-prompts_zh/data-github-app-installation-pr-description.md) (**424** tks) - 安装 Claude Code GitHub 应用程序集成时的 PR 描述模板。
- [Data: HTTP error codes reference](./system-prompts_zh/data-http-error-codes-reference.md) (**1922** tks) - Claude API 返回的 HTTP 错误代码以及常见原因和处理策略的参考。
- [Data: Live documentation sources](./system-prompts_zh/data-live-documentation-sources.md) (**2336** tks) - WebFetch URL，用于从官方来源获取当前的 Claude API 和 Agent SDK 文档。
- [Data: Message Batches API reference — Python](./system-prompts_zh/data-message-batches-api-reference-python.md) (**1544** tks) - Python 批处理 API 参考，包括批量创建、状态轮询和结果检索，成本为 50%。
- [Data: Prompt Caching — Design & Optimization](./system-prompts_zh/data-prompt-caching-design-optimization.md) (**2657** tks) - 有关如何设计提示构建代码以实现有效缓存的文档，包括放置模式和反模式。
- [Data: Session memory template](./system-prompts_zh/data-session-memory-template.md) (**292** tks) - 会话内存 `summary.md` 文件的模板结构。
- [Data: Streaming reference — Python](./system-prompts_zh/data-streaming-reference-python.md) (**1528** tks) - Python 流参考，包括同步/异步流和处理不同的内容类型。
- ZXQMASK0091QX​​Z (**1703** tks) - TypeScript 流参考，包括基本流和处理不同的内容类型。
- [Data: Tool use concepts](./system-prompts_zh/data-tool-use-concepts.md) (**3721** tks) - Claude API 工具使用的概念基础，包括工具定义、工具选择和最佳实践。- [Data: Tool use reference — Python](./system-prompts_zh/data-tool-use-reference-python.md) (**5106** tks) - Python 工具使用参考，包括工具运行程序、手动代理循环、代码执行和结构化输出。
- [Data: Tool use reference — TypeScript](./system-prompts_zh/data-tool-use-reference-typescript.md) (**5033** tks) - TypeScript 工具使用参考，包括工具运行器、手动代理循环、代码执行和结构化输出。

###系统提示

主系统部分提示。- [System Prompt: Advisor tool instructions](./system-prompts_zh/system-prompt-advisor-tool-instructions.md) (**443** tks) - Advisor 工具的使用说明。
- [System Prompt: Agent Summary Generation](./system-prompts_zh/system-prompt-agent-summary-generation.md) (**178** tks) - 用于生成“代理摘要”的系统提示。
- [System Prompt: Agent memory instructions](./system-prompts_zh/system-prompt-agent-memory-instructions.md) (**337** tks) - 在代理系统提示中包含内存更新指导的说明。
- [System Prompt: Agent thread notes](./system-prompts_zh/system-prompt-agent-thread-notes.md) (**156** tks) - 代理线程的行为指南，涵盖绝对路径、响应格式、避免表情符号和工具调用标点符号。
- [System Prompt: Auto mode](./system-prompts_zh/system-prompt-auto-mode.md) (**255** tks) - 连续任务执行，类似于后台代理。
- [System Prompt: Avoiding Unnecessary Sleep Commands (part of PowerShell tool description)](./system-prompts_zh/system-prompt-avoiding-unnecessary-sleep-commands-part-of-powershell-tool-description.md) (**182** tks) - 避免 PowerShell 脚本中不必要的睡眠命令的指南，包括等待和通知的替代方法。
- [System Prompt: Buddy Mode](./system-prompts_zh/system-prompt-buddy-mode.md) (**205** tks) - 生成居住在终端中的编码同伴并对开发人员的工作进行评论的说明，重点是根据给定的统计数据和灵感词创建令人难忘的、独特的个性。
- [System Prompt: Censoring assistance with malicious activities](./system-prompts_zh/system-prompt-censoring-assistance-with-malicious-activities.md) (**98** tks) - 协助授权安全测试、防御安全、CTF 挑战和教育环境，同时审查恶意活动请求的指南。
- [System Prompt: Chrome browser MCP tools](./system-prompts_zh/system-prompt-chrome-browser-mcp-tools.md) (**156** tks) - 使用前通过 MCPSearch 加载 Chrome 浏览器 MCP 工具的说明。
- [System Prompt: Claude in Chrome browser automation](./system-prompts_zh/system-prompt-claude-in-chrome-browser-automation.md) (**759** tks) - 在 Chrome 浏览器自动化工具中有效使用 Claude 的说明。
- [System Prompt: Context compaction summary](./system-prompts_zh/system-prompt-context-compaction-summary.md) (**278** tks) - 用于上下文压缩摘要的提示（对于 SDK）。
- [System Prompt: Description part of memory instructions](./system-prompts_zh/system-prompt-description-part-of-memory-instructions.md) (**148** tks) - 用于描述内存是什么的字段。  这是指导Claude如何创造记忆的更大努力的一部分。
- [System Prompt: Doing tasks (ambitious tasks)](./system-prompts_zh/system-prompt-doing-tasks-ambitious-tasks.md) (**47** tks) - 允许用户完成雄心勃勃的任务；尊重用户对范围的判断。
- [System Prompt: Doing tasks (help and feedback)](./system-prompts_zh/system-prompt-doing-tasks-help-and-feedback.md) (**24** tks) - 如何通知用户有关帮助和反馈渠道的信息。
- [System Prompt: Doing tasks (minimize file creation)](./system-prompts_zh/system-prompt-doing-tasks-minimize-file-creation.md) (**47** tks) - 更喜欢编辑现有文件而不是创建新文件。
- [System Prompt: Doing tasks (no compatibility hacks)](./system-prompts_zh/system-prompt-doing-tasks-no-compatibility-hacks.md) (**52** tks) - 完全删除未使用的代码而不是添加兼容性垫片。
- [System Prompt: Doing tasks (no premature abstractions)](./system-prompts_zh/system-prompt-doing-tasks-no-premature-abstractions.md) (**72** tks) - 不要为一次性操作或假设需求创建抽象。
- [System Prompt: Doing tasks (no time estimates)](./system-prompts_zh/system-prompt-doing-tasks-no-time-estimates.md) (**47** tks) - 避免给出时间估计或预测。
- [System Prompt: Doing tasks (no unnecessary additions)](./system-prompts_zh/system-prompt-doing-tasks-no-unnecessary-additions.md) (**78** tks) - 请勿添加超出要求的功能、重构或改进。
- [System Prompt: Doing tasks (no unnecessary error handling)](./system-prompts_zh/system-prompt-doing-tasks-no-unnecessary-error-handling.md) (**64** tks) - 不要为不可能的场景添加错误处理；仅在边界处验证。
- [System Prompt: Doing tasks (read before modifying)](./system-prompts_zh/system-prompt-doing-tasks-read-before-modifying.md) (**46** tks) - 在建议修改之前阅读并理解现有代码。
- [System Prompt: Doing tasks (security)](./system-prompts_zh/system-prompt-doing-tasks-security.md) (**67** tks) - 避免引入注入、XSS 等安全漏洞。
- [System Prompt: Doing tasks (software engineering focus)](./system-prompts_zh/system-prompt-doing-tasks-software-engineering-focus.md) (**104** tks) - 用户主要请求软件工程任务；在该上下文中解释说明。
- [System Prompt: Executing actions with care](./system-prompts_zh/system-prompt-executing-actions-with-care.md) (**590** tks) - 仔细执行操作的说明。
- [System Prompt: Fork usage guidelines](./system-prompts_zh/system-prompt-fork-usage-guidelines.md) (**419** tks) - 何时分叉子代理的说明以及禁止在飞行中读取分叉输出或伪造分叉结果的规则。
- [System Prompt: Git status](./system-prompts_zh/system-prompt-git-status.md) (**37** tks) - 在对话开始时显示当前 git 状态的系统提示。
- [System Prompt: Hooks Configuration](./system-prompts_zh/system-prompt-hooks-configuration.md) (**1493** tks) - 系统提示挂钩配置。  用于上述 Claude Code 配置技能。- [System Prompt: How to use the SendUserMessage tool](./system-prompts_zh/system-prompt-how-to-use-the-sendusermessage-tool.md) (**283** tks) - SendUserMessage 工具的使用说明。
- [System Prompt: Insights at a glance summary](./system-prompts_zh/system-prompt-insights-at-a-glance-summary.md) (**569** tks) - 为洞察报告生成简洁的 4 部分摘要（有效内容、障碍、速效、雄心勃勃的工作流程）。
- [System Prompt: Insights friction analysis](./system-prompts_zh/system-prompt-insights-friction-analysis.md) (**139** tks) - 分析汇总的使用数据以识别摩擦模式并对重复出现的问题进行分类。
- [System Prompt: Insights on the horizon](./system-prompts_zh/system-prompt-insights-on-the-horizon.md) (**148** tks) - 确定雄心勃勃的未来工作流程和自主人工智能辅助开发的机会。
- [System Prompt: Insights session facets extraction](./system-prompts_zh/system-prompt-insights-session-facets-extraction.md) (**310** tks) - 从单个Claude Code会话记录中提取结构化方面（目标类别、满意度、摩擦）。
- [System Prompt: Insights suggestions](./system-prompts_zh/system-prompt-insights-suggestions.md) (**748** tks) - 生成可操作的建议，包括 CLAUDE.md 添加、要尝试的功能和使用模式。
- [System Prompt: Learning mode (insights)](./system-prompts_zh/system-prompt-learning-mode-insights.md) (**142** tks) - 当学习模式处于活动状态时提供教育见解的说明。
- [System Prompt: Learning mode](./system-prompts_zh/system-prompt-learning-mode.md) (**1042** tks) - 主系统提示带有人工协作指令的学习模式。
- [System Prompt: MCP Tool Result Truncation](./system-prompts_zh/system-prompt-mcp-tool-result-truncation.md) (**147** tks) - 处理 MCP 工具长输出的指南，包括何时使用直接文件查询与子代理进行分析。
- [System Prompt: Memory description of user feedback](./system-prompts_zh/system-prompt-memory-description-of-user-feedback.md) (**139** tks) - 描述用户反馈记忆类型，存储有关工作方法的指导，强调记录成功和失败并检查与团队记忆的矛盾。
- [System Prompt: Minimal mode](./system-prompts_zh/system-prompt-minimal-mode.md) (**164** tks) - 描述最小模式的行为和约束，该模式跳过挂钩、LSP、插件、自动内存和其他功能，同时需要通过 CLI 标志的显式上下文。
- [System Prompt: One of six rules for using sleep command](./system-prompts_zh/system-prompt-one-of-six-rules-for-using-sleep-command.md) (**23** tks) - 使用 sleep 命令的六个规则之一。
- [System Prompt: Option previewer](./system-prompts_zh/system-prompt-option-previewer.md) (**151** tks) - 在并排布局中预览 UI 选项的系统提示。
- [System Prompt: Output efficiency](./system-prompts_zh/system-prompt-output-efficiency.md) (**177** tks) - 指示Claude在文本输出中保持简洁和直接，以推理答案为主，并限制对基本信息的回应。
- [System Prompt: Parallel tool call note (part of "Tool usage policy")](./system-prompts_zh/system-prompt-parallel-tool-call-note-part-of-tool-usage-policy.md) (**102** tks) - 系统提示告诉 Claude 使用并行工具调用。
- [System Prompt: Partial compaction instructions](./system-prompts_zh/system-prompt-partial-compaction-instructions.md) (**725** tks) - 当用户决定仅压缩对话的一部分时如何压缩的说明，以及结构化摘要格式和分析过程。
- [System Prompt: Phase four of plan mode](./system-prompts_zh/system-prompt-phase-four-of-plan-mode.md) (**142** tks) - 计划模式的第四阶段。
- [System Prompt: PowerShell edition for 5.1](./system-prompts_zh/system-prompt-powershell-edition-for-51.md) (**285** tks) - 提供有关 Windows PowerShell 5.1 信息的系统提示。
- [System Prompt: Remote plan mode (ultraplan)](./system-prompts_zh/system-prompt-remote-plan-mode-ultraplan.md) (**652** tks) - 在远程规划会话期间注入系统提醒，指示 Claude 探索代码库，通过 ExitPlanMode 生成丰富的图表计划，并在批准后通过拉取请求实施它。
- [System Prompt: Remote planning session](./system-prompts_zh/system-prompt-remote-planning-session.md) (**432** tks) - 系统提醒，配置远程规划会话来探索代码库，通过 ExitPlanMode 生成实施计划，并处理计划批准、拒绝或传送回用户本地终端。
- [System Prompt: Scratchpad directory](./system-prompts_zh/system-prompt-scratchpad-directory.md) (**170** tks) - 使用临时文件专用暂存器目录的说明。
- [System Prompt: Skillify Current Session](./system-prompts_zh/system-prompt-skillify-current-session.md) (**1882** tks) - 将当前会话转换为技能的系统提示。- [System Prompt: Subagent delegation examples](./system-prompts_zh/system-prompt-subagent-delegation-examples.md) (**606** tks) - 提供示例交互，展示协调器代理应如何将任务委托给子代理、处理等待状态和报告结果。
- [System Prompt: Team memory content display](./system-prompts_zh/system-prompt-team-memory-content-display.md) (**55** tks) - 渲染共享团队内存文件内容以及注入对话上下文的路径和内容。
- [System Prompt: Teammate Communication](./system-prompts_zh/system-prompt-teammate-communication.md) (**130** tks) - 集群中队友通信的系统提示。
- [System Prompt: Tone and style (code references)](./system-prompts_zh/system-prompt-tone-and-style-code-references.md) (**39** tks) - 引用代码时包含 file_path:line_number 的指令。
- [System Prompt: Tone and style (concise output — short)](./system-prompts_zh/system-prompt-tone-and-style-concise-output-short.md) (**16** tks) - 简短回应的说明。
- [System Prompt: Tool execution denied](./system-prompts_zh/system-prompt-tool-execution-denied.md) (**144** tks) - 工具执行被拒绝时的系统提示。
- [System Prompt: Tool usage (create files)](./system-prompts_zh/system-prompt-tool-usage-create-files.md) (**30** tks) - 更喜欢使用 Write 工具而不是 cat heredoc 或 echo 重定向。
- [System Prompt: Tool usage (delegate exploration)](./system-prompts_zh/system-prompt-tool-usage-delegate-exploration.md) (**95** tks) - 使用任务工具进行更广泛的代码库探索和深入研究。
- [System Prompt: Tool usage (direct search)](./system-prompts_zh/system-prompt-tool-usage-direct-search.md) (**39** tks) - 直接使用 Glob/Grep 进行简单的定向搜索。
- [System Prompt: Tool usage (edit files)](./system-prompts_zh/system-prompt-tool-usage-edit-files.md) (**26** tks) - 更喜欢编辑工具而不是 sed/awk。
- [System Prompt: Tool usage (read files)](./system-prompts_zh/system-prompt-tool-usage-read-files.md) (**29** tks) - 更喜欢读取工具而不是 cat/head/tail/sed。
- [System Prompt: Tool usage (reserve Bash)](./system-prompts_zh/system-prompt-tool-usage-reserve-bash.md) (**75** tks) - 专门为系统命令和终端操作保留 Bash 工具。
- [System Prompt: Tool usage (search content)](./system-prompts_zh/system-prompt-tool-usage-search-content.md) (**30** tks) - 更喜欢 Grep 工具而不是 grep 或 rg。
- [System Prompt: Tool usage (search files)](./system-prompts_zh/system-prompt-tool-usage-search-files.md) (**26** tks) - 更喜欢使用 Glob 工具而不是 find 或 ls。
- [System Prompt: Tool usage (skill invocation)](./system-prompts_zh/system-prompt-tool-usage-skill-invocation.md) (**102** tks) - 斜杠命令通过技能工具调用用户可调用的技能。
- [System Prompt: Tool usage (subagent guidance)](./system-prompts_zh/system-prompt-tool-usage-subagent-guidance.md) (**103** tks) - 关于何时以及如何有效使用子代理的指南。
- [System Prompt: Tool usage (task management)](./system-prompts_zh/system-prompt-tool-usage-task-management.md) (**70** tks) - 使用 TodoWrite 分解和跟踪工作进度。
- [System Prompt: Worker instructions](./system-prompts_zh/system-prompt-worker-instructions.md) (**272** tks) - 员工在实施变更时应遵循的说明。
- [System Prompt: Writing subagent prompts](./system-prompts_zh/system-prompt-writing-subagent-prompts.md) (**287** tks) - 将任务委派给子代理时编写有效提示的指南，涵盖上下文继承与新子代理场景。

### 系统提醒

大型系统提醒的文本。- [System Reminder: /btw side question](./system-prompts_zh/system-reminder-btw-side-question.md) (**244** tks) - 无需工具的 /btw 斜杠命令边问题的系统提醒。
- [System Reminder: Agent mention](./system-prompts_zh/system-reminder-agent-mention.md) (**45** tks) - 用户想要调用代理的通知。
- [System Reminder: Compact file reference](./system-prompts_zh/system-reminder-compact-file-reference.md) (**57** tks) - 对话摘要之前读取的文件参考。
- [System Reminder: Exited plan mode](./system-prompts_zh/system-reminder-exited-plan-mode.md) (**73** tks) - 退出计划模式时的通知。
- [System Reminder: File exists but empty](./system-prompts_zh/system-reminder-file-exists-but-empty.md) (**27** tks) - 读取空文件时发出警告。
- [System Reminder: File modified by user or linter](./system-prompts_zh/system-reminder-file-modified-by-user-or-linter.md) (**97** tks) - 文件被外部修改的通知。
- [System Reminder: File opened in IDE](./system-prompts_zh/system-reminder-file-opened-in-ide.md) (**37** tks) - 用户在 IDE 中打开文件的通知。
- [System Reminder: File shorter than offset](./system-prompts_zh/system-reminder-file-shorter-than-offset.md) (**59** tks) - 文件读取偏移量超过文件长度时发出警告。
- [System Reminder: File truncated](./system-prompts_zh/system-reminder-file-truncated.md) (**74** tks) - 文件因大小而被截断的通知。
- [System Reminder: Hook additional context](./system-prompts_zh/system-reminder-hook-additional-context.md) (**35** tks) - 来自钩子的附加上下文。
- [System Reminder: Hook blocking error](./system-prompts_zh/system-reminder-hook-blocking-error.md) (**52** tks) - 来自阻塞挂钩命令的错误。
- [System Reminder: Hook stopped continuation prefix](./system-prompts_zh/system-reminder-hook-stopped-continuation-prefix.md) (**12** tks) - 钩子停止继续消息的前缀。
- [System Reminder: Hook stopped continuation](./system-prompts_zh/system-reminder-hook-stopped-continuation.md) (**30** tks) - 当钩子停止继续时的消息。
- [System Reminder: Hook success](./system-prompts_zh/system-reminder-hook-success.md) (**29** tks) - 来自挂钩的成功消息。
- [System Reminder: Invoked skills](./system-prompts_zh/system-reminder-invoked-skills.md) (**33** tks) - 此会话中调用的技能列表。
- [System Reminder: Lines selected in IDE](./system-prompts_zh/system-reminder-lines-selected-in-ide.md) (**66** tks) - 有关用户在 IDE 中选择的行的通知。
- [System Reminder: MCP resource no content](./system-prompts_zh/system-reminder-mcp-resource-no-content.md) (**41** tks) - 当 MCP 资源没有内容时显示。
- [System Reminder: MCP resource no displayable content](./system-prompts_zh/system-reminder-mcp-resource-no-displayable-content.md) (**43** tks) - 当 MCP 资源没有可显示内容时显示。
- [System Reminder: Malware analysis after Read tool call](./system-prompts_zh/system-reminder-malware-analysis-after-read-tool-call.md) (**87** tks) - 分析恶意软件而不对其进行改进或增强的说明。
- [System Reminder: Memory file contents](./system-prompts_zh/system-reminder-memory-file-contents.md) (**36** tks) - 按路径列出的内存文件的内容。
- [System Reminder: Nested memory contents](./system-prompts_zh/system-reminder-nested-memory-contents.md) (**33** tks) - 嵌套内存文件的内容。
- [System Reminder: New diagnostics detected](./system-prompts_zh/system-reminder-new-diagnostics-detected.md) (**35** tks) - 有关新诊断问题的通知。
- [System Reminder: Output style active](./system-prompts_zh/system-reminder-output-style-active.md) (**32** tks) - 输出样式处于活动状态的通知。
- [System Reminder: Plan file reference](./system-prompts_zh/system-reminder-plan-file-reference.md) (**62** tks) - 对现有计划文件的引用。
- [System Reminder: Plan mode is active (5-phase)](./system-prompts_zh/system-reminder-plan-mode-is-active-5-phase.md) (**1297** tks) - 增强计划模式系统提醒，具有并行探索和多代理计划。
- [System Reminder: Plan mode is active (iterative)](./system-prompts_zh/system-reminder-plan-mode-is-active-iterative.md) (**936** tks) - 具有用户访谈工作流程的主要代理的迭代计划模式系统提醒。
- [System Reminder: Plan mode is active (subagent)](./system-prompts_zh/system-reminder-plan-mode-is-active-subagent.md) (**307** tks) - 简化子代理的计划模式系统提醒。
- [System Reminder: Plan mode re-entry](./system-prompts_zh/system-reminder-plan-mode-re-entry.md) (**236** tks) - 当用户在之前通过 Shift+Tab 或批准 Claude 的计划退出后进入计划模式时发送的系统提醒。
- [System Reminder: Session continuation](./system-prompts_zh/system-reminder-session-continuation.md) (**37** tks) - 会话从另一台计算机继续进行的通知。
- [System Reminder: Task tools reminder](./system-prompts_zh/system-reminder-task-tools-reminder.md) (**123** tks) - 提醒使用任务跟踪工具。
- [System Reminder: Team Coordination](./system-prompts_zh/system-reminder-team-coordination.md) (**250** tks) - 团队协调的系统提醒。
- [System Reminder: Team Shutdown](./system-prompts_zh/system-reminder-team-shutdown.md) (**136** tks) - 团队关闭的系统提醒。
- [System Reminder: TodoWrite reminder](./system-prompts_zh/system-reminder-todowrite-reminder.md) (**98** tks) - 提醒使用 TodoWrite 工具进行任务跟踪。
- [System Reminder: Token usage](./system-prompts_zh/system-reminder-token-usage.md) (**39** tks) - 当前代币使用统计数据。
- [System Reminder: USD budget](./system-prompts_zh/system-reminder-usd-budget.md) (**42** tks) - 当前美元预算统计数据。
- [System Reminder: Ultraplan mode](./system-prompts_zh/system-reminder-ultraplan-mode.md) (**437** tks) - 使用 Ultraplan 模式通过多智能体探索和批判创建详细实施计划的系统提醒。
- [System Reminder: Verify plan reminder](./system-prompts_zh/system-reminder-verify-plan-reminder.md) (**47** tks) - 提醒验证已完成的计划。

### 内置工具说明- [Tool Description: AskUserQuestion](./system-prompts_zh/tool-description-askuserquestion.md) (**287** tks) - 用于询问用户问题的工具说明。
- [Tool Description: Computer](./system-prompts_zh/tool-description-computer.md) (**161** tks) - Chrome 浏览器计算机自动化工具的主要描述。
- [Tool Description: Config](./system-prompts_zh/tool-description-config.md) (**275** tks) - 用于获取和设置 Claude Code 配置设置的工具，包含使用说明和可配置设置列表。
- [Tool Description: CronCreate](./system-prompts_zh/tool-description-croncreate.md) (**948** tks) - 描述 CronCreate 工具，用于通过抖动和非分钟调度指导对一次性或基于 cron 的循环作业进行排队。
- [Tool Description: Edit](./system-prompts_zh/tool-description-edit.md) (**240** tks) - 用于在文件中执行精确字符串替换的工具。
- [Tool Description: EnterPlanMode](./system-prompts_zh/tool-description-enterplanmode.md) (**878** tks) - 用于进入计划模式以探索和设计实施方法的工具描述。
- [Tool Description: EnterWorktree](./system-prompts_zh/tool-description-enterworktree.md) (**359** tks) - EnterWorktree 工具的工具描述。
- [Tool Description: ExitPlanMode](./system-prompts_zh/tool-description-exitplanmode.md) (**417** tks) - ExitPlanMode 工具的描述，它呈现一个计划对话框供用户批准。
- [Tool Description: ExitWorktree](./system-prompts_zh/tool-description-exitworktree.md) (**527** tks) - 粗略地说，与 ExitWorktree 相反。
- [Tool Description: Grep](./system-prompts_zh/tool-description-grep.md) (**300** tks) - 使用 ripgrep 进行内容搜索的工具描述。
- [Tool Description: LSP](./system-prompts_zh/tool-description-lsp.md) (**255** tks) - LSP 工具的描述。
- [Tool Description: NotebookEdit](./system-prompts_zh/tool-description-notebookedit.md) (**121** tks) - 用于编辑 Jupyter 笔记本单元的工具描述。
- [Tool Description: PowerShell](./system-prompts_zh/tool-description-powershell.md) (**1455** tks) - 描述 PowerShell 命令执行工具，包括语法指导、超时设置以及在文件操作时优先使用专用工具而不是 PowerShell 的说明。
- [Tool Description: ReadFile](./system-prompts_zh/tool-description-readfile.md) (**412** tks) - 读取文件的工具说明。
- [Tool Description: SendMessageTool](./system-prompts_zh/tool-description-sendmessagetool.md) (**362** tks) - SendMessageTool 的代理团队版本。
- [Tool Description: Skill](./system-prompts_zh/tool-description-skill.md) (**326** tks) - 在主要对话中执行技能的工具描述。
- [Tool Description: Sleep](./system-prompts_zh/tool-description-sleep.md) (**154** tks) - 用于等待/睡眠的工具，具有用户输入的早期唤醒功能。
- [Tool Description: TaskCreate](./system-prompts_zh/tool-description-taskcreate.md) (**499** tks) - TaskCreate 工具的工具描述。
- [Tool Description: TeamDelete](./system-prompts_zh/tool-description-teamdelete.md) (**154** tks) - TeamDelete 工具的工具描述。
- [Tool Description: TeammateTool](./system-prompts_zh/tool-description-teammatetool.md) (**1585** tks) - 用于管理团队和协调群体中的队友的工具。
- [Tool Description: TodoWrite](./system-prompts_zh/tool-description-todowrite.md) (**2037** tks) - 用于创建和管理任务列表的工具描述。
- [Tool Description: WebFetch](./system-prompts_zh/tool-description-webfetch.md) (**297** tks) - Web 获取功能的工具描述。
- [Tool Description: WebSearch](./system-prompts_zh/tool-description-websearch.md) (**321** tks) - Web 搜索功能的工具描述。
- [Tool Description: Write](./system-prompts_zh/tool-description-write.md) (**129** tks) - 用于将文件写入本地文件系统的工具。

**一些工具说明的附加说明**- [Tool Description: Agent (usage notes)](./system-prompts_zh/tool-description-agent-usage-notes.md) (**798** tks) - 任务/代理工具的使用说明和说明，包括启动子代理、后台执行、恢复和工作树隔离的指南。
- [Tool Description: Agent (when to launch subagents)](./system-prompts_zh/tool-description-agent-when-to-launch-subagents.md) (**186** tks) - 描述何时使用代理工具 - 用于启动专门的子代理子进程来自主处理复杂的多步骤任务。
- [Tool Description: AskUserQuestion (preview field)](./system-prompts_zh/tool-description-askuserquestion-preview-field.md) (**134** tks) - 使用单选问题选项上的 HTML 预览字段来显示 UI 模型、代码片段和图表等视觉工件的说明。
- [Tool Description: Bash (Git commit and PR creation instructions)](./system-prompts_zh/tool-description-bash-git-commit-and-pr-creation-instructions.md) (**1611** tks) - 创建 git 提交和 GitHub 拉取请求的说明。
- [Tool Description: Bash (alternative — communication)](./system-prompts_zh/tool-description-bash-alternative-communication.md) (**18** tks) - Bash 工具替代方案：直接输出文本而不是 echo/printf。
- [Tool Description: Bash (alternative — content search)](./system-prompts_zh/tool-description-bash-alternative-content-search.md) (**27** tks) - Bash 工具替代方案：使用 Grep 进行内容搜索而不是 grep/rg。
- [Tool Description: Bash (alternative — edit files)](./system-prompts_zh/tool-description-bash-alternative-edit-files.md) (**27** tks) - Bash 工具替代方案：使用 Edit 代替 sed/awk 进行文件编辑。
- [Tool Description: Bash (alternative — file search)](./system-prompts_zh/tool-description-bash-alternative-file-search.md) (**26** tks) - Bash 工具替代方案：使用 Glob 进行文件搜索而不是 find/ls。
- [Tool Description: Bash (alternative — read files)](./system-prompts_zh/tool-description-bash-alternative-read-files.md) (**27** tks) - Bash 工具替代方案：使用 Read 来读取文件，而不是 cat/head/tail。
- [Tool Description: Bash (alternative — write files)](./system-prompts_zh/tool-description-bash-alternative-write-files.md) (**29** tks) - Bash 工具替代方案：使用 Write 进行文件写入，而不是 echo/cat。
- [Tool Description: Bash (built-in tools note)](./system-prompts_zh/tool-description-bash-built-in-tools-note.md) (**53** tks) - 请注意，内置工具提供比 Bash 等效工具更好的用户体验。
- [Tool Description: Bash (git — avoid destructive ops)](./system-prompts_zh/tool-description-bash-git-avoid-destructive-ops.md) (**58** tks) - Bash 工具 git 指令：考虑破坏性操作的更安全替代方案。
- [Tool Description: Bash (git — never skip hooks)](./system-prompts_zh/tool-description-bash-git-never-skip-hooks.md) (**59** tks) - Bash 工具 git 指令：除非用户请求，否则永远不要跳过挂钩或绕过签名。
- [Tool Description: Bash (git — prefer new commits)](./system-prompts_zh/tool-description-bash-git-prefer-new-commits.md) (**22** tks) - Bash 工具 git 指令：更喜欢新提交而不是修改。
- [Tool Description: Bash (maintain cwd)](./system-prompts_zh/tool-description-bash-maintain-cwd.md) (**41** tks) - Bash 工具说明：使用绝对路径并避免 cd。
- [Tool Description: Bash (no newlines)](./system-prompts_zh/tool-description-bash-no-newlines.md) (**24** tks) - Bash 工具指令：不要使用换行符分隔命令。
- [Tool Description: Bash (overview)](./system-prompts_zh/tool-description-bash-overview.md) (**19** tks) - Bash 工具描述的开头行。
- [Tool Description: Bash (parallel commands)](./system-prompts_zh/tool-description-bash-parallel-commands.md) (**72** tks) - Bash 工具指令：作为并行工具调用运行独立命令。
- [Tool Description: Bash (prefer dedicated tools)](./system-prompts_zh/tool-description-bash-prefer-dedicated-tools.md) (**71** tks) - 警告在查找、grep、cat 等方面优先使用 Bash 专用工具。
- [Tool Description: Bash (quote file paths)](./system-prompts_zh/tool-description-bash-quote-file-paths.md) (**35** tks) - Bash 工具说明：引用包含空格的文件路径。
- [Tool Description: Bash (sandbox — adjust settings)](./system-prompts_zh/tool-description-bash-sandbox-adjust-settings.md) (**26** tks) - 与用户合作在失败时调整沙箱设置。
- [Tool Description: Bash (sandbox — default to sandbox)](./system-prompts_zh/tool-description-bash-sandbox-default-to-sandbox.md) (**38** tks) - 默认为沙箱；仅当用户询问或有沙箱限制证据时才绕过。
- [Tool Description: Bash (sandbox — evidence list header)](./system-prompts_zh/tool-description-bash-sandbox-evidence-list-header.md) (**15** tks) - 沙箱引起的故障证据列表的标题。
- [Tool Description: Bash (sandbox — evidence: access denied)](./system-prompts_zh/tool-description-bash-sandbox-evidence-access-denied.md) (**15** tks) - 沙盒证据：拒绝访问允许目录之外的路径。
- [Tool Description: Bash (sandbox — evidence: network failures)](./system-prompts_zh/tool-description-bash-sandbox-evidence-network-failures.md) (**17** tks) - 沙箱证据：非白名单主机的网络连接失败。
- [Tool Description: Bash (sandbox — evidence: operation not permitted)](./system-prompts_zh/tool-description-bash-sandbox-evidence-operation-not-permitted.md) (**18** tks) - 沙盒证据：不允许操作的错误。
- [Tool Description: Bash (sandbox — evidence: unix socket errors)](./system-prompts_zh/tool-description-bash-sandbox-evidence-unix-socket-errors.md) (**11** tks) - 沙盒证据：unix 套接字连接错误。
- [Tool Description: Bash (sandbox — explain restriction)](./system-prompts_zh/tool-description-bash-sandbox-explain-restriction.md) (**36** tks) - 解释哪个沙箱限制导致了失败。
- [Tool Description: Bash (sandbox — failure evidence condition)](./system-prompts_zh/tool-description-bash-sandbox-failure-evidence-condition.md) (**48** tks) - 条件：命令失败，有沙箱限制的证据。
- [Tool Description: Bash (sandbox — mandatory mode)](./system-prompts_zh/tool-description-bash-sandbox-mandatory-mode.md) (**34** tks) - 策略：所有命令必须在沙箱模式下运行。- [Tool Description: Bash (sandbox — no exceptions)](./system-prompts_zh/tool-description-bash-sandbox-no-exceptions.md) (**17** tks) - 在任何情况下命令都不能在沙箱之外运行。
- [Tool Description: Bash (sandbox — no sensitive paths)](./system-prompts_zh/tool-description-bash-sandbox-no-sensitive-paths.md) (**36** tks) - 不建议将敏感路径添加到沙箱白名单。
- [Tool Description: Bash (sandbox — per-command)](./system-prompts_zh/tool-description-bash-sandbox-per-command.md) (**52** tks) - 单独处理每个命令；默认为沙箱以供将来的命令使用。
- [Tool Description: Bash (sandbox — response header)](./system-prompts_zh/tool-description-bash-sandbox-response-header.md) (**17** tks) - 当看到沙箱引起的故障时如何响应的标头。
- [Tool Description: Bash (sandbox — retry without sandbox)](./system-prompts_zh/tool-description-bash-sandbox-retry-without-sandbox.md) (**33** tks) - 在沙箱失败时立即使用危险禁用沙箱重试。
- [Tool Description: Bash (sandbox — tmpdir)](./system-prompts_zh/tool-description-bash-sandbox-tmpdir.md) (**58** tks) - 在沙盒模式下使用 $TMPDIR 作为临时文件。
- [Tool Description: Bash (sandbox — user permission prompt)](./system-prompts_zh/tool-description-bash-sandbox-user-permission-prompt.md) (**14** tks) - 请注意，禁用沙箱将提示用户授予权限。
- [Tool Description: Bash (semicolon usage)](./system-prompts_zh/tool-description-bash-semicolon-usage.md) (**29** tks) - Bash 工具指令：当顺序很重要但失败不重要时使用分号。
- [Tool Description: Bash (sequential commands)](./system-prompts_zh/tool-description-bash-sequential-commands.md) (**42** tks) - Bash 工具指令：带有 && 的链相关命令。
- [Tool Description: Bash (sleep — keep short)](./system-prompts_zh/tool-description-bash-sleep-keep-short.md) (**29** tks) - Bash 工具说明：将睡眠持续时间保持在 1-5 秒。
- [Tool Description: Bash (sleep — no polling background tasks)](./system-prompts_zh/tool-description-bash-sleep-no-polling-background-tasks.md) (**37** tks) - Bash 工具说明：不轮询后台任务，等待通知。
- [Tool Description: Bash (sleep — run immediately)](./system-prompts_zh/tool-description-bash-sleep-run-immediately.md) (**21** tks) - Bash 工具指令：不要在可以立即运行的命令之间休眠。
- [Tool Description: Bash (sleep — use check commands)](./system-prompts_zh/tool-description-bash-sleep-use-check-commands.md) (**34** tks) - Bash 工具说明：轮询时使用检查命令而不是休眠。
- [Tool Description: Bash (timeout)](./system-prompts_zh/tool-description-bash-timeout.md) (**83** tks) - Bash 工具说明：可选超时配置。
- [Tool Description: Bash (verify parent directory)](./system-prompts_zh/tool-description-bash-verify-parent-directory.md) (**38** tks) - Bash 工具说明：创建文件之前验证父目录。
- [Tool Description: Bash (working directory)](./system-prompts_zh/tool-description-bash-working-directory.md) (**37** tks) - 关于工作目录持久性和 shell 状态的 Bash 工具注释。
- [Tool Description: SendMessageTool (non-agent-teams)](./system-prompts_zh/tool-description-sendmessagetool-non-agent-teams.md) (**133** tks) - 发送一条用户会阅读的消息，很好地描述了这个工具。
- [Tool Description: TaskList (teammate workflow)](./system-prompts_zh/tool-description-tasklist-teammate-workflow.md) (**133** tks) - 附加到任务列表工具描述的条件部分。
- [Tool Description: ToolSearch (second part)](./system-prompts_zh/tool-description-toolsearch-second-part.md) (**202** tks) - 大部分工具描述。
- [Tool Description: request_teach_access (part of teach mode)](./system-prompts_zh/tool-description-request_teach_access-part-of-teach-mode.md) (**139** tks) - 描述一种工具，该工具请求权限以使用全屏工具提示覆盖而不是直接访问来引导用户逐步完成任务。
- [Tool Parameter: Computer action](./system-prompts_zh/tool-parameter-computer-action.md) (**251** tks) - Chrome 浏览器计算机工具的操作参数选项。

### 技能

针对专门任务的内置技能提示。- [Skill: /init CLAUDE.md and skill setup (new version)](./system-prompts_zh/skill-init-claudemd-and-skill-setup-new-version.md) (**4618** tks) - 用于在当前存储库中设置 CLAUDE.md 和相关技能/挂钩的全面入门流程，包括代码库探索、用户访谈和迭代提案细化。
- [Skill: /loop slash command](./system-prompts_zh/skill-loop-slash-command.md) (**1040** tks) - 将用户输入解析为间隔和提示，将间隔转换为 cron 表达式，并安排重复任务。
- [Skill: /stuck slash command](./system-prompts_zh/skill-stuck-slash-command.md) (**964** tks) - 诊断冻结或缓慢的Claude Code会话。
- [Skill: Build with Claude API (reference guide)](./system-prompts_zh/skill-build-with-claude-api-reference-guide.md) (**468** tks) - 用于呈现具有快速任务导航的特定语言参考文档的模板。
- [Skill: Build with Claude API](./system-prompts_zh/skill-build-with-claude-api.md) (**5420** tks) - 与 Claude 一起构建 LLM 支持的应用程序的主要路由指南，包括语言检测、界面选择和架构概述。
- [Skill: Computer Use MCP](./system-prompts_zh/skill-computer-use-mcp.md) (**1206** tks) - 使用计算机使用的 MCP 工具的说明，包括工具选择层、应用程序访问层、链接安全和财务行为限制。
- [Skill: Create verifier skills](./system-prompts_zh/skill-create-verifier-skills.md) (**2625** tks) - 提示为验证代理创建验证者技能以自动验证代码更改。
- [Skill: Debugging](./system-prompts_zh/skill-debugging.md) (**412** tks) - 调试用户在 Claude Code 会话中遇到的问题的说明。
- [Skill: Simplify](./system-prompts_zh/skill-simplify.md) (**877** tks) - 简化代码的说明。
- [Skill: Update Claude Code Config](./system-prompts_zh/skill-update-claude-code-config.md) (**1255** tks) - 修改Claude Code配置文件(settings.json)的技能。
- [Skill: Verify CLI changes (example for Verify skill)](./system-prompts_zh/skill-verify-cli-changes-example-for-verify-skill.md) (**565** tks) - 用于验证 CLI 更改的示例工作流程，作为验证技能的一部分。
- [Skill: Verify server/API changes (example for Verify skill)](./system-prompts_zh/skill-verify-serverapi-changes-example-for-verify-skill.md) (**612** tks) - 用于验证服务器/API 更改的示例工作流程，作为验证技能的一部分。
- [Skill: Verify skill](./system-prompts_zh/skill-verify-skill.md) (**1779** tks) - 用于验证代码更改的固执己见的验证工作流程的技能。
- [Skill: update-config (7-step verification flow)](./system-prompts_zh/skill-update-config-7-step-verification-flow.md) (**1160** tks) - 一项技能，指导 Claude 通过 7 个步骤来构建和验证 Claude 代码的挂钩，确保它们在用户的特定项目环境中正常工作。
