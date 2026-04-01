<!--
name: '技能：调试'
description: 有关调试用户在 Claude Code 会话中遇到的问题的说明
ccVersion: 2.1.71
variables:
  - DEBUG_LOGGING_WAS_ALREADY_ACTIVE
  - DEBUG_LOG_PATH
  - DEBUG_LOG_SUMMARY
  - ISSUE_DESCRIPTION
  - GET_SETTINGS_FILE_PATH_FN
  - LOG_LINE_COUNT
  - CLAUDE_CODE_GUIDE_SUBAGENT_NAME
-->
# 调试技巧

帮助用户调试他们在当前 Claude Code 会话中遇到的问题。
${DEBUG_LOGGING_WAS_ALREADY_ACTIVE?"":`
## Debug Logging Just Enabled

Debug logging was OFF for this session until now. Nothing prior to this /debug invocation was captured.

Tell the user that debug logging is now active at `${DEBUG_LOG_PATH}`, ask them to reproduce the issue, then re-read the log. If they can't reproduce, they can also restart with `claude --debug` 从启动时捕获日志。
`}
## 会话调试日志

当前会话的调试日志位于：`ZXQMASK0001QXZ`

${DEBUG_LOG_SUMMARY}

对于其他上下文，请 grep 查找整个文件中的 [ERROR] 和 [WARN] 行。

## 问题描述

${ISSUE_DESCRIPTION||"The user did not describe a specific issue. Read the debug log and summarize any errors, warnings, or notable issues."}

## 设置

请记住，设置位于：
* 用户 - ${GET_SETTINGS_FILE_PATH_FN("userSettings")}
* 项目 - ${GET_SETTINGS_FILE_PATH_FN("projectSettings")}
*本地-${GET_SETTINGS_FILE_PATH_FN("localSettings")}

## 说明

1.查看用户的问题描述
2. 最后 ${LOG_LINE_COUNT} 行显示调试文件格式。在文件中查找 [ERROR] 和 [WARN] 条目、堆栈跟踪和故障模式
3.考虑启动${CLAUDE_CODE_GUIDE_SUBAGENT_NAME}子代理来了解相关的Claude Code功能
4. 用通俗易懂的语言解释你的发现
5.提出具体的修复或后续步骤建议
