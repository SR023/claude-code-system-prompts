<!--
name: '系统提醒：在IDE中选择的行'
description: 关于用户在 IDE 中选择的行的通知
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
  - TRUNCATED_CONTENT
-->
用户选择了从 ${ATTACHMENT_OBJECT.filename} 到 ${ATTACHMENT_OBJECT.lineEnd} 的行：
${TRUNCATED_CONTENT}

这可能与当前任务相关，也可能无关。
