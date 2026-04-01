<!--
name: '工具说明：Bash（git——永不跳过钩子）'
description: Bash 工具 git 指令：除非用户请求，否则永远不要跳过挂钩或绕过签名
ccVersion: 2.1.53
-->
除非用户明确要求，否则切勿跳过挂钩（--no-verify）或绕过签名（--no-gpg-sign，-c commit.gpgsign = false）。如果挂钩失败，请调查并解决根本问题。
