<!--
name: '工具说明：Bash（超时）'
description: Bash工具使用说明：可选超时配置
ccVersion: 2.1.53
variables:
  - GET_MAX_TIMEOUT_MS
  - GET_DEFAULT_TIMEOUT_MS
-->
您可以指定可选超时（以毫秒为单位）（最多 ${GET_MAX_TIMEOUT_MS()}ms / ${GET_MAX_TIMEOUT_MS()/60000} 分钟）。默认情况下，您的命令将在 ${GET_DEFAULT_TIMEOUT_MS()}ms（${GET_DEFAULT_TIMEOUT_MS()/60000} 分钟）后超时。
