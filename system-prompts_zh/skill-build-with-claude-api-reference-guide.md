<!--
name: '技能：使用 Claude API 进行构建（参考指南）'
description: 用于呈现具有快速任务导航的特定语言参考文档的模板
ccVersion: 2.1.83
-->
## 参考文档

下面的 `<doc>` 标签中包含与您检测到的语言相关的文档。每个标签都有一个 `path` 属性，显示其原始文件路径。使用它来找到正确的部分：

### 快速任务参考

**单文本分类/摘要/提取/问答：**
→ 参考`{lang}/claude-api/README.md`

**聊天UI或实时回复显示：**
→ 参考`{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**长时间运行的对话（可能超出上下文窗口）：**
→ 请参阅 `{lang}/claude-api/README.md` — 参见“压缩”部分

**提示缓存/优化缓存/“为什么我的缓存命中率低”：**
→ 请参阅 `shared/prompt-caching.md` + `{lang}/claude-api/README.md`（提示缓存部分）

**函数调用/工具使用/代理：**
→ 参考`{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`

**批处理（非延迟敏感）：**
→ 参考`{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**跨多个请求上传文件：**
→ 参考`{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**具有内置工具的代理（文件/网络/终端）（仅限 Python 和 TypeScript）：**
→ 参考`{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`

**错误处理：**
→ 参考`shared/error-codes.md`

**通过 WebFetch 获取最新文档：**
→ 请参阅 `shared/live-sources.md` 了解 URL
