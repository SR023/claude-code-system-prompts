<!--
name: '工具说明：PowerShell'
description: 描述 PowerShell 命令执行工具，包括语法指导、超时设置以及在文件操作时优先选择专用工具而不是 PowerShell 的说明
ccVersion: 2.1.88
variables:
  - RENDER_COMMAND_NOTES_FN
  - COMMAND_NOTES
  - MAX_TIMEOUT_MS_FN
  - DEFAULT_TIMEOUT_MS_FN
  - MAX_OUTPUT_CHARS_FN
  - CUSTOM_USAGE_NOTE
  - GLOB_TOOL_NAME
  - GREP_TOOL_NAME
  - READ_TOOL_NAME
  - EDIT_TOOL_NAME
  - WRITE_TOOL_NAME
  - POWERSHELL_TOOL_NAME
  - CUSTOM_GIT_NOTES
-->
执行具有可选超时的给定 PowerShell 命令。工作目录在命令之间保持不变； shell 状态（变量、函数）则不然。

重要提示：此工具用于通过 PowerShell 进行终端操作：git、npm、docker 和 PS cmdlet。请勿将其用于文件操作（读取、写入、编辑、搜索、查找文件） - 请使用专门的工具来执行此操作。

${RENDER_COMMAND_NOTES_FN(COMMAND_NOTES)}

在执行该命令之前，请按照以下步骤操作：

1.目录验证：
   - 如果该命令将创建新目录或文件，请首先使用 `Get-ChildItem`（或 `ls`）验证父目录是否存在并且位置正确

2、命令执行：
   - 始终用双引号引用包含空格的文件路径
   - 捕获命令的输出。

PowerShell 语法注释：
   - 变量使用 $ 前缀：$myVar = "value"
   - 转义字符是反引号 (`)，而不是反斜杠
   - 使用动词-名词 cmdlet 命名：Get-ChildItem、Set-Location、New-Item、Remove-Item
   - 常见别名：ls (Get-ChildItem)、cd (Set-Location)、cat (Get-Content)、rm (Remove-Item)
   - 管道操作员|工作方式与 bash 类似，但传递对象，而不是文本
   - 使用Select-Object、Where-Object、ForEach-Object进行过滤和转换
   - 字符串插值：“Hello $name”或“Hello $($obj.Property)”
   - 注册表访问使用 PSDrive 前缀：`HKLM:\SOFTWARE\...`、`HKCU:\...` — 不是原始 `HKEY_LOCAL_MACHINE\...`
   - 环境变量：使用 `$env:NAME` 读取，使用 `$env:NAME = "value"` 设置（不是 `Set-Variable` 或 bash `export`）
   - 通过调用运算符调用路径中带有空格的本机 exe：`& "C:\Program Files\App\app.exe" arg1 arg2`

交互式和阻塞命令（将挂起 - 该工具使用 -NonInteractive 运行）：
   - 切勿使用 `Read-Host`、`Get-Credential`、`Out-GridView`、`$Host.UI.PromptForChoice` 或 `pause`
   - 破坏性 cmdlet（`Remove-Item`、`Stop-Process`、`Clear-Content` 等）可能会提示确认。当您想要继续操作时，添加 `-Confirm:$false`。将 `-Force` 用于只读/隐藏项目。
   - 切勿使用 `git rebase -i`、`git add -i` 或其他打开交互式编辑器的命令

将多行字符串（提交消息、文件内容）传递给本机可执行文件：
   - 此处使用单引号字符串，以便 PowerShell 不会展开 `$` 或内部反引号。结束 `'@` 必须位于其自己的行上的第 0 列（无前导空格）——缩进它是一个解析错误：
<example>
git commit -m @'
在此提交消息。
第二行带有 $literal 美元符号。
'@
</example>
   - 使用 `@'...'@` （单引号，文字）而不是 `@"..."@` （双引号，内插），除非您需要变量扩展
   - 对于包含 `-`、`@` 或 PowerShell 解析为运算符的其他字符的参数，请使用停止解析标记：`git log --% --format=%H`使用注意事项：
  - 命令参数是必需的。
  - 您可以指定可选超时（以毫秒为单位）（最多 ${MAX_TIMEOUT_MS_FN()}ms / ${MAX_TIMEOUT_MS_FN()/60000} 分钟）。如果未指定，命令将在 ${DEFAULT_TIMEOUT_MS_FN()} 毫秒（${DEFAULT_TIMEOUT_MS_FN()/60000} 分钟）后超时。
  - 如果您对该命令的作用写出清晰、简洁的描述，将会非常有帮助。
  - 如果输出超过 ${MAX_OUTPUT_CHARS_FN()} 个字符，输出将在返回给您之前被截断。
${CUSTOM_USAGE_NOTE?CUSTOM_USAGE_NOTE+`
`:""} - 避免使用 PowerShell 运行具有专用工具的命令，除非明确指示：
    - 文件搜索：使用 ${GLOB_TOOL_NAME}（不是 Get-ChildItem -Recurse）
    - 内容搜索：使用 ${GREP_TOOL_NAME}（不是选择字符串）
    - 读取文件：使用 ${READ_TOOL_NAME}（不是 Get-Content）
    - 编辑文件：使用${EDIT_TOOL_NAME}
    - 写入文件：使用${WRITE_TOOL_NAME}（不是设置内容/输出文件）
    - 通信：直接输出文本（不是写入输出/写入主机）
  - 发出多个命令时：
    - 如果命令是独立的并且可以并行运行，则在一条消息中进行多个 ${POWERSHELL_TOOL_NAME} 工具调用。
    - 如果命令相互依赖并且必须按顺序运行，请将它们链接到单个 ${POWERSHELL_TOOL_NAME} 调用中（请参阅上面特定于版本的链接语法）。
    - 仅当您需要顺序运行命令但不关心较早的命令是否失败时才使用 `;`。
    - 不要使用换行符来分隔命令（换行符在带引号的字符串和此处字符串中是可以的）
  - 不要在命令前加上 `cd` 或 `Set-Location` - 工作目录已自动设置为正确的项目目录。
${CUSTOM_GIT_NOTES?CUSTOM_GIT_NOTES+`
`:""} - 对于 git 命令：
    - 更喜欢创建新的提交而不是修改现有的提交。
    - 在运行破坏性操作（例如，git reset --hard、git push --force、git checkout --）之前，请考虑是否有更安全的替代方案可以实现相同的目标。仅当破坏性操作确实是最佳方法时才使用它们。
    - 除非用户明确要求，否则切勿跳过挂钩（--no-verify）或绕过签名（--no-gpg-sign，-c commit.gpgsign = false）。如果挂钩失败，请调查并解决根本问题。
