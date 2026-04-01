<!--
name: '工具说明：发送消息工具'
description: SendMessageTool 的代理团队版本。
ccVersion: 2.1.83
-->
# 发送消息

向另一个代理发送消息。```json
{"to": "researcher", "summary": "assign task 1", "message": "start on task #1"}
```
| `to` | |
|---|---|
| `"researcher"` |队友名字|
| `"*"` |向所有队友广播——昂贵（与团队规模成线性），仅在每个人真正需要时使用 |${""}

您的纯文本输出对其他代理不可见 - 要进行通信，您必须调用此工具。队友的消息自动传递；你不检查收件箱。按姓名而非 UUID 引用队友。转发时，不要引用原文——它已经呈现给用户了。${""}

## 协议响应（旧版）

如果您收到带有 `type: "shutdown_request"` 或 `type: "plan_approval_request"` 的 JSON 消息，请使用匹配的 `_response` 类型进行响应 — 回显 `request_id`，设置 `approve` true/false：```json
{"to": "team-lead", "message": {"type": "shutdown_response", "request_id": "...", "approve": true}}
{"to": "researcher", "message": {"type": "plan_approval_response", "request_id": "...", "approve": false, "feedback": "add error handling"}}
```
批准关闭将终止您的进程。拒绝计划将队友送回修改。除非有要求，否则不要发起 `shutdown_request`。不要发送结构化 JSON 状态消息 - 使用 TaskUpdate。
