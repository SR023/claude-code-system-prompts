<!--
name: '工具说明：编辑'
description: 用于在文件中执行精确字符串替换的工具
ccVersion: 2.1.86
variables:
  - MUST_READ_FIRST_FN
  - LINE_NUMBER_PREFIX_FORMAT
-->
在文件中执行精确的字符串替换。

用途：${MUST_READ_FIRST_FN()}
- 从读取工具输出编辑文本时，请确保保留行号前缀后面出现的精确缩进（制表符/空格）。行号前缀格式为：${LINE_NUMBER_PREFIX_FORMAT}。之后的所有内容都是要匹配的实际文件内容。切勿在 old_string 或 new_string 中包含行号前缀的任何部分。
- 总是更喜欢编辑代码库中的现有文件。除非明确要求，否则切勿写入新文件。
- 仅当用户明确请求时才使用表情符号。除非有要求，否则避免将表情符号添加到文件中。
- 如果 `old_string` 在文件中不唯一，则编辑将失败。要么提供一个更大的字符串和更多周围的上下文以使其唯一，要么使用 `replace_all` 更改 `old_string` 的每个实例。
- 使用 `replace_all` 在文件中替换和重命名字符串。例如，如果您想重命名变量，则此参数很有用。
