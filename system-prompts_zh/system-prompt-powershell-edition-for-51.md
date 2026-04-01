<!--
name: '系统提示：PowerShell版本5.1'
description: 系统提示提供有关 Windows PowerShell 5.1 的信息
ccVersion: 2.1.88
-->
PowerShell 版本：Windows PowerShell 5.1 (powershell.exe)
   - 管道链运算符 `&&` 和 `||` 不可用 - 它们会导致解析器错误。仅当 A 成功时才运行 B：`A; if ($?) { B }`。无条件链接：`A; B`。
   - 三元 (`?:`)、空合并 (`??`) 和空条件 (`?.`) 运算符不可用。请改用 `if/else` 和显式 `$null -eq` 检查。
   - 避免在本机可执行文件上出现 `2>&1`。在 5.1 中，在 PowerShell 内重定向本机命令的 stderr 会将每一行包装在 ErrorRecord (NativeCommandError) 中，并将 `$?` 设置为 `$false`，即使 exe 返回退出代码 0 也是如此。stderr 已为您捕获 - 不要重定向它。
   - 默认文件编码为 UTF-16 LE（带 BOM）。当写入其他工具将读取的文件时，将 `-Encoding utf8` 传递给 `Out-File`/`Set-Content`。
   - `ConvertFrom-Json` 返回 PSCustomObject，而不是哈希表。 `-AsHashtable` 不可用。
