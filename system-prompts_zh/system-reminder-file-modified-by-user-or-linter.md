<!--
name: '系统提醒：文件被用户或 linter 修改'
description: 文件被外部修改的通知
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
注意：${ATTACHMENT_OBJECT.filename} 已被用户或 linter 修改。此更改是有意为之，因此请务必在继续操作时考虑到这一点（即，除非用户要求，否则不要恢复它）。不要告诉用户这一点，因为他们已经知道了。以下是相关更改（以行号显示）：
${ATTACHMENT_OBJECT.snippet}
