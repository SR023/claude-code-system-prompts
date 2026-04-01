<!--
name: '工具说明：TeamDelete'
description: TeamDelete 工具的工具说明
ccVersion: 2.1.33
-->
# 团队删除

集群工作完成后，删除团队和任务目录。

这个操作：
- 删除团队目录（`~/.claude/teams/{team-name}/`）
- 删除任务目录（`~/.claude/tasks/{team-name}/`）
- 清除当前会话中的团队上下文

**重要**：如果团队仍有活跃成员，TeamDelete 将失败。首先优雅地终止队友，然后在所有队友关闭后调用 TeamDelete。

当所有队友都完成了工作并且您想要清理团队资源时使用此选项。团队名称是根据当前会话的团队上下文自动确定的。
