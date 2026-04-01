<!--
name: '技能：/循环斜杠命令'
description: 将用户输入解析为间隔和提示，将间隔转换为 cron 表达式，并安排重复任务
ccVersion: 2.1.79
variables:
  - CRON_CREATE_TOOL_NAME
  - DEFAULT_INTERVAL
  - CANCEL_TIMEFRAME_DAYS
  - CRON_DELETE_TOOL_NAME
  - USER_INPUT
-->
# /loop — 安排重复提示

将以下输入解析为 `[interval] <prompt…>` 并使用 ${CRON_CREATE_TOOL_NAME} 进行调度。

## 解析（按优先顺序）

1. **前导标记**：如果第一个空格分隔的标记与 `^\d+[smhd]$` 匹配（例如 `5m`、`2h`），则为间隔；剩下的就是提示了。
2. **尾随“every”子句**：否则，如果输入以 `every <N><unit>` 或 `every <N> <unit-word>` 结尾（例如 `every 20m`、`every 5 minutes`、`every 2 hours`），则将其提取为间隔并将其从提示中删除。仅当“every”后面是时间表达式时才匹配 - `check every PR` 没有间隔。
3. **默认**：否则，间隔为`ZXQMASK0001QXZ`，整个输入为提示。

如果生成的提示为空，则显示用法 `/loop [interval] <prompt>` 并停止 — 不调用 ${CRON_CREATE_TOOL_NAME}。

示例：
- `5m /babysit-prs`→区间`5m`，提示`/babysit-prs`（规则1）
- `check the deploy every 20m`→区间`20m`，提示`check the deploy`（规则2）
- `run tests every 5 minutes`→区间`5m`，提示`run tests`（规则2）
- `check the deploy`→区间`ZXQMASK0003QXZ`，提示`check the deploy`（规则3）
- `check every PR`→间隔`ZXQMASK0004QXZ`，提示`check every PR`（规则3——“每个”后面不跟时间）
- `5m` → 空提示 → 显示用法

## 间隔 → cron

支持的后缀：`s`（秒，向上舍入到最近的分钟，分钟为 1）、`m`（分钟）、`h`（小时）、`d`（天）。转换：

|间隔模式| cron 表达式 |笔记|
|------------------------|--------------------------------|------------------------------------------|
| `Nm` 其中 N ≤ 59 | `*/N * * * *` |每 N 分钟 |
| `Nm` 其中 N ≥ 60 | `0 */H * * *` |四舍五入到小时（H = N/60，必须除以 24）|
| `Nh` 其中 N ≤ 23 | `0 */N * * *` |每 N 小时 |
| `Nd` | `0 0 */N * *` |每 N 天午夜当地时间 |
| `Ns` |视为 `ceil(N/60)m` | cron 最小粒度为 1 分钟 |

**如果间隔没有清晰地划分其单位**（例如，`7m` → `*/7 * * * *` 在 :56→:00 处给出不均匀的间隙；`90m` → 1.5h，而 cron 无法表达），请选择最近的干净间隔，并在调度之前告诉用户您舍入的结果。

## 行动

1. 拨打 ${CRON_CREATE_TOOL_NAME}：
   - `cron`：上表中的表达式
   - `prompt`：上面解析的提示，逐字（斜杠命令不变地传递）
   - `recurring`：`true`
2. 简要确认：计划内容、cron 表达式、人类可读的节奏、重复任务在 ${CANCEL_TIMEFRAME_DAYS} 天后自动过期，并且可以使用 ${CRON_DELETE_TOOL_NAME} 更快地取消（包括作业 ID）。
3. **然后立即执行解析后的提示** - 不要等待第一个 cron 触发。如果是斜线命令，则通过技能工具调用；否则直接对其进行操作。

## 输入

${USER_INPUT}
