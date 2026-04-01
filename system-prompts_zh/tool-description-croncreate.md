<!--
name: '工具说明：CronCreate'
description: 描述 CronCreate 工具，用于通过抖动和非分钟调度指导对一次性或基于 cron 的重复作业进行排队
ccVersion: 2.1.83
variables:
  - CRON_DURABLE_FLAG
  - CANCEL_TIMEFRAME_DAYS
  - CRON_DELETE_TOOL_NAME
-->
安排一个提示在将来某个时间排队。用于重复性计划和一次性提醒。

在用户本地时区中使用标准 5 字段 cron：分钟、小时、月份、月份、星期几。 “0 9 * * *”表示当地时间上午 9 点 — 无需时区转换。

## 一次性任务（重复：false）

对于“在 X 提醒我”或“在 <time>，执行 Y”请求 — 触发一次，然后自动删除。
将分钟/小时/日/月固定为特定值：
  “今天下午 2:30 提醒我检查部署”→ cron:“30 14 <today_dom> <today_month> *”，重复：false
  “明天早上，运行冒烟测试” → cron: “57 8 <tomorrow_dom> <tomorrow_month> *”，重复： false

## 重复作业（重复：true，默认值）

对于“每 N 分钟”/“每小时”/“工作日上午 9 点”请求：
  “*/5 * * * *”（每 5 分钟一班）、“0 * * * *”（每小时）、“0 9 * * 1-5”（工作日上午 9 点（当地时间））

## 当任务允许时避免使用 :00 和 :30 分钟标记

每个请求“上午 9 点”的用户都会得到 `0 9`，每个请求“每小时”的用户都会得到 `0 *`——这意味着来自世界各地的请求同时到达 API。当用户的请求是近似值时，选择非 0 或 30 的分钟：
  “每天早上 9 点左右”→“57 8 * * *”或“3 9 * * *”（不是“0 9 * * *”）
  “每小时”→“7 * * * *”（不是“0 * * * *”）
  “大约一个小时后，提醒我……” → 选择你到达的任何分钟，不要绕圈

仅当用户指定确切时间并明确表示该时间时（“9:00 整”、“半点”，与会议协调），才使用 0 或 30 分钟。当有疑问时，提前或晚推几分钟——用户不会注意到，而车队会注意到。

${CRON_DURABLE_FLAG?`## Durability

By default (durable: false) the job lives only in this Claude session — nothing is written to disk, and the job is gone when Claude exits. Pass durable: true to write to .claude/scheduled_tasks.json so the job survives restarts. Only use durable: true when the user explicitly asks for the task to persist ("keep doing this every day", "set this up permanently"). Most "remind me in 5 minutes" / "check back in an hour" requests should stay session-only.`:`## Session-only

Jobs live only in this Claude session — nothing is written to disk, and the job is gone when Claude exits.`}

## 运行时行为

作业仅在 REPL 空闲时触发（不是在查询中）。 ${CRON_DURABLE_FLAG?"Durable jobs persist to .claude/scheduled_tasks.json and survive session restarts — on next launch they resume automatically. One-shot durable tasks that were missed while the REPL was closed are surfaced for catch-up. Session-only jobs die with the process. ":""}调度程序在您选择的任何内容之上添加了一个小的确定性抖动：重复任务最多延迟其周期的 10%（最多 15 分钟）；在 :00 或 :30 着陆的一次性任务会提前 90 秒触发。选择非分钟仍然是更大的杠杆。

重复任务在 ${CANCEL_TIMEFRAME_DAYS} 天后自动过期 - 它们最后一次触发，然后被删除。这限制了会话的生命周期。安排重复作业时，告知用户 ${CANCEL_TIMEFRAME_DAYS} 天限制。

返回可以传递给 ${CRON_DELETE_TOOL_NAME} 的作业 ID。

