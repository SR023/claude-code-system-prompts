<!--
name: '系统提醒：挂钩附加上下文'
description: 来自钩子的附加上下文
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} 挂钩附加上下文：${ATTACHMENT_OBJECT.content.join(`
`)}
