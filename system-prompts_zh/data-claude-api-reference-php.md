<!--
name: '数据：Claude API 参考 — PHP'
description: PHP SDK 参考
ccVersion: 2.1.83
-->
# 克劳德 API — PHP

> **注意：** PHP SDK 是 PHP 的官方 Anthropic SDK。 Beta 工具运行程序可通过 `$client->beta->messages->toolRunner()` 获得。通过 `StructuredOutputModel` 类支持结构化输出助手。代理 SDK 不可用。支持 Bedrock、Vertex AI 和 Foundry 客户端。

＃＃ 安装```bash
composer require "anthropic-ai/sdk"
```
## 客户端初始化```php
use Anthropic\Client;

// Using API key from environment variable
$client = new Client(apiKey: getenv("ANTHROPIC_API_KEY"));
```
### 亚马逊基岩```php
use Anthropic\Bedrock;

// Constructor is private — use the static factory. Reads AWS credentials from env.
$client = Bedrock\Client::fromEnvironment(region: 'us-east-1');
```
### 谷歌顶点人工智能```php
use Anthropic\Vertex;

// Constructor is private. Parameter is `location`, not `region`.
$client = Vertex\Client::fromEnvironment(
    location: 'us-east5',
    projectId: 'my-project-id',
);
```
### 人类铸造厂```php
use Anthropic\Foundry;

// Constructor is private. baseUrl or resource is required.
$client = Foundry\Client::withCredentials(
    authToken: getenv('ANTHROPIC_FOUNDRY_AUTH_TOKEN'),
    baseUrl: 'https://<resource>.services.ai.azure.com/anthropic',
);
```
---

## 基本消息请求```php
$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    messages: [
        ['role' => 'user', 'content' => 'What is the capital of France?'],
    ],
);

// content is an array of polymorphic blocks (TextBlock, ToolUseBlock,
// ThinkingBlock). Accessing ->text on content[0] without checking the block
// type will throw if the first block is not a TextBlock (e.g., when extended
// thinking is enabled and a ThinkingBlock comes first). Always guard:
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
    }
}
```
如果您只想要第一个文本块：```php
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
        break;
    }
}
```
---

## 流媒体

> **需要 SDK v0.5.0+。** v0.4.0 及更早版本使用单个 `$params` 数组；使用命名参数调用会抛出 `Unknown named parameter $model`。升级：`composer require "anthropic-ai/sdk:^0.7"````php
use Anthropic\Messages\RawContentBlockDeltaEvent;
use Anthropic\Messages\TextDelta;

$stream = $client->messages->createStream(
    model: '{{OPUS_ID}}',
    maxTokens: 64000,
    messages: [
        ['role' => 'user', 'content' => 'Write a haiku'],
    ],
);

foreach ($stream as $event) {
    if ($event instanceof RawContentBlockDeltaEvent && $event->delta instanceof TextDelta) {
        echo $event->delta->text;
    }
}
```
---

## 工具使用

### 工具运行程序（测试版）

**测试版：** PHP SDK 通过 `$client->beta->messages->toolRunner()` 提供工具运行程序。使用 `BetaRunnableTool` 定义工具 — 定义数组加上 `run` 闭包：```php
use Anthropic\Lib\Tools\BetaRunnableTool;

$weatherTool = new BetaRunnableTool(
    definition: [
        'name' => 'get_weather',
        'description' => 'Get the current weather for a location.',
        'input_schema' => [
            'type' => 'object',
            'properties' => [
                'location' => ['type' => 'string', 'description' => 'City and state'],
            ],
            'required' => ['location'],
        ],
    ],
    run: function (array $input): string {
        return "The weather in {$input['location']} is sunny and 72°F.";
    },
);

$runner = $client->beta->messages->toolRunner(
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'What is the weather in Paris?']],
    model: '{{OPUS_ID}}',
    tools: [$weatherTool],
);

foreach ($runner as $message) {
    foreach ($message->content as $block) {
        if ($block->type === 'text') {
            echo $block->text;
        }
    }
}
```
### 手动循环

工具作为数组传递。 **SDK 使用驼峰命名法键**（`inputSchema`、`toolUseID`、`stopReason`）并自动映射到 API 上的 Snake_case — 从 v0.5.0 开始。有关循环模式，请参阅[共享工具使用概念](../shared/tool-use-concepts.md)。```php
use Anthropic\Messages\ToolUseBlock;

$tools = [
    [
        'name' => 'get_weather',
        'description' => 'Get the current weather in a given location',
        'inputSchema' => [  // camelCase, not input_schema
            'type' => 'object',
            'properties' => [
                'location' => ['type' => 'string', 'description' => 'City and state'],
            ],
            'required' => ['location'],
        ],
    ],
];

$messages = [['role' => 'user', 'content' => 'What is the weather in SF?']];

$response = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    tools: $tools,
    messages: $messages,
);

while ($response->stopReason === 'tool_use') {  // camelCase property
    $toolResults = [];
    foreach ($response->content as $block) {
        if ($block instanceof ToolUseBlock) {
            // $block->name  : string               — tool name to dispatch on
            // $block->input : array<string,mixed>  — parsed JSON input
            // $block->id    : string               — pass back as toolUseID
            $result = executeYourTool($block->name, $block->input);
            $toolResults[] = [
                'type' => 'tool_result',
                'toolUseID' => $block->id,  // camelCase, not tool_use_id
                'content' => $result,
            ];
        }
    }

    // Append assistant turn + user turn with tool results
    $messages[] = ['role' => 'assistant', 'content' => $response->content];
    $messages[] = ['role' => 'user', 'content' => $toolResults];

    $response = $client->messages->create(
        model: '{{OPUS_ID}}',
        maxTokens: 16000,
        tools: $tools,
        messages: $messages,
    );
}

// Final text response
foreach ($response->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
    }
}
```
`$block->type === 'tool_use'` 也可以； `instanceof ToolUseBlock` 缩小为 PHPStan。


---

## 延伸思考

**自适应思维是 Claude 4.6+ 模型的推荐模式。** Claude 动态决定思考的时间和程度。```php
use Anthropic\Messages\ThinkingBlock;

$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    thinking: ['type' => 'adaptive'],
    messages: [
        ['role' => 'user', 'content' => 'Solve: 27 * 453'],
    ],
);

// ThinkingBlock(s) precede TextBlock in content
foreach ($message->content as $block) {
    if ($block instanceof ThinkingBlock) {
        echo "Thinking:\n{$block->thinking}\n\n";
        // $block->signature is an opaque string — preserve verbatim if
        // passing thinking blocks back in multi-turn conversations
    } elseif ($block->type === 'text') {
        echo "Answer: {$block->text}\n";
    }
}
```
> **已弃用：** `['type' => 'enabled', 'budgetTokens' => N]`（固定预算扩展思维）仍然适用于 Claude 4.6，但已弃用。使用上面的适应性思维。

`$block->type === 'thinking'` 也适用于检查； `instanceof` 缩小为 PHPStan。

---

## 提示缓存

`system:` 采用文本块数组；在最后一个块上设置 `cacheControl`。数组形状语法（驼峰式键）是惯用的。有关放置模式和静默无效器审核清单，请参阅 `shared/prompt-caching.md`。```php
$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    system: [
        ['type' => 'text', 'text' => $longSystemPrompt, 'cacheControl' => ['type' => 'ephemeral']],
    ],
    messages: [['role' => 'user', 'content' => 'Summarize the key points']],
);
```
对于 1 小时 TTL：`'cacheControl' => ['type' => 'ephemeral', 'ttl' => '1h']`。 `messages->create(...)` 上还有一个顶级 `cacheControl:`，它自动放置在最后一个可缓存块上。

通过 `$message->usage->cacheCreationInputTokens` / `$message->usage->cacheReadInputTokens` 验证命中。

---

## 结构化输出

### 使用 StructuredOutputModel（推荐）

定义一个实现 `StructuredOutputModel` 的 PHP 类并将其作为 `outputConfig` 传递：```php
use Anthropic\Lib\Contracts\StructuredOutputModel;
use Anthropic\Lib\Concerns\StructuredOutputModelTrait;
use Anthropic\Lib\Attributes\Constrained;

class Person implements StructuredOutputModel
{
    use StructuredOutputModelTrait;

    #[Constrained(description: 'Full name')]
    public string $name;

    public int $age;

    public ?string $email = null;  // nullable = optional field
}

$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'Generate a profile for Alice, age 30']],
    outputConfig: ['format' => Person::class],
);

$person = $message->parsedOutput();  // Person instance
echo $person->name;
```
类型是根据 PHP 类型提示推断的。使用 `#[Constrained(description: '...')]` 添加描述。可空属性 (`?string`) 成为可选字段。

### 原始模式```php
$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'Extract: John (john@co.com), Enterprise plan']],
    outputConfig: [
        'format' => [
            'type' => 'json_schema',
            'schema' => [
                'type' => 'object',
                'properties' => [
                    'name' => ['type' => 'string'],
                    'email' => ['type' => 'string'],
                    'plan' => ['type' => 'string'],
                ],
                'required' => ['name', 'email', 'plan'],
                'additionalProperties' => false,
            ],
        ],
    ],
);

// First text block contains valid JSON
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        $data = json_decode($block->text, true);
        break;
    }
}
```
---

## Beta 功能和服务器端工具

**`betas:` 不是 `$client->messages->create()` 上的参数** — 它仅存在于 beta 命名空间中。将其用于需要显式选择加入标头的功能：```php
use Anthropic\Beta\Messages\BetaRequestMCPServerURLDefinition;

$response = $client->beta->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    mcpServers: [
        BetaRequestMCPServerURLDefinition::with(
            name: 'my-server',
            url: 'https://example.com/mcp',
        ),
    ],
    betas: ['mcp-client-2025-11-20'],  // only valid on ->beta->messages
    messages: [['role' => 'user', 'content' => 'Use the MCP tools']],
);
```
**服务器端工具**（bash、web_search、text_editor、code_execution）是 GA，可在两条路径上工作 - `Anthropic\Messages\ToolBash20250124` / `WebSearchTool20260209` / `ToolTextEditor20250728` / `CodeExecutionTool20260120`（非测试版）、`Anthropic\Beta\Messages\BetaToolBash20250124` / `BetaWebSearchTool20260209` / `BetaToolTextEditor20250728` / `BetaCodeExecutionTool20260120` 测试版。这些不需要 `betas:` 标头。
