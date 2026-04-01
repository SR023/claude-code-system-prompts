<!--
name: '技能：更新Claude Code配置'
description: 修改Claude Code配置文件（settings.json）的技巧。
ccVersion: 2.1.77
variables:
  - SETTINGS_FILE_LOCATION_PROMPT
  - HOOKS_CONFIGURATION_PROMPT
  - CONSTRUCTING_HOOK_PROMPT
-->
# 更新配置技能

通过更新 settings.json 文件修改 Claude Code 配置。

## 当需要钩子时（不是内存）

如果用户希望响应事件而自动发生某些事情，他们需要在 settings.json 中配置 **hook**。内存/首选项无法触发自动操作。

**这些需要钩子：**
- “压缩之前，问我要保留什么” → PreCompact 钩子
- “写入文件后，运行 prettier” → PostToolUse 与 Write|Edit 匹配器挂钩
- “当我运行 bash 命令时，记录它们” → PreToolUse 与 Bash 匹配器挂钩
- “代码更改后始终运行测试” → PostToolUse 挂钩

**挂钩事件：** PreToolUse、PostToolUse、PreCompact、PostCompact、Stop、Notification、SessionStart

## 关键：先读后写

**在进行更改之前始终读取现有设置文件。** 将新设置与现有设置合并 - 切勿替换整个文件。

## 关键：使用 AskUserQuestion 来消除歧义

当用户的请求不明确时，使用AskUserQuestion来澄清：
- 要修改哪个设置文件（用户/项目/本地）
- 是否添加到现有数组或替换它们
- 存在多个选项时的特定值

## 决定：配置工具与直接编辑

**使用配置工具**进行这些简单的设置：
- `theme`、`editorMode`、`verbose`、`model`
- `language`、`alwaysThinkingEnabled`
- `permissions.defaultMode`

**直接编辑settings.json**：
- 挂钩（PreToolUse、PostToolUse 等）
- 复杂的权限规则（允许/拒绝数组）
- 环境变量
- MCP服务器配置
- 插件配置

## 工作流程

1. **澄清意图** - 询问请求是否不明确
2. **读取现有文件** - 在目标设置文件上使用读取工具
3. **仔细合并** - 保留现有设置，尤其是数组
4. **编辑文件** - 使用编辑工具（如果文件不存在，请用户先创建它）
5. **确认** - 告诉用户更改了什么

## 合并数组（重要！）

添加到权限数组或挂钩数组时，**与现有**合并，不要替换：

**错误**（替换现有权限）：```json
{ "permissions": { "allow": ["Bash(npm:*)"] } }
```
**右**（保留现有+添加新内容）：```json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",      // existing
      "Edit(.claude)",    // existing
      "Bash(npm:*)"       // new
    ]
  }
}
```
${SETTINGS_FILE_LOCATION_PROMPT}

${HOOKS_CONFIGURATION_PROMPT}

${CONSTRUCTING_HOOK_PROMPT}

## 工作流程示例

### 添加一个钩子

用户：“在 Claude 编写代码后格式化我的代码”

1. **澄清**：哪种格式化程序？ （更漂亮、gofmt 等）
2. **阅读**：`.claude/settings.json`（如果缺少，则创建）
3. **合并**：添加到现有的钩子中，不替换
4. **结果**：```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_response.filePath // .tool_input.file_path' | { read -r f; prettier --write \"$f\"; } 2>/dev/null || true"
      }]
    }]
  }
}
```
### 添加权限

用户：“允许 npm 命令而不提示”

1. **读取**：现有权限
2. **合并**：添加`Bash(npm:*)`以允许数组
3. **结果**：与现有允许相结合

### 环境变量

用户：“设置 DEBUG=true”

1. **决定**：用户设置（全局）还是项目设置？
2. **读取**：目标文件
3. **合并**：添加到env对象```json
{ "env": { "DEBUG": "true" } }
```
## 要避免的常见错误

1. **替换而不是合并** - 始终保留现有设置
2. **文件错误** - 询问用户范围是否不清楚
3. **无效的 JSON** - 更改后验证语法
4. **忘记先读** - 始终先读后写

## 钩子故障排除

如果挂钩没有运行：
1. **检查设置文件** - 读取 ~/.claude/settings.json 或 .claude/settings.json
2. **验证 JSON 语法** - 无效的 JSON 静默失败
3. **检查匹配器** - 与工具名称匹配吗？ （例如，“重击”、“写入”、“编辑”）
4. **检查钩子类型** - 是“命令”、“提示”还是“代理”？
5. **测试命令** - 手动运行hook命令看是否有效
6. **使用--debug** - 运行`claude --debug`查看钩子执行日志
