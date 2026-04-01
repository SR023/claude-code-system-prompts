<!--
name: '技能：验证 CLI 更改（验证技能的示例）'
description: 作为验证技能的一部分，用于验证 CLI 更改的示例工作流程。
ccVersion: 2.1.83
-->
# 验证 CLI 更改

句柄是直接调用。证据是 stdout/stderr/exit 代码。

## 模式

1. 构建（如果CLI需要构建）
2. 使用执行更改后的代码的参数运行
3. 捕获输出和退出代码
4. 与预期比较

CLI 通常是最容易验证的——没有生命周期，没有端口。

## 工作示例

**差异：** 将 `--json` 标志添加到 `status` 子命令。新旗帜
在`cmd/status.go`中解析，新的输出分支。

**声明（提交消息）：**“机器可读的状态输出。”

**推论：** `tool status --json` 现在存在，发出有效的 JSON
人类输出显示的相同领域。 `tool status` 无标志
是不变的。

**计划：**
1. 构建
2. `tool status` → 人力输出，与之前相同（非回归）
3. `tool status --json` → 有效的 JSON，可解析
4. JSON 字段与人工输出字段匹配

**执行：**```bash
go build -o /tmp/tool ./cmd/tool

/tmp/tool status
# → Status: healthy
# → Uptime: 3h12m
# → Connections: 47

/tmp/tool status --json
# → {"status":"healthy","uptime_seconds":11520,"connections":47}

/tmp/tool status --json | jq -e .status
# → "healthy"
# (jq -e exits nonzero if the path is null/false — cheap validity check)

echo $?
# → 0
```
**结论：** PASS — 标志有效，JSON 有效，字段排列。

## 失败是什么样子

- `unknown flag: --json` → 未连接，或者您正在运行陈旧的版本
- 输出不是有效的 JSON（`jq` 错误）→ 序列化错误
- `tool status`（无标志）更改→回归；差异触动更多
  比它应该的
- JSON 的字段名称与预期不同 → 声明/代码不匹配，
  可能没问题，注意一下

## 从标准输入读取，破坏性命令

如果 CLI 读取测试数据中的 stdin → 管道。
如果它写入文件/访问网络/删除内容→将其指向
tmp dir / 模拟 / 空运行标志。如果没有安全模式并且
diff 触及破坏性路径，说出来并验证你能做什么
围绕它。
