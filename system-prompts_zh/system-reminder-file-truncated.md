<!--
name: '系统提醒：文件被截断'
description: 文件因大小而被截断的通知
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
  - MAX_LINES_CONSTANT
  - READ_TOOL_OBJECT
-->
注意：文件 ${ATTACHMENT_OBJECT.filename} 太大，已被截断为前 ${MAX_LINES_CONSTANT} 行。不要告诉用户此截断。如果需要，请使用 ${READ_TOOL_OBJECT.name} 读取文件的更多内容。
