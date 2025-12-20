# Ví Dụ Cấu Hình Endpoint

Hướng dẫn hợp nhất nhiều AI provider (Azure AI Foundry, Claude, OpenAI, Gemini) thành một endpoint duy nhất qua ProxyPal.

## Mục Lục

- [Azure AI Foundry (Anthropic)](#azure-ai-foundry-anthropic)
- [Claude API Direct](#claude-api-direct)
- [OpenAI API](#openai-api)
- [Gemini API](#gemini-api)
- [Kết Hợp Nhiều Provider](#kết-hợp-nhiều-provider)
- [Model Mapping](#model-mapping)

---

## Azure AI Foundry (Anthropic)

### Cấu Hình `config.json`

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
          "name": "claude-opus-4-5",
          "alias": "claude-haiku-4-5-20251001"
        }
      ]
    }
  ]
}
```

### Giải Thích

| Field | Giá Trị | Mô Tả |
|-------|---------|-------|
| `apiKey` | `YOUR_AZURE_AI_FOUNDRY_API_KEY` | API key từ Azure Portal |
| `baseUrl` | `https://YOUR-RESOURCE.services.ai.azure.com/anthropic` | Endpoint Foundry |
| `models[].name` | `claude-opus-4-5` | Model thực tế trên Azure |
| `models[].alias` | `claude-opus-4-5-20251101` | Tên model mà client gọi |

### Luồng Hoạt Động

```
Client gọi: claude-sonnet-4-5-20250929
       ↓
ProxyPal nhận request
       ↓
Tìm alias match → route đến Azure Foundry
       ↓
Gọi model thực: claude-opus-4-5
       ↓
Azure AI Foundry xử lý → Response
```

---

## Claude API Direct

### Cấu Hình

```json
{
  "claudeApiKeys": [
    {
      "apiKey": "sk-ant-api03-xxxxx",
      "baseUrl": "https://api.anthropic.com",
      "models": [
        { "name": "claude-opus-4-5-20251101", "alias": "claude-opus-4-5-20251101" },
        { "name": "claude-sonnet-4-5-20250929", "alias": "claude-sonnet-4-5-20250929" }
      ]
    }
  ]
}
```

### Sử Dụng OAuth (Khuyến Nghị)

Thay vì API key, bạn có thể dùng OAuth qua giao diện ProxyPal:
1. Mở ProxyPal → Settings
2. Click "Connect Claude"
3. Đăng nhập tài khoản Claude Pro/Team

---

## OpenAI API

### Cấu Hình

```json
{
  "ampOpenaiProviders": [
    {
      "name": "OpenAI Direct",
      "baseUrl": "https://api.openai.com/v1",
      "apiKey": "sk-xxxxx",
      "models": ["gpt-4o", "gpt-4-turbo", "gpt-3.5-turbo"]
    }
  ]
}
```

### Azure OpenAI

```json
{
  "ampOpenaiProviders": [
    {
      "name": "Azure OpenAI",
      "baseUrl": "https://YOUR-RESOURCE.openai.azure.com/openai/deployments/YOUR-DEPLOYMENT",
      "apiKey": "YOUR_AZURE_OPENAI_KEY",
      "models": ["gpt-4o", "gpt-4"]
    }
  ]
}
```

---

## Gemini API

### OAuth (Khuyến Nghị)

1. Mở ProxyPal → Settings
2. Click "Connect Gemini"
3. Đăng nhập Google Account với Gemini Advanced

### API Key

```json
{
  "geminiApiKeys": [
    {
      "apiKey": "AIzaSyxxxxx",
      "models": ["gemini-2.5-pro", "gemini-2.5-flash"]
    }
  ]
}
```

---

## Kết Hợp Nhiều Provider

### Ví Dụ: Azure Foundry + Claude Direct + Gemini

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
        { "name": "claude-opus-4-5", "alias": "azure-opus" }
      ]
    },
    {
      "apiKey": "sk-ant-api03-direct",
      "baseUrl": "https://api.anthropic.com",
      "models": [
        { "name": "claude-sonnet-4-5-20250929", "alias": "direct-sonnet" }
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
      "name": "Azure OpenAI",
      "baseUrl": "https://openai-resource.openai.azure.com/openai/deployments/gpt4",
      "apiKey": "AZURE_OPENAI_KEY",
      "models": ["gpt-4o"]
    }
  ]
}
```

### Kết Quả

Tất cả được hợp nhất qua **1 endpoint**: `http://localhost:8317/v1`

```bash
# Client chỉ cần biết 1 endpoint
curl http://localhost:8317/v1/chat/completions \
  -H "Authorization: Bearer proxypal-local" \
  -d '{"model": "azure-opus", "messages": [...]}'

curl http://localhost:8317/v1/chat/completions \
  -H "Authorization: Bearer proxypal-local" \
  -d '{"model": "gemini-2.5-pro", "messages": [...]}'
```

---

## Model Mapping

### Cấu Hình Mapping

Route request từ model này sang model khác:

```json
{
  "ampModelMappings": [
    {
      "from": "claude-opus-4-5-20251101",
      "to": "gemini-2.5-pro",
      "enabled": true
    },
    {
      "from": "gpt-4o",
      "to": "claude-sonnet-4-5-20250929",
      "enabled": true
    }
  ],
  "forceModelMappings": false
}
```

### Ví Dụ Sử Dụng

| Client Gọi | ProxyPal Route Đến | Lý Do |
|------------|-------------------|-------|
| `claude-opus-4-5-20251101` | `gemini-2.5-pro` | Tiết kiệm chi phí |
| `gpt-4o` | `claude-sonnet-4-5-20250929` | Prefer Claude |
| `gemini-2.5-flash` | `gemini-2.5-flash` | Không mapping → giữ nguyên |

### Force Mapping

```json
{
  "forceModelMappings": true
}
```

Khi `true`: Mọi request **bắt buộc** phải match mapping, nếu không → reject.

---

## Sử Dụng Từ Coding Tools

### Environment Variables

```bash
# Anthropic-compatible clients (Claude Code, Aider)
export ANTHROPIC_BASE_URL=http://localhost:8317
export ANTHROPIC_API_KEY=proxypal-local

# OpenAI-compatible clients (Cursor, Continue, Cline)
export OPENAI_BASE_URL=http://localhost:8317/v1
export OPENAI_API_KEY=proxypal-local
```

### Server Mode (Chia Sẻ Qua Mạng)

```bash
# Remote machine
export ANTHROPIC_BASE_URL=http://192.168.1.100:8317
export ANTHROPIC_API_KEY=YOUR_REMOTE_API_KEY

# Hoặc qua Tailscale Funnel
export ANTHROPIC_BASE_URL=https://your-pc.tail12345.ts.net
export ANTHROPIC_API_KEY=YOUR_REMOTE_API_KEY
```

---

## Tham Khảo

- [examples/config.example.json](../examples/config.example.json) - Cấu hình mẫu đầy đủ
- [examples/proxy-config.example.yaml](../examples/proxy-config.example.yaml) - YAML config cho CLIProxyAPI
- [README.md](../README.md) - Hướng dẫn tổng quan
