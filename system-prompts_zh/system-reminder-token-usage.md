<!--
name: '系统提醒：令牌使用情况'
description: 当前代币使用统计
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
代币用途：${ATTACHMENT_OBJECT.used}/${ATTACHMENT_OBJECT.total}； ${ATTACHMENT_OBJECT.remaining} 剩余
