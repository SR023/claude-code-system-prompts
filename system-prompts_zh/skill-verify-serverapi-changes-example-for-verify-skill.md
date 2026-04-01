<!--
name: '技能：验证服务器/API 更改（验证技能示例）'
description: 作为验证技能的一部分，用于验证服务器/API 更改的示例工作流程。
ccVersion: 2.1.83
-->
# 验证服务器/API 更改

手柄为 `curl`（或同等手柄）。证据就是回应。

## 模式

1. 启动服务器（后台，进行准备情况轮询 - 见下文）
2. `curl` diff 触及的路线，其输入命中已更改的分支
3. 捕获完整响应（状态+标题+正文）
4. 与预期比较

## 生命周期

如果有运行技能，它就能处理这个问题。如果不：```bash
<start-command> &> /tmp/server.log &
SERVER_PID=$!
for i in {1..30}; do curl -sf localhost:PORT/health >/dev/null && break; sleep 1; done
# ... your curls ...
kill $SERVER_PID
```
没有就绪终点？轮询您要测试的路线，直到
停止返回连接拒绝，然后添加节拍。

## 工作示例

**差异：**将 `Retry-After` 标头添加到 `rateLimit.ts` 中的 429 响应中。
**声明（公关机构）：**“客户现在可以正确退出。”

**推论：** 达到速率限制现在应该返回 `Retry-After: <n>`
在响应标头中。以前没有。

**计划：**
1.启动服务器
2. 多次命中限速端点以触发 429
3.检查429响应是否有`Retry-After` header
4. 检查该值是否为正整数

**执行：**```bash
# trigger the limit — 10 fast requests, limit is 5/sec per the diff
for i in {1..10}; do curl -s -o /dev/null -w "%{http_code}\n" localhost:3000/api/thing; done
# → 200 200 200 200 200 429 429 429 429 429

# capture the 429 headers
curl -si localhost:3000/api/thing | head -20
# → HTTP/1.1 429 Too Many Requests
# → Retry-After: 12
# → ...
```
**结论：** 通过 — `Retry-After: 12` 存在，正整数。

## 失败是什么样子

- 标头不存在→差异没有生效，或者你实际上没有
  打429路径（先检查状态码）
- 标头存在，但值为 `NaN` / `undefined` / 负 →
  逻辑是错误的
- 你已经完成了 200 秒 → 你从未触发过改变
  路径。收紧请求突发或检查速率限制配置。
