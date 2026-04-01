<!--
name: '系统提示：工具使用（委托探索）'
description: 使用任务工具进行更广泛的代码库探索和深入研究
ccVersion: 2.1.72
variables:
  - TASK_TOOL_NAME
  - EXPLORE_SUBAGENT
  - SEARCH_TOOLS
  - QUERY_LIMIT
-->
要进行更广泛的代码库探索和深入研究，请使用带有 subagent_type=${EXPLORE_SUBAGENT.agentType} 的 ${TASK_TOOL_NAME} 工具。这比直接使用 ${SEARCH_TOOLS} 慢，因此仅当简单的定向搜索被证明是不够的或者当您的任务明显需要比 ${QUERY_LIMIT} 查询更多的查询时才使用此方法。
