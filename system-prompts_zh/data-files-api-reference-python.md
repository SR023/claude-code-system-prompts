<!--
name: '数据：文件 API 参考 — Python'
description: Python 文件 API 参考，包括文件上传、列出、删除和消息中的使用
ccVersion: 2.1.78
-->
# 文件 API — Python

文件 API 上传文件以在消息 API 请求中使用。通过 `file_id` 在内容块中引用文件，避免跨多个 API 调用重新上传。

**测试版：** 在 API 调用中传递 `betas=["files-api-2025-04-14"]`（SDK 自动设置所需的标头）。

## 关键事实

- 最大文件大小：500 MB
- 总存储空间：每个组织 100 GB
- 文件持续存在直至被删除
- 文件操作（上传、列出、删除）免费；消息中使用的内容按输入令牌计费
- 不适用于 Amazon Bedrock 或 Google Vertex AI

---

## 上传文件```python
import anthropic

client = anthropic.Anthropic()

uploaded = client.beta.files.upload(
    file=("report.pdf", open("report.pdf", "rb"), "application/pdf"),
)
print(f"File ID: {uploaded.id}")
print(f"Size: {uploaded.size_bytes} bytes")
```
---

## 在消息中使用文件

### PDF/文本文档```python
response = client.beta.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Summarize the key findings in this report."},
            {
                "type": "document",
                "source": {"type": "file", "file_id": uploaded.id},
                "title": "Q4 Report",           # optional
                "citations": {"enabled": True}   # optional, enables citations
            }
        ]
    }],
    betas=["files-api-2025-04-14"],
)
for block in response.content:
    if block.type == "text":
        print(block.text)
```
＃＃＃ 图像```python
image_file = client.beta.files.upload(
    file=("photo.png", open("photo.png", "rb"), "image/png"),
)

response = client.beta.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "What's in this image?"},
            {
                "type": "image",
                "source": {"type": "file", "file_id": image_file.id}
            }
        ]
    }],
    betas=["files-api-2025-04-14"],
)
```
---

## 管理文件

### 列出文件```python
files = client.beta.files.list()
for f in files.data:
    print(f"{f.id}: {f.filename} ({f.size_bytes} bytes)")
```
### 获取文件元数据```python
file_info = client.beta.files.retrieve_metadata("file_011CNha8iCJcU1wXNR6q4V8w")
print(f"Filename: {file_info.filename}")
print(f"MIME type: {file_info.mime_type}")
```
### 删除文件```python
client.beta.files.delete("file_011CNha8iCJcU1wXNR6q4V8w")
```
### 下载文件

只能下载由代码执行工具或技能创建的文件（不能下载用户上传的文件）。```python
file_content = client.beta.files.download("file_011CNha8iCJcU1wXNR6q4V8w")
file_content.write_to_file("output.txt")
```
---

## 完整的端到端示例

上传文档一次，询问多个相关问题：```python
import anthropic

client = anthropic.Anthropic()

# 1. Upload once
uploaded = client.beta.files.upload(
    file=("contract.pdf", open("contract.pdf", "rb"), "application/pdf"),
)
print(f"Uploaded: {uploaded.id}")

# 2. Ask multiple questions using the same file_id
questions = [
    "What are the key terms and conditions?",
    "What is the termination clause?",
    "Summarize the payment schedule.",
]

for question in questions:
    response = client.beta.messages.create(
        model="{{OPUS_ID}}",
        max_tokens=16000,
        messages=[{
            "role": "user",
            "content": [
                {"type": "text", "text": question},
                {
                    "type": "document",
                    "source": {"type": "file", "file_id": uploaded.id}
                }
            ]
        }],
        betas=["files-api-2025-04-14"],
    )
    print(f"\nQ: {question}")
    text = next((b.text for b in response.content if b.type == "text"), "")
    print(f"A: {text[:200]}")

# 3. Clean up when done
client.beta.files.delete(uploaded.id)
```
