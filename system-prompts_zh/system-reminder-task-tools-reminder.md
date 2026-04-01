<!--
name: '系统提醒：任务工具提醒'
description: 使用任务跟踪工具的提醒
ccVersion: 2.1.18
variables:
  - TASK_CREATE_TOOL_NAME
  - TASK_UPDATE_TOOL_NAME
-->
最近没有使用过任务工具。如果您正在处理的任务将受益于跟踪进度，请考虑使用 ${TASK_CREATE_TOOL_NAME} 添加新任务，并使用 ${TASK_UPDATE_TOOL_NAME} 更新任务状态（开始时设置为 in_progress，完成时设置为已完成）。如果任务列表已经过时，还可以考虑清理它。仅当与当前工作相关时才使用它们。这只是一个温和的提醒 - 如果不适用请忽略。确保您永远不会向用户提及此提醒
