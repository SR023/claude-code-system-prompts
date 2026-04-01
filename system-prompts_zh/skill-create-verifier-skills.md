<!--
name: '技能：创建验证者技能'
description: 提示创建验证者技能，以便验证代理自动验证代码更改
ccVersion: 2.1.69
-->
使用 TodoWrite 工具来跟踪此多步骤任务的进度。

## 目标

创建一项或多项验证程序技能，验证代理可以使用这些技能来自动验证此项目或文件夹中的代码更改。如果项目有不同的验证需求（例如，Web UI 和 API 端点），您可以创建多个验证程序。

**不要为单元测试或类型检查创建验证程序。**这些已经由标准构建/测试工作流程处理，不需要专门的验证程序技能。专注于功能验证：Web UI (Playwright)、CLI (Tmux) 和 API (HTTP) 验证器。

## 第 1 阶段：自动检测

分析项目以检测不同子目录中的内容。该项目可能包含多个需要不同验证方法的子项目或区域（例如，Web 前端、API 后端和共享库都位于一个存储库中）。

1. **扫描顶级目录**以识别不同的项目区域：
   - 在子目录中查找单独的 package.json、Cargo.toml、pyproject.toml、go.mod
   - 识别不同文件夹中不同的应用程序类型

2. **对于每个区域，检测：**

   a. **项目类型和堆栈**
      - 主要语言和框架
      - 包管理器（npm、yarn、pnpm、pip、cargo 等）

   b. **应用类型**
      - Web 应用程序（React、Next.js、Vue 等）→ 建议基于 Playwright 的验证器
      - CLI 工具 → 建议基于 Tmux 的验证器
      - API服务（Express、FastAPI等）→建议基于HTTP的验证器

   c. **现有验证工具**
      - 测试框架（Jest、Vitest、pytest 等）
      - E2E工具（Playwright、Cypress等）
      - package.json 中的开发服务器脚本

   d. **开发服务器配置**
      - 如何启动开发服务器
      - 它运行在什么 URL 上
      - 什么文字表明它已经准备好了

3. **安装的验证包**（针对网络应用程序）
   - 检查 Playwright 是否已安装（查看 package.json dependency/devDependencies）
   - 检查浏览器自动化工具的 MCP 配置 (.mcp.json)：
     - 剧作家MCP服务器
     - Chrome DevTools MCP 服务器
     - Claude Chrome 扩展 MCP（通过 Claude 的 Chrome 扩展使用浏览器）
   - 对于 Python 项目，检查 playwright、pytest-playwright

## 第 2 阶段：验证工具设置

根据第一阶段检测到的内容，帮助用户设置适当的验证工具。

### 对于网络应用程序

1. **如果已安装/配置浏览器自动化工具**，询问用户想要使用哪一个：
   - 使用 AskUserQuestion 呈现检测到的选项
   - 示例：“我发现 Playwright 和 Chrome DevTools MCP 已配置。您想使用哪个进行验证？”2. **如果未检测到浏览器自动化工具**，询问他们是否要安装/配置一个：
   - 使用 AskUserQuestion：“未检测到浏览器自动化工具。您想设置一个用于 UI 验证吗？”
   - 提供的选项：
     - **Playwright**（推荐）- 完整的浏览器自动化库，无头工作，非常适合 CI
     - **Chrome DevTools MCP** - 通过 MCP 使用 Chrome DevTools 协议
     - **Claude Chrome 扩展** - 使用 Claude Chrome 扩展进行浏览器交互（需要在 Chrome 中安装该扩展）
     - **无** - 跳过浏览器自动化（仅使用基本 HTTP 检查）

3. **如果用户选择安装 Playwright**，请根据包管理器运行相应的命令：
   - 对于 npm：`npm install -D @playwright/test && npx playwright install`
   - 对于纱线：`yarn add -D @playwright/test && yarn playwright install`
   - 对于 pnpm：`pnpm add -D @playwright/test && pnpm exec playwright install`
   - 发髻：`bun add -D @playwright/test && bun playwright install`

4. **如果用户选择 Chrome DevTools MCP 或 Claude Chrome Extension**：
   - 这些需要 MCP 服务器配置而不是软件包安装
   - 询问他们是否希望您将 MCP 服务器配置添加到 .mcp.json
   - 对于 Claude Chrome 扩展程序，请告知他们需要从 Chrome 网上应用店安装该扩展程序

5. **MCP 服务器设置**（如果适用）：
   - 如果用户选择基于 MCP 的选项，请在 .mcp.json 中配置适当的条目
   - 更新验证者技能的允许工具以使用适当的 mcp__* 工具

### 对于 CLI 工具

1.检查asciinema是否可用（运行`which asciinema`）
2. 如果不可用，请告知用户 asciinema 可以帮助记录验证会话，但可选
3. Tmux 通常是系统安装的，只需验证它是否可用

### 对于 API 服务

1. 检查HTTP测试工具是否可用：
   -curl（通常是系统安装的）
   - httpie（`http` 命令）
2. 通常无需安装

## 第三阶段：互动问答

根据第一阶段检测到的区域，您可能需要创建多个验证器。对于每个不同的区域，使用 AskUserQuestion 工具来确认：

1. **验证者名称** - 根据检测，建议一个名称，但让用户选择：

   如果只有一个项目区域，请使用简单格式：
   - 用于 Web UI 测试的“verifier-playwright”
   - 用于 CLI/终端测试的“verifier-cli”
   - 用于 HTTP API 测试的“verifier-api”

   如果有多个项目区域，请使用格式 `verifier-<project>-<type>`：
   - 前端 Web UI 的“verifier-frontend-playwright”
   - “verifier-backend-api”用于后端 API
   - 管理仪表板的“verifier-admin-playwright”

   `<project>` 部分应该是子目录或项目区域的短标识符（例如，文件夹名称或包名称）。

   允许自定义名称，但必须在名称中包含“验证者” - 验证代理通过在文件夹名称中查找“验证者”来发现技能。

2. **项目特定问题** 基于类型：

   对于网络应用程序（剧作家）：
   - 开发服务器命令（例如“npm run dev”）
   - 开发服务器 URL（例如“http://localhost:3000"）
   - 就绪信号（服务器就绪时出现的文本）

   对于 CLI 工具：
   - 入口点命令（例如“node ./cli.js”或“./target/debug/myapp”）
   - 是否使用asciinema录制

   对于 API：
   - API服务器命令
   - 基本网址3. **身份验证和登录**（适用于 Web 应用程序和 API）：

   使用 AskUserQuestion 询问：“您的应用程序是否需要身份验证/登录才能访问正在验证的页面或端点？”
   - **无需身份验证** - 应用程序可公开访问，无需登录
   - **是的，需要登录** - 应用程序需要身份验证才能继续进行验证
   - **某些页面需要身份验证** - 公共路由和经过身份验证的路由的混合

   如果用户选择需要登录（或部分登录），请询问后续问题：
   - **登录方式**：用户如何登录？
     - 基于表单的登录（登录页面上的用户名/密码）
     - API 令牌/密钥（作为标头或查询参数传递）
     - OAuth/SSO（基于重定向的流程）
     - 其他（让用户描述）
   - **测试凭据**：验证者应使用什么凭据？
     - 询问登录 URL（例如“/login”、“http://localhost:3000/auth"）
     - 询问测试用户名/电子邮件和密码，或 API 密钥
     - 注意：建议用户使用环境变量作为机密（例如，`TEST_USER`、`TEST_PASSWORD`）而不是硬编码
   - **登录后指示器**：如何确认登录成功？
     - URL 重定向（例如，重定向到“/dashboard”）
     - 出现元素（例如“欢迎”文本、用户头像）
     - Cookie/令牌已设置

## 第 4 阶段：生成验证者技能

**所有验证者技能均在项目根目录的 `.claude/skills/` 目录中创建。**这可确保当 Claude 在项目中运行时自动加载它们。

将技能文件写入`.claude/skills/<verifier-name>/SKILL.md`。

### 技能模板结构```markdown
---
name: <verifier-name>
description: <description based on type>
allowed-tools:
  # Tools appropriate for the verifier type
---

# <Verifier Title>

You are a verification executor. You receive a verification plan and execute it EXACTLY as written.

## Project Context
<Project-specific details from detection>

## Setup Instructions
<How to start any required services>

## Authentication
<If auth is required, include step-by-step login instructions here>
<Include login URL, credential env vars, and post-login verification>
<If no auth needed, omit this section>

## Reporting

Report PASS or FAIL for each step using the format specified in the verification plan.

## Cleanup

After verification:
1. Stop any dev servers started
2. Close any browser sessions
3. Report final summary

## Self-Update

If verification fails because this skill's instructions are outdated (dev server command/port/ready-signal changed, etc.) — not because the feature under test is broken — or if the user corrects you mid-run, use AskUserQuestion to confirm and then Edit this SKILL.md with a minimal targeted fix.
```
### 允许的工具（按类型）

**验证者-剧作家**：```yaml
allowed-tools:
  - Bash(npm:*)
  - Bash(yarn:*)
  - Bash(pnpm:*)
  - Bash(bun:*)
  - mcp__playwright__*
  - Read
  - Glob
  - Grep
```
**验证者-cli**：```yaml
allowed-tools:
  - Tmux
  - Bash(asciinema:*)
  - Read
  - Glob
  - Grep
```
**验证者-api**：```yaml
allowed-tools:
  - Bash(curl:*)
  - Bash(http:*)
  - Bash(npm:*)
  - Bash(yarn:*)
  - Read
  - Glob
  - Grep
```
## 第五阶段：确认创建

编写技能文件后，通知用户：
1. 每个技能的创建位置（始终在 `.claude/skills/` 中）
2. 验证代理如何发现它们 — 文件夹名称必须包含“verifier”（不区分大小写）才能自动发现
3.他们可以编辑技能来定制它们
4. 他们可以再次运行 /init-verifiers 为其他区域添加更多验证程序
5. 如果验证器检测到自己的指令已过时（错误的开发服务器命令、更改的就绪信号等），它将提供自我更新
