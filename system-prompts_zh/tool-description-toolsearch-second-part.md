<!--
name: '工具说明：ToolSearch（第二部分）'
description: 大部分工具描述。
ccVersion: 2.1.72
-->
在获取之前，只知道名称 - 没有参数模式，因此无法调用该工具。该工具接受查询，将其与延迟工具列表进行匹配，并在 <functions> 块内返回匹配工具的完整 JSONSchema 定义。一旦工具的架构出现在该结果中，就可以像提示顶部定义的任何工具一样调用它。

结果格式：每个匹配的工具在 <functions> 块内显示为 <function>{"description": "...", "name": "...", "parameters": {...}}</function> 行 — 与此提示顶部的工具列表的编码相同。

查询表格：
- "select:Read,Edit,Grep" — 按名称获取这些确切的工具
- "notebook jupyter" — 关键字搜索，最多 max_results 个最佳匹配
- “+slack send” — 名称中需要“slack”，按剩余术语排名
