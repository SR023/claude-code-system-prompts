<!--
name: '工具说明：配置'
description: 用于获取和设置 Claude Code 配置设置的工具，包含使用说明和可配置设置列表
ccVersion: 2.1.88
variables:
  - GLOBAL_SETTINGS_LIST
  - PROJECT_SETTINGS_LIST
  - ADDITIONAL_SETTINGS_NOTE
-->
获取或设置 Claude Code 配置设置。

  查看或更改克劳德代码设置。当用户请求配置更改、询问当前设置或调整设置对他们有利时使用。


## 用法
- **获取当前值：** 省略“value”参数
- **设置新值：** 包含“value”参数

## 可配置的设置列表
您可以更改以下设置：

### 全局设置（存储在 ~/.claude.json 中）
${GLOBAL_SETTINGS_LIST.join(`
`)}

### 项目设置（存储在settings.json中）
${PROJECT_SETTINGS_LIST.join(`
`)}

${ADDITIONAL_SETTINGS_NOTE}
## 示例
- 获取主题：{“设置”：“主题”}
- 设置深色主题：{ "setting": "theme", "value": "dark" }
- 启用 vim 模式: { "setting": "editorMode", "value": "vim" }
- 启用详细：{ "setting": "verbose", "value": true }
- 更改模型：{ "setting": "model", "value": "opus" }
- 更改权限模式：{ "setting": "permissions.defaultMode", "value": "plan" }
