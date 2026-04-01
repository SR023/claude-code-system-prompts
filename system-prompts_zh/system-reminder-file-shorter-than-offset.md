<!--
name: '系统提醒：文件短于偏移量'
description: 文件读取偏移量超过文件长度时发出警告
ccVersion: 2.1.18
variables:
  - RESULT_OBJECT
-->
<system-reminder>警告：文件存在，但短于提供的偏移量 (${RESULT_OBJECT.file.startLine})。该文件有 ${RESULT_OBJECT.file.totalLines} 行。</system-reminder>
