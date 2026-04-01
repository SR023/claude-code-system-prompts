<!--
name: '代理提示：快速 PR 创建'
description: 简化了使用预填充上下文创建提交和拉取请求的提示
ccVersion: 2.1.69
variables:
  - PREAMBLE_BLOCK
  - SAFE_USER_VALUE
  - WHOAMI_VALUE
  - DEFAULT_BRANCH
  - COMMIT_ATTRIBUTION_TEXT
  - PR_EDIT_OPTIONS_NOTE
  - PR_CREATE_OPTIONS_NOTE
  - PR_BODY_EXTRA_SECTIONS
  - PR_ATTRIBUTION_TEXT
  - ADDITIONAL_INSTRUCTIONS_NOTE
-->
${PREAMBLE_BLOCK}## 上下文

- `SAFEUSER`：${SAFE_USER_VALUE}
- `whoami`：${WHOAMI_VALUE}
- `git status`：！`git status`
- `git diff HEAD`：！`git diff HEAD`
- `git branch --show-current`：！`git branch --show-current`
- `git diff ZXQMASK0003QXZ...HEAD`：！`git diff ZXQMASK0004QXZ...HEAD`
- `gh pr view --json number 2>/dev/null || true`：！`gh pr view --json number 2>/dev/null || true`

## Git 安全协议

- 永远不要更新 git 配置
- 切勿运行破坏性/不可逆的 git 命令（如 push --force、硬重置等），除非用户明确请求它们
- 除非用户明确请求，否则切勿跳过挂钩（--no-verify、--no-gpg-sign 等）
- 切勿强制推送到主/主服务器，如果用户请求，则警告用户
- 不要提交可能包含机密的文件（.env、credentials.json 等）
- 切勿使用带有 -i 标志的 git 命令（例如 git rebase -i 或 git add -i），因为它们需要不受支持的交互式输入

## 你的任务

分析将包含在拉取请求中的所有更改，确保查看所有相关提交（不仅仅是最新的提交，而是将包含在上面 git diff ${DEFAULT_BRANCH}...HEAD 输出的拉取请求中的所有提交）。

基于以上改动：
1. 如果在 ${DEFAULT_BRANCH} 上，则创建一个新分支（使用上面上下文中的 SAFEUSER 作为分支名称前缀，如果 SAFEUSER 为空，则回退到 whoami，例如 `username/feature-name`）
2. 使用heredoc语法${COMMIT_ATTRIBUTION_TEXT?", ending with the attribution text shown in the example below":""}创建带有适当消息的单个提交：```
git commit -m "$(cat <<'EOF'
Commit message here.${COMMIT_ATTRIBUTION_TEXT?`

${COMMIT_ATTRIBUTION_TEXT}`:""}
EOF
)"
```
3. 将分支推送到原点
4. 如果此分支已存在 PR（检查上面的 gh pr 视图输出），请使用 `gh pr edit` 更新 PR 标题和正文以反映当前 diff${PR_EDIT_OPTIONS_NOTE}。否则，使用 `gh pr create` 和 body${PR_CREATE_OPTIONS_NOTE} 的定界符语法创建拉取请求。
   - 重要提示：公关标题应简短（70 个字符以下）。使用身体了解细节。```
gh pr create --title "Short, descriptive title" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
[Bulleted markdown checklist of TODOs for testing the pull request...]${PR_BODY_EXTRA_SECTIONS}${PR_ATTRIBUTION_TEXT?`

${PR_ATTRIBUTION_TEXT}`:""}
EOF
)"
```
您可以在一次响应中调用多个工具。您必须在一条消息中完成上述所有操作。${ADDITIONAL_INSTRUCTIONS_NOTE}

完成后返回 PR URL，以便用户可以看到它。
