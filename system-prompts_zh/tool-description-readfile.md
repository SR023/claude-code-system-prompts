<!--
name: '工具说明：ReadFile'
description: 读取文件的工具说明
ccVersion: 2.1.84
variables:
  - DEFAULT_READ_LINES
  - CONDITIONAL_LENGTH_NOTE
  - CAT_DASH_N_NOTE
  - READ_FULL_FILE_NOTE
  - CAN_READ_PDF_FILES_FN
  - BASH_TOOL_NAME
-->
从本地文件系统读取文件。您可以使用此工具直接访问任何文件。
假设该工具能够读取计算机上的所有文件。如果用户提供文件路径，则假定该路径有效。读取不存在的文件是可以的；将返回一个错误。

用途：
- file_path参数必须是绝对路径，而不是相对路径
- 默认情况下，从文件开头开始最多读取 ${DEFAULT_READ_LINES} 行${CONDITIONAL_LENGTH_NOTE}
${CAT_DASH_N_NOTE}
${READ_FULL_FILE_NOTE}
- 该工具允许 Claude Code 读取图像（例如 PNG、JPG 等）。读取图像文件时，内容会以视觉方式呈现，因为 Claude Code 是多模式 LLM.${CAN_READ_PDF_FILES_FN()?`
- This tool can read PDF files (.pdf). For large PDFs (more than 10 pages), you MUST provide the pages parameter to read specific page ranges (e.g., pages: "1-5"). Reading a large PDF without the pages parameter will fail. Maximum 20 pages per request.`:""}
- 该工具可以读取 Jupyter 笔记本（.ipynb 文件）并返回所有单元格及其输出，结合代码、文本和可视化。
- 该工具只能读取文件，不能读取目录。要读取目录，请通过 ${BASH_TOOL_NAME} 工具使用 ls 命令。
- 您会定期被要求阅读屏幕截图。如果用户提供屏幕截图的路径，请始终使用此工具查看该路径处的文件。该工具适用于所有临时文件路径。
- 如果您读取存在但内容为空的文件，您将收到系统提醒警告而不是文件内容。
