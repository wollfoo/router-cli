# Custom Providers

Hướng dẫn chi tiết thêm Custom OpenAI-Compatible Providers và Azure AI Foundry (Anthropic API) vào ProxyPal.

---

## Mục Lục

1. [Tổng Quan](#1-tổng-quan)
2. [Custom OpenAI-Compatible Providers](#2-custom-openai-compatible-providers)
3. [Azure AI Foundry (Anthropic)](#3-azure-ai-foundry-anthropic)
4. [Amazon Bedrock](#4-amazon-bedrock)
5. [Self-Hosted Providers](#5-self-hosted-providers)
6. [Cấu Hình Config File](#6-cấu-hình-config-file)
7. [Test Endpoints](#7-test-endpoints)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Tổng Quan

### Hai Loại Custom Providers

| Loại | API Format | Ví dụ |
|------|------------|-------|
| **OpenAI-Compatible** | `/v1/chat/completions` | OpenRouter, Together, Groq |
| **Anthropic-Compatible** | `/v1/messages` | Azure AI Foundry, Bedrock |

### Khi Nào Dùng

- Có API key riêng từ provider
- Muốn dùng model không có trong OAuth providers
- Cần custom endpoint (enterprise, self-hosted)
- Kết hợp nhiều nguồn AI

---

## 2. Custom OpenAI-Compatible Providers

### Cấu Hình Qua UI

1. Sidebar → **API Keys**
2. Tab → **"OpenAI Compatible"**
3. Click **"Add Provider"**

### Điền Thông Tin

| Field | Mô tả | Bắt buộc |
|-------|-------|----------|
| **Name** | Tên hiển thị (VD: "OpenRouter") | ✓ |
| **Base URL** | Endpoint API | ✓ |
| **API Key** | API key của bạn | ✓ |
| **Models** | Danh sách models (1 model/dòng) | ✓ |

### Ví Dụ Giao Diện

```
┌─────────────────────────────────────────────────────────┐
│  Add OpenAI-Compatible Provider                         │
│                                                         │
│  Name *                                                 │
│  [OpenRouter                                    ]       │
│                                                         │
│  Base URL *                                             │
│  [https://openrouter.ai/api/v1                  ]       │
│                                                         │
│  API Key *                                              │
│  [sk-or-v1-xxxxxxxxxxxxxxxxxxxx                 ] 👁    │
│                                                         │
│  Models (one per line) *                                │
│  ┌─────────────────────────────────────────────┐        │
│  │ openai/gpt-4-turbo                          │        │
│  │ anthropic/claude-3-opus                     │        │
│  │ google/gemini-pro                           │        │
│  │ meta-llama/llama-3-70b-instruct            │        │
│  └─────────────────────────────────────────────┘        │
│                                                         │
│  [Cancel]                          [Add Provider]       │
└─────────────────────────────────────────────────────────┘
```

### Providers Phổ Biến

| Provider | Base URL | API Key |
|----------|----------|---------|
| **OpenRouter** | `https://openrouter.ai/api/v1` | `sk-or-v1-xxx` |
| **Together AI** | `https://api.together.xyz/v1` | `xxx` |
| **Groq** | `https://api.groq.com/openai/v1` | `gsk_xxx` |
| **Fireworks** | `https://api.fireworks.ai/inference/v1` | `fw_xxx` |
| **DeepSeek** | `https://api.deepseek.com/v1` | `sk-xxx` |
| **Mistral** | `https://api.mistral.ai/v1` | `xxx` |
| **Perplexity** | `https://api.perplexity.ai` | `pplx-xxx` |
| **Anyscale** | `https://api.endpoints.anyscale.com/v1` | `xxx` |
| **LiteLLM** | `http://localhost:4000` | (tùy config) |

### OpenRouter Ví Dụ Chi Tiết

```json
{
  "ampOpenaiProviders": [
    {
      "name": "OpenRouter",
      "baseUrl": "https://openrouter.ai/api/v1",
      "apiKey": "sk-or-v1-your-api-key-here",
      "models": [
        "openai/gpt-4-turbo",
        "openai/gpt-4o",
        "anthropic/claude-3-opus",
        "anthropic/claude-3-sonnet",
        "google/gemini-pro-1.5",
        "meta-llama/llama-3.1-405b-instruct"
      ]
    }
  ]
}
```

---

## 3. Azure AI Foundry (Anthropic)

### Tổng Quan

Azure AI Foundry cung cấp Claude models qua Anthropic API format (không phải OpenAI format).

### Cấu Trúc ClaudeApiKey

```typescript
interface ClaudeApiKey {
  apiKey: string;                      // API key từ Azure
  baseUrl?: string;                    // Custom endpoint
  proxyUrl?: string;                   // Proxy nếu cần
  headers?: Record<string, string>;    // Custom headers
  models?: ModelMapping[];             // Model aliasing
}

interface ModelMapping {
  name: string;   // Tên model trên Azure (VD: "claude-opus-4-5")
  alias: string;  // Tên model khi request (VD: "claude-opus-4-5-20251101")
}
```

### Cấu Hình Qua UI

#### Bước 1: Mở API Keys

- Sidebar → **API Keys**
- Tab → **Claude** (không phải OpenAI Compatible)

#### Bước 2: Click "Add Claude API Key"

**Lưu ý:** Proxy phải đang chạy.

#### Bước 3: Điền Thông Tin

| Field | Value | Lưu ý |
|-------|-------|-------|
| **API Key** | API key từ Azure Portal | Bắt buộc |
| **Base URL** | `https://{RESOURCE}.services.ai.azure.com/anthropic` | KHÔNG có `/v1` |

#### Bước 4: Thêm Model Aliasing

**Quan trọng:** Azure deployment names khác Anthropic standard names.

| Alias (client gọi) | Name (trên Azure) |
|--------------------|-------------------|
| `claude-opus-4-5-20251101` | `claude-opus-4-5` |
| `claude-sonnet-4-5-20250929` | `claude-sonnet-4-5` |
| `claude-haiku-4-5-20251001` | `claude-haiku-4-5` |

#### Bước 5: Save

Click **"Add Key"**.

### Giao Diện UI

```
┌─────────────────────────────────────────────────────────┐
│  Add Claude API Key                                     │
│                                                         │
│  API Key *                                              │
│  [your-azure-api-key-here...                    ] 👁    │
│                                                         │
│  Base URL (optional)                                    │
│  [https://resource.services.ai.azure.com/       ]       │
│  [anthropic                                     ]       │
│                                                         │
│  Proxy URL (optional)                                   │
│  [                                              ]       │
│                                                         │
│  Models (aliasing)                                      │
│  ┌─────────────────────────────────────────────┐        │
│  │ Name: claude-opus-4-5                        │        │
│  │ Alias: claude-opus-4-5-20251101             │ [🗑]   │
│  ├─────────────────────────────────────────────┤        │
│  │ Name: claude-sonnet-4-5                      │        │
│  │ Alias: claude-sonnet-4-5-20250929           │ [🗑]   │
│  └─────────────────────────────────────────────┘        │
│  [+ Add Model Alias]                                    │
│                                                         │
│  [Cancel]                              [Add Key]        │
└─────────────────────────────────────────────────────────┘
```

### Cấu Hình Config File

```json
{
  "claudeApiKeys": [
    {
      "apiKey": "YOUR_AZURE_AI_FOUNDRY_API_KEY",
      "baseUrl": "https://YOUR-RESOURCE.services.ai.azure.com/anthropic",
      "headers": {
        "x-api-key": "YOUR_AZURE_AI_FOUNDRY_API_KEY",
        "anthropic-version": "2023-06-01"
      },
      "models": [
        {
          "name": "claude-opus-4-5",
          "alias": "claude-opus-4-5-20251101"
        },
        {
          "name": "claude-opus-4-5",
          "alias": "claude-sonnet-4-5-20250929"
        },
        {
          "name": "claude-haiku-4-5",
          "alias": "claude-haiku-4-5-20251001"
        }
      ]
    }
  ]
}
```

### Chuyển Đổi Từ Environment Variables

Nếu bạn đang dùng Claude Code với Azure:

```bash
# Environment variables hiện tại
export CLAUDE_CODE_USE_FOUNDRY=1
export ANTHROPIC_FOUNDRY_RESOURCE="your-resource-name"
export ANTHROPIC_FOUNDRY_API_KEY="your-api-key"
export ANTHROPIC_DEFAULT_OPUS_MODEL="claude-opus-4-5"
```

**Chuyển sang ProxyPal:**

| Azure Env | ProxyPal Field |
|-----------|----------------|
| `ANTHROPIC_FOUNDRY_API_KEY` | **API Key** |
| `https://{RESOURCE}.services.ai.azure.com/anthropic` | **Base URL** |
| `ANTHROPIC_DEFAULT_*_MODEL` | **Models aliasing** |

---

## 4. Amazon Bedrock

### Cấu Hình

Amazon Bedrock dùng Anthropic API format với authentication khác:

```json
{
  "claudeApiKeys": [
    {
      "apiKey": "your-aws-secret-access-key",
      "baseUrl": "https://bedrock-runtime.us-east-1.amazonaws.com",
      "headers": {
        "x-amz-access-key": "YOUR_ACCESS_KEY_ID",
        "x-amz-secret-key": "YOUR_SECRET_ACCESS_KEY",
        "x-amz-region": "us-east-1"
      },
      "models": [
        {
          "name": "anthropic.claude-3-opus-20240229-v1:0",
          "alias": "claude-opus-4-5-20251101"
        },
        {
          "name": "anthropic.claude-3-sonnet-20240229-v1:0",
          "alias": "claude-sonnet-4-5-20250929"
        }
      ]
    }
  ]
}
```

**Lưu ý:** Bedrock model IDs có format khác.

---

## 5. Self-Hosted Providers

### LiteLLM Proxy

```json
{
  "ampOpenaiProviders": [
    {
      "name": "LiteLLM Local",
      "baseUrl": "http://localhost:4000",
      "apiKey": "sk-1234",
      "models": ["gpt-4", "claude-3-opus", "gemini-pro"]
    }
  ]
}
```

### Ollama

```json
{
  "ampOpenaiProviders": [
    {
      "name": "Ollama",
      "baseUrl": "http://localhost:11434/v1",
      "apiKey": "ollama",
      "models": ["llama3", "codellama", "mistral"]
    }
  ]
}
```

### vLLM

```json
{
  "ampOpenaiProviders": [
    {
      "name": "vLLM Server",
      "baseUrl": "http://localhost:8000/v1",
      "apiKey": "token-abc123",
      "models": ["meta-llama/Llama-3-70b-chat-hf"]
    }
  ]
}
```

### LocalAI

```json
{
  "ampOpenaiProviders": [
    {
      "name": "LocalAI",
      "baseUrl": "http://localhost:8080/v1",
      "apiKey": "any",
      "models": ["gpt-3.5-turbo", "text-embedding-ada-002"]
    }
  ]
}
```

---

## 6. Cấu Hình Config File

### File Location

| OS | Path |
|----|------|
| **Windows** | `%APPDATA%\proxypal\config.json` |
| **macOS** | `~/Library/Application Support/proxypal/config.json` |
| **Linux** | `~/.config/proxypal/config.json` |

### Cấu Hình Hoàn Chỉnh

```json
{
  "port": 8317,

  "claudeApiKeys": [
    {
      "apiKey": "AZURE_FOUNDRY_KEY",
      "baseUrl": "https://foundry-resource.services.ai.azure.com/anthropic",
      "headers": {
        "x-api-key": "AZURE_FOUNDRY_KEY",
        "anthropic-version": "2023-06-01"
      },
      "models": [
        { "name": "claude-opus-4-5", "alias": "claude-opus-4-5-20251101" },
        { "name": "claude-sonnet-4-5", "alias": "claude-sonnet-4-5-20250929" }
      ]
    },
    {
      "apiKey": "sk-ant-api03-direct-key",
      "baseUrl": "https://api.anthropic.com",
      "models": [
        { "name": "claude-3-haiku-20240307", "alias": "haiku-fast" }
      ]
    }
  ],

  "geminiApiKeys": [
    {
      "apiKey": "AIzaSyxxxxx",
      "models": ["gemini-2.5-pro", "gemini-2.5-flash"]
    }
  ],

  "ampOpenaiProviders": [
    {
      "name": "OpenRouter",
      "baseUrl": "https://openrouter.ai/api/v1",
      "apiKey": "sk-or-v1-xxx",
      "models": ["openai/gpt-4-turbo", "anthropic/claude-3-opus"]
    },
    {
      "name": "Groq",
      "baseUrl": "https://api.groq.com/openai/v1",
      "apiKey": "gsk_xxx",
      "models": ["llama3-70b-8192", "mixtral-8x7b-32768"]
    },
    {
      "name": "Local Ollama",
      "baseUrl": "http://localhost:11434/v1",
      "apiKey": "ollama",
      "models": ["llama3", "codellama"]
    }
  ]
}
```

---

## 7. Test Endpoints

### Test Azure AI Foundry Trực Tiếp

```powershell
curl -X POST "https://your-resource.services.ai.azure.com/anthropic/v1/messages" `
  -H "Content-Type: application/json" `
  -H "x-api-key: YOUR_API_KEY" `
  -H "anthropic-version: 2023-06-01" `
  -d '{
    "model": "claude-opus-4-5",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### Test Qua ProxyPal

```powershell
# Anthropic format
curl http://localhost:8317/v1/messages `
  -H "Content-Type: application/json" `
  -H "x-api-key: proxypal-local" `
  -H "anthropic-version: 2023-06-01" `
  -d '{
    "model": "claude-opus-4-5-20251101",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

```bash
# OpenAI format (cho custom providers)
curl http://localhost:8317/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer proxypal-local" \
  -d '{
    "model": "openai/gpt-4-turbo",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### Liệt Kê Models

```bash
curl http://localhost:8317/v1/models \
  -H "Authorization: Bearer proxypal-local" | jq '.data[].id'
```

---

## 8. Troubleshooting

### Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Giải pháp |
|-----|-------------|-----------|
| `401 Unauthorized` | API key sai | Kiểm tra lại key |
| `404 Not Found` | Base URL sai | Kiểm tra URL format |
| `DeploymentNotFound` | Model name không khớp | Thêm model aliasing |
| `no auth available` | Không có provider nào configured | Thêm ít nhất 1 provider/key |
| `Invalid model` | Model không tồn tại | Kiểm tra danh sách models |

### Azure AI Foundry Lỗi

#### "DeploymentNotFound"

**Nguyên nhân:** Model name trong request không khớp deployment name trên Azure.

**Giải pháp:**

1. Vào Azure Portal → AI Foundry → Deployments
2. Xem deployment name thực tế (VD: `claude-opus-4-5`)
3. Thêm model aliasing trong ProxyPal

#### "InvalidApiKey"

**Nguyên nhân:** API key không đúng hoặc hết hạn.

**Giải pháp:**

1. Azure Portal → AI Services → Keys and Endpoint
2. Copy Key 1 hoặc Key 2
3. Update trong ProxyPal

### Custom Provider Lỗi

#### "Model not in list"

**Nguyên nhân:** Model ID không khớp với list đã cấu hình.

**Giải pháp:**

1. Kiểm tra provider documentation cho đúng model IDs
2. Cập nhật danh sách models trong config

#### "Connection refused"

**Nguyên nhân:** Self-hosted server không chạy.

**Giải pháp:**

1. Kiểm tra server (Ollama, LiteLLM, etc.) đang chạy
2. Kiểm tra port đúng
3. Kiểm tra firewall

### Debug

1. Settings → Enable **"Debug Mode"**
2. Settings → Enable **"Request Logging"**
3. Logs page → Filter **"DEBUG"**
4. Xem chi tiết requests/responses

---

## Tham Khảo

- [Model Mapping](./MODEL_MAPPING.md) - Route giữa các models
- [Ví Dụ Endpoint](./VI_DU_ENDPOINT.md) - Cấu hình mẫu đầy đủ
- [Amp Code](./AMP_CODE.md) - Integration với Amp
- [Xử Lý Lỗi](./TROUBLESHOOTING.md)
