<!--
name: '座席提示：最近消息摘要'
description: 用于总结最近消息的代理提示。
ccVersion: 2.1.84
-->
您的任务是创建对话的最近部分的详细摘要 - 先前保留的上下文后面的消息。早期的消息保持完整，不需要总结。您的总结仅集中在最近的消息中讨论、学到和完成的内容上。

${`Before providing your final summary, wrap your analysis in <analysis> tags to organize your thoughts and ensure you've covered all necessary points. In your analysis process:

1. Analyze the recent messages chronologically. For each section thoroughly identify:
   - The user's explicit requests and intents
   - Your approach to addressing the user's requests
   - Key decisions, technical concepts and code patterns
   - Specific details like:
     - file names
     - full code snippets
     - function signatures
     - file edits
   - Errors that you ran into and how you fixed them
   - Pay special attention to specific user feedback that you received, especially if the user told you to do something differently.
2. Double-check for technical accuracy and completeness, addressing each required element thoroughly.`}

您的摘要应包括以下部分：

1. 主要请求和意图：从最近的消息中捕获用户的明确请求和意图
2.关键技术概念：列出最近讨论的重要技术概念、技术和框架。
3. 文件和代码部分：枚举检查、修改或创建的特定文件和代码部分。在适用的情况下包含完整的代码片段，并包含有关读取或编辑此文件的重要性的摘要。
4. 错误和修复：列出遇到的错误以及修复方法。
5. 解决问题：记录已解决的问题以及任何正在进行的故障排除工作。
6. 所有用户消息：列出最近部分中非工具结果的所有用户消息。
7. 待处理任务：概述最近消息中的所有待处理任务。
8. 当前工作：准确描述在此摘要请求之前正在进行的工作。
9. 可选下一步：列出与最近工作相关的下一步。包括最近对话中的直接引用。

以下是如何构建输出的示例：

<example>
<analysis>
[你的思考过程，确保所有要点都得到彻底、准确的涵盖]
</analysis>

<summary>
1. 主要请求和意图：
   [详细说明]

2. 关键技术概念：
   - [概念1]
   - [概念2]

3. 文件和代码部分：
   - [文件名1]
      - [总结为什么这个文件很重要]
      - [重要代码片段]

4. 错误及修正：
    - [错误描述]：
      - [你是如何解决的]

5、问题解决：
   [说明]

6. 所有用户留言：
    - [详细的非工具使用用户消息]

7. 待处理任务：
   - [任务1]

8. 目前的工作：
   [当前工作的精确描述]

9. 可选的下一步：
   [可选的下一步要采取的步骤]

</summary>
</example>

请仅根据最近的消息（在保留之前的上下文之后）提供您的摘要，遵循此结构并确保您的回复的准确性和彻底性。
