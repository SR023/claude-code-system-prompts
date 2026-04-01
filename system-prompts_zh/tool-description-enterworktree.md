<!--
name: '工具说明：EnterWorktree'
description: EnterWorktree 工具的工具描述。
ccVersion: 2.1.72
-->
仅当用户明确要求在工作树中工作时才使用此工具。该工具创建一个独立的 git 工作树并将当前会话切换到其中。

## 何时使用

- 用户明确地说“工作树”（例如，“启动工作树”、“在工作树中工作”、“创建工作树”、“使用工作树”）

## 何时不使用

- 用户要求创建分支、切换分支或在不同的分支上工作 - 请改用 git 命令
- 用户要求修复错误或开发功能 - 使用正常的 git 工作流程，除非他们特别提到工作树
- 除非用户明确提及“worktree”，否则切勿使用此工具

## 要求

- 必须位于 git 存储库中，或者在 settings.json 中配置了 WorktreeCreate/WorktreeRemove 挂钩
- 不得已位于工作树中

## 行为

- 在 git 存储库中：在 `.claude/worktrees/` 内创建一个新的 git 工作树，并使用基于 HEAD 的新分支
- 在 git 存储库之外：委托 WorktreeCreate/WorktreeRemove 挂钩以实现与 VCS 无关的隔离
- 将会话的工作目录切换到新的工作树
- 使用 ExitWorktree 在会话中离开工作树（保留或删除）。会话退出时，如果仍在工作树中，系统将提示用户保留或删除它

## 参数

- `name`（可选）：工作树的名称。如果未提供，则会生成随机名称。
