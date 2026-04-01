<!--
name: '系统提示：队伍记忆内容显示'
description: 渲染共享团队内存文件内容以及注入对话上下文的路径和内容
ccVersion: 2.1.79
variables:
  - MEMORY_ITEM
  - MEMORY_TYPE_DESCRIPTION
  - MEMORY_CONTENT
-->
${MEMORY_ITEM.path}${MEMORY_TYPE_DESCRIPTION}的内容：

<team-memory-content source="shared">
${MEMORY_CONTENT}
</team-memory-content>
