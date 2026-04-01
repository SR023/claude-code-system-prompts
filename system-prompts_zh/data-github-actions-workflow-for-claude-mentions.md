<!--
name: '数据：@claude 提及的 GitHub Actions 工作流程'
description: 用于通过 @claude 提及触发 Claude 代码的 GitHub Actions 工作流程模板
ccVersion: 2.0.58
-->
姓名：克劳德·代码

于：
  问题评论：
    类型：[已创建]
  pull_request_review_comment：
    类型：[已创建]
  问题：
    类型：[已打开、已分配]
  拉请求审查：
    类型：[已提交]

职位：
  克劳德：
    如果： |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude')) ||
      (github.event_name == '问题' && (包含(github.event.issue.body, '@claude') || contains(github.event.issue.title, '@claude')))
    运行：ubuntu-latest
    权限：
      内容： 阅读
      拉取请求：读取
      问题：阅读
      id 令牌：写入
      actions: read # Claude 需要读取 PR 上的 CI 结果
    步骤：
      - 名称：签出存储库
        使用：actions/checkout@v4
        与：
          获取深度：1

      - 名称：运行克劳德代码
        id: 克劳德
        使用：anthropics/claude-code-action@v1
        与：
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

          # 这是一个可选设置，允许 Claude 读取 PR 上的 CI 结果
          附加权限： |
            行动：阅读

          # 可选：向 Claude 提供自定义提示。如果未指定，Claude 将执行标记它的注释中指定的指令。
          # 提示：“更新拉取请求描述以包含更改摘要。”

          # 可选：添加 claude_args 来自定义行为和配置
          #参见https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md
          # 或 https://code.claude.com/docs/en/cli-reference 以获得可用选项
          # claude_args: '--允许的工具 Bash(gh pr:*)'
