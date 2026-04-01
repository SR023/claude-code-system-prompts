<!--
name: '系统提示：Hooks配置'
description: 系统提示挂钩配置。  用于上述 Claude Code 配置技能。
ccVersion: 2.1.77
-->
## 钩子配置

Hooks 在 Claude Code 生命周期的特定点运行命令。

### 钩子结构```json
{
  "hooks": {
    "EVENT_NAME": [
      {
        "matcher": "ToolName|OtherTool",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60,
            "statusMessage": "Running..."
          }
        ]
      }
    ]
  }
}
```
### 挂钩事件

|活动 |匹配器|目的|
|--------|---------|---------|
|许可请求 |工具名称|在权限提示之前运行 |
|预工具使用 |工具名称|在工具之前运行，可以阻止 |
|发布工具使用 |工具名称|成功后运行工具 |
|工具使用失败后 |工具名称|工具失败后运行 |
|通知 |通知类型 |根据通知运行 |
|停止| - |克劳德停止时运行（包括清除、恢复、紧凑）|
|预紧凑 | “手动”/“自动”|压实前|
|后紧凑 | “手动”/“自动”|压缩后（收到摘要）|
|用户提示提交 | - |当用户提交|
|会话开始 | - |会话开始时 |

**常用工具匹配器：** `Bash`、`Write`、`Edit`、`Read`、`Glob`、`Grep`

### 钩子类型

**1.命令挂钩** - 运行 shell 命令：```json
{ "type": "command", "command": "prettier --write $FILE", "timeout": 30 }
```
**2. Prompt Hook** - 使用 LLM 评估条件：```json
{ "type": "prompt", "prompt": "Is this safe? $ARGUMENTS" }
```
仅适用于工具事件：PreToolUse、PostToolUse、PermissionRequest。

**3. Agent Hook** - 使用工具运行代理：```json
{ "type": "agent", "prompt": "Verify tests pass: $ARGUMENTS" }
```
仅适用于工具事件：PreToolUse、PostToolUse、PermissionRequest。

### 挂钩输入（标准输入 JSON）```json
{
  "session_id": "abc123",
  "tool_name": "Write",
  "tool_input": { "file_path": "/path/to/file.txt", "content": "..." },
  "tool_response": { "success": true }  // PostToolUse only
}
```
### 挂钩 JSON 输出

Hooks 可以返回 JSON 来控制行为：```json
{
  "systemMessage": "Warning shown to user in UI",
  "continue": false,
  "stopReason": "Message shown when blocking",
  "suppressOutput": false,
  "decision": "block",
  "reason": "Explanation for decision",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Context injected back to model"
  }
}
```
**字段：**
- `systemMessage` - 向用户显示消息（所有挂钩）
- `continue` - 设置为 `false` 以阻止/停止（默认值：true）
- `stopReason` - `continue` 为 false 时显示的消息
- `suppressOutput` - 从记录中隐藏标准输出（默认值：false）
- `decision` - PostToolUse/Stop/UserPromptSubmit 挂钩的“阻止”（已弃用 PreToolUse，请改用 hookSpecificOutput.permissionDecision）
- `reason` - 决定说明
- `hookSpecificOutput` - 事件特定输出（必须包括 `hookEventName`）：
  - `additionalContext` - 文本注入模型上下文
  - `permissionDecision` - “允许”、“拒绝”或“询问”（仅限 PreToolUse）
  - `permissionDecisionReason` - 权限决定的原因（仅限 PreToolUse）
  - `updatedInput` - 修改工具输入（仅限 PreToolUse）

### 常见模式

**写入后自动格式化：**```json
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
**记录所有 bash 命令：**```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.command' >> ~/.claude/bash-log.txt"
      }]
    }]
  }
}
```
**停止向用户显示消息的挂钩：**

命令必须输出带有 `systemMessage` 字段的 JSON：```bash
# Example command that outputs: {"systemMessage": "Session complete!"}
echo '{"systemMessage": "Session complete!"}'
```
**代码更改后运行测试：**```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path // .tool_response.filePath' | grep -E '\\.(ts|js)$' && npm test || true"
      }]
    }]
  }
}
```
