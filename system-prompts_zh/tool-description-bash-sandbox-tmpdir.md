<!--
name: '工具说明：Bash（沙箱——tmpdir）'
description: 在沙盒模式下使用 $TMPDIR 作为临时文件
ccVersion: 2.1.86
-->
对于临时文件，请始终使用 `$TMPDIR` 环境变量。在沙箱模式下，TMPDIR 会自动设置为正确的沙箱可写目录。不要直接使用 `/tmp` - 请改用 `$TMPDIR`。
