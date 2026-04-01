<!--
name: '数据：Claude API 参考 — C#'
description: C# SDK 参考，包括安装、客户端初始化、基本请求、流式传输和工具使用
ccVersion: 2.1.83
-->
# Claude API — C#

> **注意：** C# SDK 是 C# 的官方 Anthropic SDK。通过消息 API 支持工具的使用。基于类注释的工具运行程序不可用；使用带有 JSON 模式的原始工具定义。该 SDK 还支持 Microsoft.Extensions.AI IChatClient 与函数调用的集成。

＃＃ 安装```bash
dotnet add package Anthropic
```
## 客户端初始化```csharp
using Anthropic;

// Default (uses ANTHROPIC_API_KEY env var)
AnthropicClient client = new();

// Explicit API key (use environment variables — never hardcode keys)
AnthropicClient client = new() {
    ApiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY")
};
```
---

## 基本消息请求```csharp
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 16000,
    Messages = [new() { Role = Role.User, Content = "What is the capital of France?" }]
};
var response = await client.Messages.Create(parameters);

// ContentBlock is a union wrapper. .Value unwraps to the variant object,
// then OfType<T> filters to the type you want. Or use the TryPick* idiom
// shown in the Thinking section below.
foreach (var text in response.Content.Select(b => b.Value).OfType<TextBlock>())
{
    Console.WriteLine(text.Text);
}
```
---

## 流媒体```csharp
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 64000,
    Messages = [new() { Role = Role.User, Content = "Write a haiku" }]
};

await foreach (RawMessageStreamEvent streamEvent in client.Messages.CreateStreaming(parameters))
{
    if (streamEvent.TryPickContentBlockDelta(out var delta) &&
        delta.Delta.TryPickText(out var text))
    {
        Console.Write(text.Text);
    }
}
```
**`RawMessageStreamEvent` TryPick 方法**（命名去掉 `Message`/`Raw` 前缀）：`TryPickStart`、`TryPickDelta`、`TryPickStop`、`TryPickContentBlockStart`、`TryPickContentBlockDelta`、 `TryPickContentBlockStop`。没有 `TryPickMessageStop` — 使用 `TryPickStop`。

---

## 思考

**自适应思维是 Claude 4.6+ 模型的推荐模式。** Claude 动态决定思考的时间和程度。```csharp
using Anthropic.Models.Messages;

var response = await client.Messages.Create(new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 16000,
    // ThinkingConfigParam? implicitly converts from the concrete variant classes —
    // no wrapper needed.
    Thinking = new ThinkingConfigAdaptive(),
    Messages =
    [
        new() { Role = Role.User, Content = "Solve: 27 * 453" },
    ],
});

// ThinkingBlock(s) precede TextBlock in Content. TryPick* narrows the union.
foreach (var block in response.Content)
{
    if (block.TryPickThinking(out ThinkingBlock? t))
    {
        Console.WriteLine($"[thinking] {t.Thinking}");
    }
    else if (block.TryPickText(out TextBlock? text))
    {
        Console.WriteLine(text.Text);
    }
}
```
> **已弃用：** `new ThinkingConfigEnabled { BudgetTokens = N }`（固定预算扩展思维）仍然适用于 Claude 4.6，但已弃用。使用上面的适应性思维。

`TryPick*` 的替代方案：`.Select(b => b.Value).OfType<ThinkingBlock>()`（与基本消息示例相同的 LINQ 模式）。

---

## 工具使用

### 定义一个工具

`Tool`（不是 `ToolParam`），带有 `InputSchema` 记录。 `InputSchema.Type` 由构造函数自动设置为 `"object"` — 不要设置它。 `ToolUnion` 具有从 `Tool` 的隐式转换，由集合表达式 `[...]` 触发。```csharp
using System.Text.Json;
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeSonnet4_6,
    MaxTokens = 16000,
    Tools = [
        new Tool {
            Name = "get_weather",
            Description = "Get the current weather in a given location",
            InputSchema = new() {
                Properties = new Dictionary<string, JsonElement> {
                    ["location"] = JsonSerializer.SerializeToElement(
                        new { type = "string", description = "City name" }),
                },
                Required = ["location"],
            },
        },
    ],
    Messages = [new() { Role = Role.User, Content = "Weather in Paris?" }],
};
```
派生自 `anthropic-sdk-csharp/src/Anthropic/Models/Messages/Tool.cs` 和 `ToolUnion.cs:799`（隐式转换）。

有关循环模式，请参阅[共享工具使用概念](../shared/tool-use-concepts.md)。
### 将响应内容转换为后续辅助消息

当在助理回合中回应 Claude 的响应时，**没有 `.ToParam()` 助手** - 手动将每个 `ContentBlock` 变体重建为其 `*Param` 对应项。不要使用 `new ContentBlockParam(block.Json)`：它会编译并序列化，但 `.Value` 保持 `null`，因此 `TryPick*`/`Validate()` 失败（降级的 JSON 传递，而不是类型路径）。```csharp
using Anthropic.Models.Messages;

Message response = await client.Messages.Create(parameters);

// No .ToParam() — reconstruct per variant. Implicit conversions from each
// *Param type to ContentBlockParam mean no explicit wrapper.
List<ContentBlockParam> assistantContent = [];
List<ContentBlockParam> toolResults = [];
foreach (ContentBlock block in response.Content)
{
    if (block.TryPickText(out TextBlock? text))
    {
        assistantContent.Add(new TextBlockParam { Text = text.Text });
    }
    else if (block.TryPickThinking(out ThinkingBlock? thinking))
    {
        // Signature MUST be preserved — the API rejects tampering
        assistantContent.Add(new ThinkingBlockParam
        {
            Thinking = thinking.Thinking,
            Signature = thinking.Signature,
        });
    }
    else if (block.TryPickRedactedThinking(out RedactedThinkingBlock? redacted))
    {
        assistantContent.Add(new RedactedThinkingBlockParam { Data = redacted.Data });
    }
    else if (block.TryPickToolUse(out ToolUseBlock? toolUse))
    {
        // ToolUseBlock has required Caller; ToolUseBlockParam.Caller is optional — don't copy it
        assistantContent.Add(new ToolUseBlockParam
        {
            ID = toolUse.ID,
            Name = toolUse.Name,
            Input = toolUse.Input,
        });
        // Execute the tool; collect ONE result per tool_use block — the API
        // rejects the follow-up if any tool_use ID lacks a matching tool_result.
        string result = ExecuteYourTool(toolUse.Name, toolUse.Input);
        toolResults.Add(new ToolResultBlockParam
        {
            ToolUseID = toolUse.ID,
            Content = result,
        });
    }
}

// Follow-up: prior messages + assistant echo + user tool_result(s)
List<MessageParam> followUpMessages =
[
    .. parameters.Messages,
    new() { Role = Role.Assistant, Content = assistantContent },
    new() { Role = Role.User, Content = toolResults },
];
```
`ToolResultBlockParam` 没有元组构造函数 - 使用对象初始值设定项。 `Content` 是字符串或列表联合；普通的 `string` 隐式转换。

---

## 上下文编辑/压缩（测试版）

**Beta 命名空间前缀不一致**（根据 `src/Anthropic/Models/Beta/Messages/*.cs` @ 12.9.0 进行源验证）。无前缀：`MessageCreateParams`、`MessageCountTokensParams`、`Role`。 **其他所有内容都有 `Beta` 前缀**：`BetaMessageParam`、`BetaMessage`、`BetaContentBlock`、`BetaToolUseBlock`，所有块参数类型。如果导入两个命名空间 (CS0104)，无前缀的 `Role` 将与 `Anthropic.Models.Messages.Role` 发生冲突。最安全：仅导入Beta；如果混合，则将 beta 别名为 `Role`：```csharp
using Anthropic.Models.Beta.Messages;
using NonBeta = Anthropic.Models.Messages;  // only if you also need non-beta types
// Now: MessageCreateParams, BetaMessageParam, Role (beta's), NonBeta.Role (if needed)
```
`BetaMessage.Content` 是 `IReadOnlyList<BetaContentBlock>` — 一个 15 变体的歧视联合。用 `TryPick*` 缩小。 **响应 `BetaContentBlock` 不可分配给参数 `BetaContentBlockParam`** — C# 中没有 `.ToParam()`。通过转换每个块来往返：```csharp
using Anthropic.Models.Beta.Messages;

var betaParams = new MessageCreateParams   // no Beta prefix — one of only 2 unprefixed
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 16000,
    Betas = ["compact-2026-01-12"],
    ContextManagement = new BetaContextManagementConfig
    {
        Edits = [new BetaCompact20260112Edit()],
    },
    Messages = messages,
};
BetaMessage resp = await client.Beta.Messages.Create(betaParams);

foreach (BetaContentBlock block in resp.Content)
{
    if (block.TryPickCompaction(out BetaCompactionBlock? compaction))
    {
        // Content is nullable — compaction can fail server-side
        Console.WriteLine($"compaction summary: {compaction.Content}");
    }
}

// Context-edit metadata lives on a separate nullable field
if (resp.ContextManagement is { } ctx)
{
    foreach (var edit in ctx.AppliedEdits)
        Console.WriteLine($"cleared {edit.ClearedInputTokens} tokens");
}

// ROUND-TRIP: BetaMessageParam.Content is BetaMessageParamContent (a string|list
// union). It implicit-converts from List<BetaContentBlockParam>, NOT from the
// response's IReadOnlyList<BetaContentBlock>. Convert each block:
List<BetaContentBlockParam> paramBlocks = [];
foreach (var b in resp.Content)
{
    if (b.TryPickText(out var t)) paramBlocks.Add(new BetaTextBlockParam { Text = t.Text });
    else if (b.TryPickCompaction(out var c)) paramBlocks.Add(new BetaCompactionBlockParam { Content = c.Content });
    // ... other variants as needed
}
messages.Add(new BetaMessageParam { Role = Role.Assistant, Content = paramBlocks });
```
所有 15 个 `BetaContentBlock.TryPick*` 变体：`Text`、`Thinking`、`RedactedThinking`、`ToolUse`、`ServerToolUse`、`WebSearchToolResult`、`WebFetchToolResult`、 `CodeExecutionToolResult`、`BashCodeExecutionToolResult`、`TextEditorCodeExecutionToolResult`、`ToolSearchToolResult`、`McpToolUse`、`McpToolResult`、`ContainerUpload`、`Compaction`。

**`BetaToolUseBlock.Input` 是 `IReadOnlyDictionary<string, JsonElement>`** — 按键索引，然后调用 `JsonElement` 提取器：```csharp
if (block.TryPickToolUse(out BetaToolUseBlock? tu))
{
    int a = tu.Input["a"].GetInt32();
    string s = tu.Input["name"].GetString()!;
}
```
---

## 努力参数

工作量嵌套在 `OutputConfig` 下，而不是顶级属性。 `ApiEnum<string, Effort>` 具有来自枚举的隐式转换，因此直接分配 `Effort.High`。```csharp
OutputConfig = new OutputConfig { Effort = Effort.High },
```
值：`Effort.Low`、`Effort.Medium`、`Effort.High`、`Effort.Max`。与 `Thinking = new ThinkingConfigAdaptive()` 结合用于成本质量控制。

---

## 提示缓存

`System` 采用 `MessageCreateParamsSystem?` — `string` 或 `List<TextBlockParam>` 的并集。没有`SystemTextBlockParam`；使用普通 `TextBlockParam`。隐式转换需要具体的 `List<TextBlockParam>` 类型（数组文字不会转换）。有关放置模式和静默无效器审核清单，请参阅 `shared/prompt-caching.md`。```csharp
System = new List<TextBlockParam> {
    new() {
        Text = longSystemPrompt,
        CacheControl = new CacheControlEphemeral(),  // auto-sets Type = "ephemeral"
    },
},
```
`CacheControlEphemeral` 上可选 `Ttl`：`new() { Ttl = Ttl.Ttl1h }` 或 `Ttl.Ttl5m`。 `CacheControl` 也存在于 `Tool.CacheControl` 和顶级 `MessageCreateParams.CacheControl` 上。

通过 `response.Usage.CacheCreationInputTokens` / `response.Usage.CacheReadInputTokens` 验证命中。

---

## 令牌计数```csharp
MessageTokensCount result = await client.Messages.CountTokens(new MessageCountTokensParams {
    Model = Model.ClaudeOpus4_6,
    Messages = [new() { Role = Role.User, Content = "Hello" }],
});
long tokens = result.InputTokens;
```
`MessageCountTokensParams.Tools` 使用与 `MessageCreateParams.Tools` (`ToolUnion`) 不同的联合类型 (`MessageCountTokensTool`) — 如果您传递工具，编译器会在重要时告诉您。

---

## 结构化输出```csharp
OutputConfig = new OutputConfig {
    Format = new JsonOutputFormat {
        Schema = new Dictionary<string, JsonElement> {
            ["type"] = JsonSerializer.SerializeToElement("object"),
            ["properties"] = JsonSerializer.SerializeToElement(
                new { name = new { type = "string" } }),
            ["required"] = JsonSerializer.SerializeToElement(new[] { "name" }),
        },
    },
},
```
`JsonOutputFormat.Type` 由构造函数自动设置为 `"json_schema"`。 `Schema` 是 `required`。

---

## PDF/文档输入

`DocumentBlockParam` 采用 `DocumentBlockParamSource` 并集：`Base64PdfSource` / `UrlPdfSource` / `PlainTextSource` / `ContentBlockSource`。 `Base64PdfSource` 自动设置 `MediaType = "application/pdf"` 和 `Type = "base64"`。```csharp
new MessageParam {
    Role = Role.User,
    Content = new List<ContentBlockParam> {
        new DocumentBlockParam { Source = new Base64PdfSource { Data = base64String } },
        new TextBlockParam { Text = "Summarize this PDF" },
    },
}
```
---

## 服务器端工具

Web 搜索、bash、文本编辑器和代码执行是内置的服务器工具。类型名称带有版本后缀；构造函数自动设置 `name`/`type`。所有隐式转换为 `ToolUnion`。```csharp
Tools = [
    new WebSearchTool20260209(),
    new ToolBash20250124(),
    new ToolTextEditor20250728(),
    new CodeExecutionTool20260120(),
],
```
另提供：`WebFetchTool20260209`、`MemoryTool20250818`。 `WebSearchTool20260209` 可选：`AllowedDomains`、`BlockedDomains`、`MaxUses`、`UserLocation`。

---

## 文件 API（测试版）

文件位于 `client.Beta.Files`（命名空间 `Anthropic.Models.Beta.Files`）下。 `BinaryContent` 从 `Stream` 和 `byte[]` 隐式转换。```csharp
using Anthropic.Models.Beta.Files;
using Anthropic.Models.Beta.Messages;

FileMetadata meta = await client.Beta.Files.Upload(
    new FileUploadParams { File = File.OpenRead("doc.pdf") });

// Referencing the uploaded file requires Beta message types:
new BetaRequestDocumentBlock {
    Source = new BetaFileDocumentSource { FileID = meta.ID },
}
```
非测试版 `DocumentBlockParamSource` 联合没有文件 ID 变体 - 文件引用需要 `client.Beta.Messages.Create()`。
