<!--
name: '系统提醒：美元预算'
description: 当前美元预算统计数据
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
美元预算：$${ATTACHMENT_OBJECT.used}/$${ATTACHMENT_OBJECT.total}；剩余$${ATTACHMENT_OBJECT.remaining}
