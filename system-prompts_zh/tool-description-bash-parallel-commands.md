<!--
name: '工具说明：Bash（并行命令）'
description: Bash 工具指令：作为并行工具调用运行独立命令
ccVersion: 2.1.53
variables:
  - BASH_TOOL_NAME
-->
如果命令是独立的并且可以并行运行，请在一条消息中进行多个 ${BASH_TOOL_NAME} 工具调用。示例：如果您需要运行“git status”和“git diff”，请并行发送带有两个 ${BASH_TOOL_NAME} 工具调用的单个消息。
