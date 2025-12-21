# Amp Code Integration

Hướng dẫn chi tiết tích hợp Amp Code (Sourcegraph's AI coding assistant) với ProxyPal để route requests đến Azure AI Foundry hoặc các providers khác.

---

## Mục Lục

1. [Giới Thiệu](#1-giới-thiệu)
2. [Cài Đặt Amp Code](#2-cài-đặt-amp-code)
3. [Cấu Hình Amp Code](#3-cấu-hình-amp-code)
4. [Cấu Hình ProxyPal](#4-cấu-hình-proxypal)
5. [Amp Model Mappings](#5-amp-model-mappings)
6. [Workflow Hoạt Động](#6-workflow-hoạt-động)
7. [Ví Dụ Cấu Hình Hoàn Chỉnh](#7-ví-dụ-cấu-hình-hoàn-chỉnh)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Giới Thiệu

### Amp Code Là Gì?

Amp Code là AI coding assistant của Sourcegraph, tương tự Claude Code hoặc Cursor AI. Nó hỗ trợ:

- Anthropic Claude models
- Extended thinking mode
- Tool calling (Bash, file operations)
- Multi-turn conversations

### Tại Sao Dùng ProxyPal?

| Lợi ích | Mô tả |
|---------|-------|
| **Unified endpoint** | Amp chỉ cần 1 endpoint thay vì nhiều |
| **Model routing** | Route requests đến provider tốt nhất |
| **Cost optimization** | Dùng model rẻ cho tasks đơn giản |
| **OAuth sharing** | Dùng Claude Pro subscription thay vì API keys |
| **Azure integration** | Dùng Azure AI Foundry enterprise |

---

## 2. Cài Đặt Amp Code

### Prerequisites

- Node.js 18+
- npm hoặc pnpm

### Cài Đặt

```bash
# npm
npm install -g @anthropics/amp

# pnpm
pnpm add -g @anthropics/amp

# Kiểm tra
amp --version
```

### Vị Trí Config File

| OS | Đường dẫn |
|----|-----------|
| **Windows** | `%USERPROFILE%\.config\amp\settings.json` |
| **macOS** | `~/.config/amp/settings.json` |
| **Linux** | `~/.config/amp/settings.json` |

---

## 3. Cấu Hình Amp Code

### File Settings

Tạo hoặc chỉnh sửa `~/.config/amp/settings.json`:

```json
{
  "amp.url": "http://localhost:8317",
  "amp.apiKey": "proxypal-local",
  "amp.anthropic.thinking.enabled": true,
  "amp.tools.stopTimeout": 600
}
```

### Các Settings Quan Trọng

| Setting | Giá trị | Mô tả |
|---------|---------|-------|
| `amp.url` | `http://localhost:8317` | ProxyPal endpoint |
| `amp.apiKey` | `proxypal-local` | API key |
| `amp.anthropic.thinking.enabled` | `true` | Bật thinking mode |
| `amp.tools.stopTimeout` | `600` | Timeout cho tool calls (giây) |

### Settings Nâng Cao

```json
{
  "amp.url": "http://localhost:8317",
  "amp.apiKey": "proxypal-local",
  "amp.anthropic.thinking.enabled": true,
  "amp.tools.stopTimeout": 600,
  "amp.dangerouslyAllowAll": true,
  "amp.permissions": [
    { "tool": "Bash", "action": "allow" },
    { "tool": "Task", "action": "allow" },
    { "tool": "Read", "action": "allow" },
    { "tool": "Write", "action": "allow" },
    { "tool": "Edit", "action": "allow" },
    { "tool": "*", "action": "allow" }
  ],
  "amp.model": "claude-sonnet-4-5-20250929"
}
```

### Permissions Explained

| Permission | Mô tả |
|------------|-------|
| `Bash` | Chạy shell commands |
| `Task` | Tạo background tasks |
| `Read` | Đọc files |
| `Write` | Ghi files mới |
| `Edit` | Sửa files hiện có |
| `*` | Tất cả tools |

---

## 4. Cấu Hình ProxyPal

### Mở Amp Integration Settings

1. Sidebar → **Settings**
2. Scroll đến section **"Amp Code Integration"**

### Thêm Amp API Key

1. Click **"Add Amp API Key"**
2. Paste key từ [ampcode.com/settings](https://ampcode.com/settings)
3. Save

### Giao Diện

```
┌─────────────────────────────────────────────────────────┐
│  Amp Code Integration                                   │
│                                                         │
│  Amp API Key                                            │
│  [amp-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx      ] 👁    │
│                                                         │
│  Amp Routing Mode                                       │
│  [▾ Mappings                                    ]       │
│    • Mappings - Use Amp Model Mappings                  │
│    • Passthrough - No mapping, use as-is                │
│    • Default - Fallback to default model                │
│                                                         │
│  ═══ Amp Model Mappings ═══                             │
│                                                         │
│  [claude-sonnet-4-5-20250929] → [claude-opus-4-5...] ☑  │
│  [claude-haiku-4-5-20251001 ] → [claude-opus-4-5...] ☑  │
│                                                         │
│  [+ Add Mapping]                                        │
└─────────────────────────────────────────────────────────┘
```

---

## 5. Amp Model Mappings

### Mục Đích

Amp Model Mappings cho phép route requests từ Amp đến model khác. Ví dụ:

- Amp gửi `claude-sonnet-4-5-20250929`
- ProxyPal route đến `claude-opus-4-5-20251101`
- Thực tế gọi Azure AI Foundry với `claude-opus-4-5`

### Cấu Hình Qua UI

1. Settings → Amp Code Integration → Amp Model Mappings
2. Click **"+ Add Mapping"**
3. Điền From và To
4. Enable toggle

### Cấu Hình Config File

```json
{
  "ampApiKey": "amp-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "ampModelMappings": [
    {
      "from": "claude-sonnet-4-5-20250929",
      "to": "claude-opus-4-5-20251101",
      "enabled": true
    },
    {
      "from": "claude-haiku-4-5-20251001",
      "to": "claude-opus-4-5-20251101",
      "enabled": true
    },
    {
      "from": "gemini-2.5-flash",
      "to": "gemini-3-pro-preview",
      "enabled": true
    }
  ],
  "ampRoutingMode": "mappings"
}
```

### Routing Modes

| Mode | Hành vi |
|------|---------|
| `mappings` | Dùng `ampModelMappings` để route |
| `passthrough` | Không mapping, giữ nguyên model |
| `default` | Fallback về model mặc định |

---

## 6. Workflow Hoạt Động

### Luồng Request

```
┌─────────────────────────────────────────────────────────┐
│  Amp Code                                               │
│  Request: claude-sonnet-4-5-20250929                    │
└──────────────────────┬──────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────┐
│  ProxyPal (localhost:8317)                              │
│                                                         │
│  1. Nhận request từ Amp                                 │
│     Model: claude-sonnet-4-5-20250929                   │
│                                                         │
│  2. Check Amp Model Mappings                            │
│     Rule: sonnet → opus ✓                               │
│     New model: claude-opus-4-5-20251101                 │
│                                                         │
│  3. Check Claude API Keys aliasing                      │
│     Alias: claude-opus-4-5-20251101 → claude-opus-4-5   │
│     Provider: Azure AI Foundry                          │
│                                                         │
│  4. Route to Azure AI Foundry                           │
│     Model: claude-opus-4-5                              │
└──────────────────────┬──────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────┐
│  Azure AI Foundry                                       │
│  Deployment: claude-opus-4-5                            │
│  → Process request                                      │
│  → Return response                                      │
└─────────────────────────────────────────────────────────┘
```

### Diagram Chi Tiết

```
Amp Code
    │
    │ POST /v1/messages
    │ model: claude-sonnet-4-5-20250929
    ▼
ProxyPal
    │
    ├─► Amp Model Mappings
    │   sonnet → opus
    │
    ├─► Claude API Keys
    │   opus-20251101 → opus-4-5 (Azure)
    │
    ├─► Route to Azure AI Foundry
    │   https://resource.azure.com/anthropic
    │
    ▼
Azure AI Foundry
    │
    │ model: claude-opus-4-5
    ▼
Response → ProxyPal → Amp Code
```

---

## 7. Ví Dụ Cấu Hình Hoàn Chỉnh

### Amp Settings (`~/.config/amp/settings.json`)

```json
{
  "amp.url": "http://localhost:8317",
  "amp.apiKey": "proxypal-local",
  "amp.anthropic.thinking.enabled": true,
  "amp.tools.stopTimeout": 600,
  "amp.dangerouslyAllowAll": true,
  "amp.permissions": [
    { "tool": "Bash", "action": "allow" },
    { "tool": "Task", "action": "allow" },
    { "tool": "Read", "action": "allow" },
    { "tool": "Write", "action": "allow" },
    { "tool": "Edit", "action": "allow" },
    { "tool": "*", "action": "allow" }
  ]
}
```

### ProxyPal Config (`%APPDATA%\proxypal\config.json`)

```json
{
  "port": 8317,
  "autoStart": true,

  "ampApiKey": "amp-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "ampModelMappings": [
    { "from": "claude-sonnet-4-5-20250929", "to": "claude-opus-4-5-20251101", "enabled": true },
    { "from": "claude-haiku-4-5-20251001", "to": "claude-opus-4-5-20251101", "enabled": true }
  ],
  "ampRoutingMode": "mappings",

  "claudeApiKeys": [
    {
      "apiKey": "YOUR_AZURE_API_KEY",
      "baseUrl": "https://your-resource.services.ai.azure.com/anthropic",
      "headers": {
        "x-api-key": "YOUR_AZURE_API_KEY",
        "anthropic-version": "2023-06-01"
      },
      "models": [
        { "name": "claude-opus-4-5", "alias": "claude-opus-4-5-20251101" },
        { "name": "claude-opus-4-5", "alias": "claude-sonnet-4-5-20250929" },
        { "name": "claude-haiku-4-5", "alias": "claude-haiku-4-5-20251001" }
      ]
    }
  ],

  "thinkingBudgetMode": "custom",
  "thinkingBudgetCustom": 16000,
  "reasoningEffortLevel": "xhigh"
}
```

### Kết Quả

Khi chạy `amp`:

1. Amp gửi request với `claude-sonnet-4-5-20250929`
2. ProxyPal mapping sang `claude-opus-4-5-20251101`
3. Azure AI Foundry nhận với `claude-opus-4-5`
4. Response trả về cho Amp

---

## 8. Troubleshooting

### Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Giải pháp |
|-----|-------------|-----------|
| `Connection refused` | ProxyPal không chạy | Start proxy |
| `401 Unauthorized` | API key sai | Kiểm tra `amp.apiKey` |
| `Model not found` | Mapping không đúng | Kiểm tra Amp Model Mappings |
| `no auth available` | Không có provider | Connect ít nhất 1 provider |
| `Thinking timeout` | Budget quá thấp | Tăng `thinkingBudgetCustom` |

### Debug Steps

#### Step 1: Kiểm tra ProxyPal

```bash
# Test endpoint
curl http://localhost:8317/v1/models \
  -H "Authorization: Bearer proxypal-local"
```

#### Step 2: Kiểm tra Amp Config

```bash
# Xem config
cat ~/.config/amp/settings.json

# Kiểm tra amp.url và amp.apiKey
```

#### Step 3: Bật Debug Mode

ProxyPal Settings → Enable **"Debug Mode"**

Logs page → Filter **"DEBUG"** → Xem:

```
[DEBUG] Amp request received
[DEBUG] Model mapping: claude-sonnet-4-5-20250929 → claude-opus-4-5-20251101
[DEBUG] Routing to Azure AI Foundry
[DEBUG] Alias: claude-opus-4-5-20251101 → claude-opus-4-5
```

#### Step 4: Test Amp

```bash
cd /path/to/project
amp

# Trong amp:
> /help
> Hello, can you see me?
```

### Thinking Mode Không Hoạt Động

**Nguyên nhân:** Budget không đủ hoặc model không hỗ trợ.

**Giải pháp:**

1. Settings → `thinkingBudgetMode: "custom"`
2. Settings → `thinkingBudgetCustom: 16000` (hoặc cao hơn)
3. Đảm bảo dùng model hỗ trợ thinking (Opus, Sonnet)

### Model Route Sai

**Nguyên nhân:** Amp Model Mappings conflict với Claude API Keys aliasing.

**Giải pháp:**

1. Kiểm tra thứ tự ưu tiên:
   - Amp Model Mappings (applied first)
   - Claude API Keys → models[] (applied second)
2. Đảm bảo không có circular mapping

---

## Tham Khảo

- [Amp Code Documentation](https://ampcode.com/docs)
- [Custom Providers](./CUSTOM_PROVIDERS.md) - Azure AI Foundry
- [Model Mapping](./MODEL_MAPPING.md) - Mapping chi tiết
- [Ví Dụ Endpoint](./VI_DU_ENDPOINT.md) - Cấu hình mẫu
