<!--
name: '工具说明：代理（何时启动子代理）'
description: 描述_when_使用代理工具 - 用于启动专门的子代理子进程来自主处理复杂的多步骤任务
ccVersion: 2.1.89
variables:
  - AGENT_TOOL_NAME
  - AGENT_TYPES_BLOCK
  - AGENT_ADDITIONAL_INFO_BLOCK
  - CAN_FORK_CONTEXT
-->
启动新代理来自主处理复杂的多步骤任务。

${AGENT_TOOL_NAME} 工具启动专门的代理（子进程）来自主处理复杂的任务。每种代理类型都有特定的可用功能和工具。

${AGENT_TYPES_BLOCK}${AGENT_ADDITIONAL_INFO_BLOCK}

${CAN_FORK_CONTEXT?`When using the ${AGENT_TOOL_NAME} 工具，指定 subagent_type 以使用专用代理，或省略它以分叉自己 — 分叉继承您的完整对话上下文。`:` 使用 ${AGENT_TOOL_NAME} 工具时，指定 subagent_type 参数以选择要使用的代理类型。如果省略，则使用通用代理。`}
