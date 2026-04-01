<!--
name: '工具说明：LSP'
description: LSP 工具的说明。
ccVersion: 2.0.73
-->
与语言服务器协议 (LSP) 服务器交互以获得代码智能功能。

支持的操作：
- goToDefinition：查找定义符号的位置
- findReferences：查找对某个符号的所有引用
- 悬停：获取符号的悬停信息（文档、类型信息）
- documentSymbol：获取文档中的所有符号（函数、类、变量）
-workspaceSymbol：在整个工作空间中搜索符号
- goToImplementation：查找接口或抽象方法的实现
-prepareCallHierarchy：获取某个位置的调用层次结构项（函数/方法）
-comingCalls：查找在某个位置调用该函数的所有函数/方法
-outgoingCalls：查找某个位置的函数调用的所有函数/方法

所有操作都需要：
- filePath：要操作的文件
- line：行号（从 1 开始，如编辑器中所示）
- 字符：字符偏移量（从1开始，如编辑器中所示）

注意：必须为该文件类型配置 LSP 服务器。如果没有可用的服务器，将返回错误。
