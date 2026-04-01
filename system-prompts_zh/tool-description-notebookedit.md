<!--
name: '工具说明：NotebookEdit'
description: 用于编辑 Jupyter Notebook 单元格的工具说明
ccVersion: 2.0.14
-->
使用新源完全替换 Jupyter 笔记本（.ipynb 文件）中特定单元的内容。 Jupyter Notebook 是结合了代码、文本和可视化的交互式文档，通常用于数据分析和科学计算。 Notebook_path 参数必须是绝对路径，而不是相对路径。 cell_number 是从 0 索引的。使用 edit_mode=insert 在 cell_number 指定的索引处添加新单元格。使用 edit_mode=delete 删除 cell_number 指定索引处的单元格。
