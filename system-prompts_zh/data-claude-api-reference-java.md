<!--
name: '数据：Claude API 参考 — Java'
description: Java SDK 参考，包括安装、客户端初始化、基本请求、流式传输和 Beta 工具使用
ccVersion: 2.1.83
-->
# Claude API — Java

> **注意：** Java SDK 支持 Claude API 和 beta 工具与带注释的类一起使用。 Agent SDK 尚不可用于 Java。

## 安装

行家：```xml
<dependency>
    <groupId>com.anthropic</groupId>
    <artifactId>anthropic-java</artifactId>
    <version>2.17.0</version>
</dependency>
```
摇篮：```groovy
implementation("com.anthropic:anthropic-java:2.17.0")
```
## 客户端初始化```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

// Default (reads ANTHROPIC_API_KEY from environment)
AnthropicClient client = AnthropicOkHttpClient.fromEnv();

// Explicit API key
AnthropicClient client = AnthropicOkHttpClient.builder()
    .apiKey("your-api-key")
    .build();
```
---

## 基本消息请求```java
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.Model;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(16000L)
    .addUserMessage("What is the capital of France?")
    .build();

Message response = client.messages().create(params);
response.content().stream()
    .flatMap(block -> block.text().stream())
    .forEach(textBlock -> System.out.println(textBlock.text()));
```
---

## 流媒体```java
import com.anthropic.core.http.StreamResponse;
import com.anthropic.models.messages.RawMessageStreamEvent;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(64000L)
    .addUserMessage("Write a haiku")
    .build();

try (StreamResponse<RawMessageStreamEvent> streamResponse = client.messages().createStreaming(params)) {
    streamResponse.stream()
        .flatMap(event -> event.contentBlockDelta().stream())
        .flatMap(deltaEvent -> deltaEvent.delta().text().stream())
        .forEach(textDelta -> System.out.print(textDelta.text()));
}
```
---

## 思考

**自适应思维是 Claude 4.6+ 模型的推荐模式。** Claude 动态决定思考的时间和程度。构建器具有直接的 `.thinking(ThinkingConfigAdaptive)` 过载 - 无需手动联合包装。```java
import com.anthropic.models.messages.ContentBlock;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;
import com.anthropic.models.messages.ThinkingConfigAdaptive;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_SONNET_4_6)
    .maxTokens(16000L)
    .thinking(ThinkingConfigAdaptive.builder().build())
    .addUserMessage("Solve this step by step: 27 * 453")
    .build();

for (ContentBlock block : client.messages().create(params).content()) {
    block.thinking().ifPresent(t -> System.out.println("[thinking] " + t.thinking()));
    block.text().ifPresent(t -> System.out.println(t.text()));
}
```
> **已弃用：** `ThinkingConfigEnabled.builder().budgetTokens(N)`（和 `.enabledThinking(N)` 快捷方式）仍然适用于 Claude 4.6，但已弃用。使用上面的适应性思维。

`ContentBlock` 缩小：`.thinking()` / `.text()` 返回 `Optional<T>` — 使用 `.ifPresent(...)` 或 `.stream().flatMap(...)`。替代方案：`isThinking()` / `asThinking()` 布尔+展开对（抛出错误的变体）。

---

## 工具使用（测试版）

Java SDK 支持测试版工具与带注释的类一起使用。工具类实现 `Supplier<String>`，以便通过 `BetaToolRunner` 自动执行。

### Tool Runner（自动循环）```java
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.BetaMessage;
import com.anthropic.helpers.BetaToolRunner;
import com.fasterxml.jackson.annotation.JsonClassDescription;
import com.fasterxml.jackson.annotation.JsonPropertyDescription;
import java.util.function.Supplier;

@JsonClassDescription("Get the weather in a given location")
static class GetWeather implements Supplier<String> {
    @JsonPropertyDescription("The city and state, e.g. San Francisco, CA")
    public String location;

    @Override
    public String get() {
        return "The weather in " + location + " is sunny and 72°F";
    }
}

BetaToolRunner toolRunner = client.beta().messages().toolRunner(
    MessageCreateParams.builder()
        .model("{{OPUS_ID}}")
        .maxTokens(16000L)
        .putAdditionalHeader("anthropic-beta", "structured-outputs-2025-11-13")
        .addTool(GetWeather.class)
        .addUserMessage("What's the weather in San Francisco?")
        .build());

for (BetaMessage message : toolRunner) {
    System.out.println(message);
}
```
### 记忆工具

Java SDK 提供了 `BetaMemoryToolHandler` 用于实现内存工具后端。您提供一个管理文件存储的处理程序，`BetaToolRunner` 自动处理内存工具调用。```java
import com.anthropic.helpers.BetaMemoryToolHandler;
import com.anthropic.helpers.BetaToolRunner;
import com.anthropic.models.beta.messages.BetaMemoryTool20250818;
import com.anthropic.models.beta.messages.BetaMessage;
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.ToolRunnerCreateParams;

// Implement BetaMemoryToolHandler with your storage backend (e.g., filesystem)
BetaMemoryToolHandler memoryHandler = new FileSystemMemoryToolHandler(sandboxRoot);

MessageCreateParams createParams = MessageCreateParams.builder()
    .model("{{OPUS_ID}}")
    .maxTokens(4096L)
    .addTool(BetaMemoryTool20250818.builder().build())
    .addUserMessage("Remember that my favorite color is blue")
    .build();

BetaToolRunner toolRunner = client.beta().messages().toolRunner(
    ToolRunnerCreateParams.builder()
        .betaMemoryToolHandler(memoryHandler)
        .initialMessageParams(createParams)
        .build());

for (BetaMessage message : toolRunner) {
    System.out.println(message);
}
```
有关内存工具的更多详细信息，请参阅[共享内存工具概念](../shared/tool-use-concepts.md)。

### 非 Beta 工具声明（手动 JSON 模式）

`Tool.InputSchema.Properties` 是一个自由格式的 `Map<String, JsonValue>` 包装器 - 通过 `putAdditionalProperty` 构建属性模式。 `type: "object"` 是默认值。构建器有一个直接的 `.addTool(Tool)` 重载，它自动包装在 `ToolUnion` 中。```java
import com.anthropic.core.JsonValue;
import com.anthropic.models.messages.Tool;

Tool tool = Tool.builder()
    .name("get_weather")
    .description("Get the current weather in a given location")
    .inputSchema(Tool.InputSchema.builder()
        .properties(Tool.InputSchema.Properties.builder()
            .putAdditionalProperty("location", JsonValue.from(Map.of("type", "string")))
            .build())
        .required(List.of("location"))
        .build())
    .build();

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_SONNET_4_6)
    .maxTokens(16000L)
    .addTool(tool)
    .addUserMessage("Weather in Paris?")
    .build();
```
对于手动工具循环，处理响应中的 `tool_use` 块，发送回 `tool_result`，循环直到 `stop_reason` 为 `"end_turn"`。请参阅[共享工具使用概念](../shared/tool-use-concepts.md)。

### 使用内容块构建 `MessageParam`（工具结果往返）

`MessageParam.Content` 是一个内部联合类（字符串 | 列表）。使用构建器的 `.contentOfBlockParams(List<ContentBlockParam>)` 别名 - 没有具有静态 `ofBlockParams` 的单独 `MessageParamContent` 类：```java
import com.anthropic.models.messages.MessageParam;
import com.anthropic.models.messages.ContentBlockParam;
import com.anthropic.models.messages.ToolResultBlockParam;

List<ContentBlockParam> results = List.of(
    ContentBlockParam.ofToolResult(ToolResultBlockParam.builder()
        .toolUseId(toolUseBlock.id())
        .content(yourResultString)
        .build())
);

MessageParam toolResultMsg = MessageParam.builder()
    .role(MessageParam.Role.USER)
    .contentOfBlockParams(results)   // builder alias for Content.ofBlockParams(...)
    .build();
```
---

## 努力参数

努力嵌套在 `OutputConfig` 内部 - `MessageCreateParams.Builder` 上没有直接的 `.effort()`。```java
import com.anthropic.models.messages.OutputConfig;

.outputConfig(OutputConfig.builder()
    .effort(OutputConfig.Effort.HIGH)  // or LOW, MEDIUM, MAX
    .build())
```
与 `Thinking = ThinkingConfigAdaptive` 结合用于成本质量控制。

---

## 提示缓存

系统消息为 `TextBlockParam` 和 `CacheControlEphemeral` 的列表。使用 `.systemOfTextBlockParams(...)` — 普通 `.system(String)` 重载无法进行缓存控制。有关放置模式和静默无效器审核清单，请参阅 `shared/prompt-caching.md`。```java
import com.anthropic.models.messages.TextBlockParam;
import com.anthropic.models.messages.CacheControlEphemeral;

.systemOfTextBlockParams(List.of(
    TextBlockParam.builder()
        .text(longSystemPrompt)
        .cacheControl(CacheControlEphemeral.builder()
            .ttl(CacheControlEphemeral.Ttl.TTL_1H)  // optional; also TTL_5M
            .build())
        .build()))
```
`MessageCreateParams.Builder` 和 `Tool.builder()` 上还有顶级 `.cacheControl(CacheControlEphemeral)`。

通过 `response.usage().cacheCreationInputTokens()` / `response.usage().cacheReadInputTokens()` 验证命中。

---

## 令牌计数```java
import com.anthropic.models.messages.MessageCountTokensParams;

long tokens = client.messages().countTokens(
    MessageCountTokensParams.builder()
        .model(Model.CLAUDE_SONNET_4_6)
        .addUserMessage("Hello")
        .build()
).inputTokens();
```
---

## 结构化输出

基于类的重载会自动从 POJO 派生 JSON 模式，并为您提供类型化的 `.text()` 返回 - 无需手动模式，无需手动解析。```java
import com.anthropic.models.messages.StructuredMessageCreateParams;

record Book(String title, String author) {}
record BookList(List<Book> books) {}

StructuredMessageCreateParams<BookList> params = MessageCreateParams.builder()
    .model(Model.CLAUDE_SONNET_4_6)
    .maxTokens(16000L)
    .outputConfig(BookList.class)  // returns a typed builder
    .addUserMessage("List 3 classic novels")
    .build();

client.messages().create(params).content().stream()
    .flatMap(cb -> cb.text().stream())
    .forEach(typed -> {
        // typed.text() returns BookList, not String
        for (Book b : typed.text().books()) System.out.println(b.title());
    });
```
支持 Jackson 注释：`@JsonPropertyDescription`、`@JsonIgnore`、`@ArraySchema(minItems=...)`。手动架构路径：`OutputConfig.builder().format(JsonOutputFormat.builder().schema(...).build())`。

---

## PDF/文档输入

`DocumentBlockParam` 构建器具有源快捷方式。包裹在`ContentBlockParam.ofDocument()`中并通过`.addUserMessageOfBlockParams()`。```java
import com.anthropic.models.messages.DocumentBlockParam;
import com.anthropic.models.messages.ContentBlockParam;
import com.anthropic.models.messages.TextBlockParam;

DocumentBlockParam doc = DocumentBlockParam.builder()
    .base64Source(base64String)  // or .urlSource("https://...") or .textSource("...")
    .title("My Document")        // optional
    .build();

.addUserMessageOfBlockParams(List.of(
    ContentBlockParam.ofDocument(doc),
    ContentBlockParam.ofText(TextBlockParam.builder().text("Summarize this").build())))
```
---

## 服务器端工具

版本后缀类型； `name`/`type` 由构建器自动设置。每种类型都存在直接 `.addTool()` 重载 — 无需手动 `ToolUnion` 包装。```java
import com.anthropic.models.messages.WebSearchTool20260209;
import com.anthropic.models.messages.ToolBash20250124;
import com.anthropic.models.messages.ToolTextEditor20250728;
import com.anthropic.models.messages.CodeExecutionTool20260120;

.addTool(WebSearchTool20260209.builder()
    .maxUses(5L)                              // optional
    .allowedDomains(List.of("example.com"))   // optional
    .build())
.addTool(ToolBash20250124.builder().build())
.addTool(ToolTextEditor20250728.builder().build())
.addTool(CodeExecutionTool20260120.builder().build())
```
另提供：`WebFetchTool20260209`、`MemoryTool20250818`、`ToolSearchToolBm25_20251119`。

### Beta 命名空间（MCP、压缩）

对于仅限测试版的功能，请使用 `com.anthropic.models.beta.messages.*` — 类名具有 `Beta` 前缀并存在于测试版包中。 beta `MessageCreateParams.Builder` 具有直接的 `.addTool(BetaToolBash20250124)` 重载和 `.addMcpServer()`：```java
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.BetaToolBash20250124;
import com.anthropic.models.beta.messages.BetaCodeExecutionTool20260120;
import com.anthropic.models.beta.messages.BetaRequestMcpServerUrlDefinition;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(16000L)
    .addBeta("mcp-client-2025-11-20")
    .addTool(BetaToolBash20250124.builder().build())
    .addTool(BetaCodeExecutionTool20260120.builder().build())
    .addMcpServer(BetaRequestMcpServerUrlDefinition.builder()
        .name("my-server")
        .url("https://example.com/mcp")
        .build())
    .addUserMessage("...")
    .build();

client.beta().messages().create(params);
```
`BetaTool*` 类型不可与非 beta `Tool*` 互换 — 每个请求选择一个命名空间。

**读取响应中的服务器工具块：** `ServerToolUseBlock` 具有 `.id()`、`.name()`（枚举）和 `._input()` 返回原始 `JsonValue` — 没有键入的 `.input()`。对于代码执行结果，展开两个级别：```java
for (ContentBlock block : response.content()) {
    block.serverToolUse().ifPresent(stu -> {
        System.out.println("tool: " + stu.name() + " input: " + stu._input());
    });
    block.codeExecutionToolResult().ifPresent(r -> {
        r.content().resultBlock().ifPresent(result -> {
            System.out.println("stdout: " + result.stdout());
            System.out.println("stderr: " + result.stderr());
            System.out.println("exit: " + result.returnCode());
        });
    });
}
```
---

## 文件 API（测试版）

在 `client.beta().files()` 下。消息中的文件引用需要测试版消息类型（非测试版 `DocumentBlockParam.Source` 没有文件 ID 变体）。```java
import com.anthropic.models.beta.files.FileUploadParams;
import com.anthropic.models.beta.files.FileMetadata;
import com.anthropic.models.beta.messages.BetaRequestDocumentBlock;
import java.nio.file.Paths;

FileMetadata meta = client.beta().files().upload(
    FileUploadParams.builder()
        .file(Paths.get("/path/to/doc.pdf"))  // or .file(InputStream) or .file(byte[])
        .build());

// Reference in a beta message:
BetaRequestDocumentBlock doc = BetaRequestDocumentBlock.builder()
    .fileSource(meta.id())
    .build();
```
其他方法：`.list()`、`.delete(String fileId)`、`.download(String fileId)`、`.retrieveMetadata(String fileId)`。
