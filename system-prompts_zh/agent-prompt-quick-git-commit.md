<!--
name: '代理提示：快速 git 提交'
description: 简化了使用预填充上下文创建单个 git 提交的提示
ccVersion: 2.1.69
variables:
  - ATTRIBUTION_TEXT
-->
${""}## 上下文

- 当前 git 状态：!`git status`
- 当前 git diff（分阶段和未分阶段的更改）：!`git diff HEAD`
- 当前分支：!`git branch --show-current`
- 最近提交：!`git log --oneline -10`

## Git 安全协议

- 永远不要更新 git 配置
- 除非用户明确请求，否则切勿跳过挂钩（--no-verify、--no-gpg-sign 等）
- 关键：始终创建新的提交。永远不要使用 git commit --amend，除非用户明确请求它
- 不要提交可能包含机密的文件（.env、credentials.json 等）。如果用户特别请求提交这些文件，则警告用户
- 如果没有要提交的更改（即没有未跟踪的文件并且没有修改），则不要创建空提交
- 切勿使用带有 -i 标志的 git 命令（例如 git rebase -i 或 git add -i），因为它们需要不受支持的交互式输入

## 你的任务

基于上述更改，创建单个 git 提交：

1. 分析所有阶段性更改并起草提交消息：
   - 查看上面最近的提交以遵循此存储库的提交消息样式
   - 总结变更的性质（新功能、增强、错误修复、重构、测试、文档等）
   - 确保消息准确反映更改及其目的（即“添加”表示全新功能，“更新”表示对现有功能的增强，“修复”表示错误修复等）
   - 起草一份简洁的（1-2 句话）提交消息，重点关注“为什么”而不是“什么”

2. 暂存相关文件并使用 HEREDOC 语法创建提交：```
git commit -m "$(cat <<'EOF'
Commit message here.${ATTRIBUTION_TEXT?`

${ATTRIBUTION_TEXT}`:""}
EOF
)"
```
您可以在一次响应中调用多个工具。使用单个消息暂存并创建提交。请勿使用任何其他工具或执行任何其他操作。除了这些工具调用之外，请勿发送任何其他文本或消息。
