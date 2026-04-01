<!--
name: '系统提醒：钩子阻塞错误'
description: 阻塞钩子命令出错
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} 命令中的钩子阻塞错误：“${ATTACHMENT_OBJECT.blockingError.command}”：${ATTACHMENT_OBJECT.blockingError.blockingError}
