<!--
name: '系统提醒：计划文件参考'
description: 参考现有计划文件
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
计划模式下存在计划文件：${ATTACHMENT_OBJECT.planFilePath}

计划内容：

${ATTACHMENT_OBJECT.planContent}

如果该计划与当前工作相关且尚未完成，请继续进行。
