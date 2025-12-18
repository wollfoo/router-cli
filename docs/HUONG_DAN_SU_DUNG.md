# Hướng dẫn sử dụng ProxyPal

> ProxyPal - Sử dụng AI subscriptions (Claude, ChatGPT, Gemini, GitHub Copilot) với bất kỳ coding tool nào.

---

## Mục lục

1. [Cài đặt](#1-cài-đặt)
2. [Khởi động và kết nối](#2-khởi-động-và-kết-nối)
3. [Cấu hình Coding Tools](#3-cấu-hình-coding-tools)
4. [Các trang chính](#4-các-trang-chính)
5. [Model Mapping](#5-model-mapping)
6. [Custom OpenAI-Compatible Providers](#6-custom-openai-compatible-providers)
7. [Custom Anthropic API (Azure AI Foundry)](#7-custom-anthropic-api-azure-ai-foundry)
8. [Amp Code Integration](#8-amp-code-integration)
9. [Keyboard Shortcuts](#9-keyboard-shortcuts)
10. [Troubleshooting Windows](#10-troubleshooting-windows)
11. [Config Files](#11-config-files)

---

## 1. Cài đặt

### Cách 1: Tải bản build sẵn

1. Vào https://github.com/nicepkg/proxypal/releases
2. Tải file `.msi` hoặc `.exe` cho Windows x64
3. Chạy installer và cài đặt

### Cách 2: Build từ source

```powershell
# Clone repo
git clone https://github.com/nicepkg/proxypal.git
cd proxypal

# Cài dependencies
pnpm install

# Chạy development mode
pnpm tauri dev

# Hoặc build production
pnpm tauri build
```

---

## 2. Khởi động và kết nối

### Bước 1: Mở ProxyPal

- Chạy app từ Start Menu hoặc Desktop shortcut
- App sẽ hiện **Welcome page** nếu lần đầu sử dụng

### Bước 2: Kết nối AI Provider

Chọn provider cần kết nối:

| Provider | Cách kết nối |
|----------|--------------|
| **Claude** | Click "Connect" → OAuth login với Anthropic |
| **ChatGPT/OpenAI** | Click "Connect" → OAuth login với OpenAI |
| **Gemini** | Click "Connect" → OAuth login với Google |
| **Qwen** | Click "Connect" → OAuth login |
| **GitHub Copilot** | Settings → Enable Copilot → Auth với GitHub |

### Bước 3: Start Proxy

- Proxy sẽ **tự động start** nếu `Auto Start` được bật
- Hoặc click nút **Start Proxy** trên Dashboard
- Endpoint mặc định: `http://localhost:8317/v1`

---

## 3. Cấu hình Coding Tools

### Cursor

```
Settings → Models → OpenAI API Base URL
URL: http://localhost:8317/v1
API Key: anything (hoặc để trống)
```

### Continue (VS Code)

```json
// ~/.continue/config.json
{
  "models": [{
    "provider": "openai",
    "model": "claude-sonnet-4-20250514",
    "apiBase": "http://localhost:8317/v1",
    "apiKey": "dummy"
  }]
}
```

### Claude Code CLI

```powershell
# Set environment variable (session)
$env:ANTHROPIC_BASE_URL = "http://localhost:8317"

# Set permanent (User level)
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "http://localhost:8317", "User")
```

### Cline (VS Code)

```
Settings → API Provider → OpenAI Compatible
Base URL: http://localhost:8317/v1
```

### OpenCode

```powershell
$env:OPENAI_BASE_URL = "http://localhost:8317/v1"
```

---

## 4. Các trang chính

| Trang | Chức năng |
|-------|-----------|
| **Dashboard** | Tổng quan, start/stop proxy, providers status |
| **Settings** | Cấu hình port, auto-start, debug mode, model mappings |
| **API Keys** | Quản lý API keys cho các provider (Gemini, Claude, Codex, OpenAI) |
| **Auth Files** | Xem các file xác thực OAuth |
| **Logs** | Xem request logs real-time |
| **Analytics** | Thống kê usage, tokens, costs |

---

## 5. Model Mapping

### Mục đích

Chuyển hướng request từ model A sang model B.

### Cách cấu hình

1. Vào **Settings** → Section **"Model Mappings"**
2. Có 2 loại:
   - **Predefined slots**: Các mapping có sẵn
   - **Custom mappings**: Tự thêm mapping bất kỳ

### Ví dụ

| From (Request) | To (Actual) |
|----------------|-------------|
| `gpt-4` | `claude-sonnet-4-20250514` |
| `gpt-4o` | `gemini-2.0-flash-exp` |
| `custom-model` | `qwen-max` |

### Toggle quan trọng

- **"Prioritize Model Mappings"**: Bật để ưu tiên mapping thay vì local API keys

---

## 6. Custom OpenAI-Compatible Providers

### Mục đích

Thêm bất kỳ provider nào hỗ trợ OpenAI API format.

### Cách cấu hình

1. Vào **Settings** → **"Custom OpenAI-Compatible Providers"**
2. Click **"Add Provider"**
3. Điền thông tin:

| Field | Mô tả | Ví dụ |
|-------|-------|-------|
| **Name** | Tên provider | `OpenRouter` |
| **Base URL** | Endpoint API | `https://openrouter.ai/api/v1` |
| **API Key** | API key của bạn | `sk-or-xxx...` |
| **Models** | Danh sách models | `openai/gpt-4-turbo` |

### Providers phổ biến

| Provider | Base URL |
|----------|----------|
| **OpenRouter** | `https://openrouter.ai/api/v1` |
| **Together AI** | `https://api.together.xyz/v1` |
| **Groq** | `https://api.groq.com/openai/v1` |
| **Fireworks** | `https://api.fireworks.ai/inference/v1` |
| **DeepSeek** | `https://api.deepseek.com/v1` |
| **Mistral** | `https://api.mistral.ai/v1` |
| **Perplexity** | `https://api.perplexity.ai` |
| **LiteLLM** | `http://localhost:4000` |

---

## 7. Custom Anthropic API (Azure AI Foundry)

### Mục đích

Thêm custom Anthropic API endpoint như **Azure AI Foundry**, **Amazon Bedrock**, hoặc bất kỳ endpoint nào hỗ trợ Anthropic API format.

### Cấu trúc ClaudeApiKey hỗ trợ

ProxyPal hỗ trợ đầy đủ các trường cho Claude API:

```typescript
interface ClaudeApiKey {
    apiKey: string;        // API key
    baseUrl?: string;      // Custom base URL (Azure, Bedrock, etc.)
    proxyUrl?: string;     // Proxy nếu cần
    headers?: Record<string, string>;  // Custom headers
    models?: ModelMapping[];  // Model aliasing (quan trọng!)
}

interface ModelMapping {
    name: string;   // Tên model thực trên provider (vd: "claude-opus-4-5")
    alias: string;  // Tên model khi request (vd: "claude-opus-4-5-20251101")
}
```

---

### Hướng dẫn add Azure AI Foundry vào ProxyPal

#### Bước 1: Mở API Keys page

- Sidebar → **API Keys**
- Hoặc dùng shortcut `Ctrl+K` → gõ "API Keys"

#### Bước 2: Chọn tab Claude

Click tab **"Claude"** (không phải OpenAI Compatible)

#### Bước 3: Click "Add Claude API Key"

Nút ở cuối danh sách. **Lưu ý**: Proxy phải đang chạy mới bấm được.

#### Bước 4: Điền thông tin

| Field | Value |
|-------|-------|
| **API Key*** | API key từ Azure portal |
| **Base URL** | `https://{RESOURCE}.services.ai.azure.com/anthropic` |

**Lưu ý:** Base URL KHÔNG có `/v1` ở cuối (proxy tự thêm).

#### Bước 5: Thêm Model Aliasing

**Quan trọng:** Azure AI Foundry deployment names khác với Anthropic standard names. Cần thêm model aliasing để proxy route đúng.

| Alias (tên request) | Name (tên trên Azure) |
|---------------------|----------------------|
| `claude-opus-4-5-20251101` | `claude-opus-4-5` |
| `claude-sonnet-4-5-20250929` | `claude-sonnet-4-5` |
| `claude-haiku-4-5-20251001` | `claude-haiku-4-5` |

Khi client request `claude-opus-4-5-20251101`, proxy sẽ gọi Azure với model `claude-opus-4-5`.

#### Bước 6: Click "Add Key"

Key sẽ được lưu và hiển thị trong danh sách.

---

### Giao diện UI

```
┌─────────────────────────────────────────────────┐
│  API Keys                                       │
├─────────────────────────────────────────────────┤
│  [Gemini] [Claude] [Codex] [OpenAI Compatible]  │  ← Chọn tab Claude
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌───────────────────────────────────────────┐  │
│  │ API Key *                                 │  │
│  │ [your-azure-api-key-here...]              │  │ ← Paste API key
│  │                                           │  │
│  │ Base URL (optional)                       │  │
│  │ [https://resource.services.ai.azure.com/  │  │ ← Paste Azure URL
│  │  anthropic]                               │  │
│  │                                           │  │
│  │ Models (aliasing)                         │  │
│  │ ┌─────────────────────────────────────┐   │  │
│  │ │ Name: claude-opus-4-5               │   │  │
│  │ │ Alias: claude-opus-4-5-20251101     │   │  │
│  │ └─────────────────────────────────────┘   │  │
│  │                                           │  │
│  │ [Add Key]  [Cancel]                       │  │
│  └───────────────────────────────────────────┘  │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

### Ví dụ Azure AI Foundry

**Environment variables từ Azure:**

```bash
export CLAUDE_CODE_USE_FOUNDRY=1
export ANTHROPIC_FOUNDRY_RESOURCE="your-resource-name"
export ANTHROPIC_FOUNDRY_API_KEY="your-api-key"
export ANTHROPIC_DEFAULT_OPUS_MODEL="claude-opus-4-5"
export ANTHROPIC_DEFAULT_SONNET_MODEL="claude-sonnet-4-5"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="claude-haiku-4-5"
```

**Chuyển đổi sang ProxyPal:**

| Azure Env | ProxyPal Field |
|-----------|----------------|
| `ANTHROPIC_FOUNDRY_API_KEY` | **API Key** |
| `https://{RESOURCE}.services.ai.azure.com/anthropic` | **Base URL** |
| `ANTHROPIC_DEFAULT_*_MODEL` | **Models aliasing** |

---

### Test endpoint

**Test Azure Foundry endpoint trực tiếp:**

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

**Test qua ProxyPal sau khi add:**

```powershell
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

---

### Lưu ý quan trọng

| Lưu ý | Chi tiết |
|-------|----------|
| **Proxy phải chạy** | Nút "Add Key" bị disabled nếu proxy chưa start |
| **API format** | Azure Foundry dùng Anthropic format (`/messages`), không phải OpenAI |
| **Model aliasing** | Bắt buộc nếu deployment names khác standard |
| **Headers** | ProxyPal tự thêm `x-api-key` và `anthropic-version` |

---

### Các endpoint Anthropic-compatible khác

| Provider | Base URL |
|----------|----------|
| **Azure AI Foundry** | `https://{resource}.services.ai.azure.com/anthropic` |
| **Amazon Bedrock** | `https://bedrock-runtime.{region}.amazonaws.com` |
| **Anthropic Direct** | `https://api.anthropic.com` (default) |
| **Self-hosted proxy** | `http://localhost:xxxx` |

---

## 8. Amp Code Integration

### Mục đích

Sử dụng **Amp Code** (Sourcegraph's AI coding assistant) thông qua ProxyPal để route requests đến Azure AI Foundry hoặc các providers khác.

### Cấu hình Amp Code

Chỉnh sửa file `~/.config/amp/settings.json`:

```json
{
  "amp.url": "http://localhost:8317",
  "amp.apiKey": "proxypal-local",
  "amp.anthropic.thinking.enabled": true
}
```

| Field | Giá trị | Mô tả |
|-------|---------|-------|
| `amp.url` | `http://localhost:8317` | ProxyPal endpoint |
| `amp.apiKey` | `proxypal-local` | API key mặc định của ProxyPal |
| `amp.anthropic.thinking.enabled` | `true` | Bật thinking mode cho Claude |

### Cấu hình ProxyPal cho Amp

ProxyPal hỗ trợ **Amp Model Mappings** để route models:

1. Vào **Settings** → **Amp Code Integration**
2. Thêm **Amp API Key** từ [ampcode.com/settings](https://ampcode.com/settings)
3. Cấu hình model mappings:

| From (Amp request) | To (Actual model) |
|-------------------|-------------------|
| `claude-sonnet-4-5-20250929` | `claude-opus-4-5-20251101` |
| `claude-haiku-4-5-20251001` | `claude-opus-4-5-20251101` |
| `gemini-2.5-flash` | `gemini-3-pro-preview` |

### Workflow

```
┌─────────────────────────────────────────┐
│  Amp Code                               │
│  Request: claude-sonnet-4-5-20250929    │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│  ProxyPal (localhost:8317)              │
│  1. Check Amp Model Mappings            │
│     sonnet → opus ✓                     │
│  2. Route to Azure AI Foundry           │
│     model: claude-opus-4-5              │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│  Azure AI Foundry                       │
│  Deployment: claude-opus-4-5            │
└─────────────────────────────────────────┘
```

### Ví dụ cấu hình hoàn chỉnh

**Amp settings (`~/.config/amp/settings.json`):**

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
    { "tool": "*", "action": "allow" }
  ]
}
```

**ProxyPal config (tham khảo `examples/config.example.json`):**

Xem thêm trong thư mục `examples/` để có cấu hình mẫu đầy đủ.

---

## 9. Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Mở Command Palette |
| `Ctrl+,` | Mở Settings |
| `Ctrl+L` | Mở Logs |

---

## 10. Troubleshooting Windows

### Port đã được sử dụng

```powershell
# Kiểm tra port 8317
netstat -ano | findstr :8317

# Kill process đang dùng port
taskkill /PID <PID> /F
```

### Proxy không start

```powershell
# Kiểm tra firewall
# Windows Defender Firewall → Allow an app → Add ProxyPal

# Hoặc chạy với admin rights
```

### OAuth không hoạt động

1. Đảm bảo proxy đang chạy (port 8317)
2. Check browser có bị block popups không
3. Thử disconnect và connect lại

### CLI agents không được detect

ProxyPal kiểm tra các paths sau:
- `C:\Program Files\nodejs\`
- `%USERPROFILE%\.cargo\bin\`
- `%USERPROFILE%\.npm-global\bin\`
- `%USERPROFILE%\AppData\Local\pnpm\`
- `%USERPROFILE%\.bun\bin\`

### "no auth available" error

Nguyên nhân: Model aliasing chưa được cấu hình đúng.

**Giải pháp:**
1. Kiểm tra `proxy-config.yaml` có section `models:` không
2. Đảm bảo alias và name được map đúng
3. Restart proxy sau khi thay đổi config

### Azure AI Foundry "DeploymentNotFound"

Nguyên nhân: Model name trong request không khớp deployment name trên Azure.

**Giải pháp:**
1. Kiểm tra deployment name thực tế trên Azure portal
2. Thêm model aliasing: `alias: "claude-opus-4-5-20251101"` → `name: "claude-opus-4-5"`

---

## 11. Config Files

### Location

| Tệp | Vị trí |
|-----|--------|
| **config.json** | `%APPDATA%\proxypal\config.json` |
| **proxy-config.yaml** | `%APPDATA%\proxypal\proxy-config.yaml` (auto-generated) |
| **Amp settings** | `~/.config/amp/settings.json` |

### Example configs

Repo chứa các tệp cấu hình mẫu trong thư mục `examples/`:

```
examples/
├── README.md                    # Hướng dẫn sử dụng
├── config.example.json          # ProxyPal config mẫu
├── proxy-config.example.yaml    # Proxy config mẫu
└── amp-settings.example.json    # Amp Code settings mẫu
```

**Lưu ý:** Các tệp example đã che dấu API keys. Copy và thay thế bằng keys thật của bạn.

### Cấu trúc config.json

```json
{
  "port": 8317,
  "autoStart": true,
  "launchAtLogin": false,
  "debug": false,
  "usageStatsEnabled": false,
  "requestLogging": false,
  "loggingToFile": false,
  "configVersion": 1,

  "ampApiKey": "your-amp-api-key",
  "ampModelMappings": [
    { "from": "claude-sonnet-4-5-20250929", "to": "claude-opus-4-5-20251101", "enabled": true }
  ],
  "ampRoutingMode": "mappings",
  "forceModelMappings": false,

  "claudeApiKeys": [
    {
      "apiKey": "your-azure-api-key",
      "baseUrl": "https://resource.services.ai.azure.com/anthropic",
      "headers": {
        "x-api-key": "your-azure-api-key",
        "anthropic-version": "2023-06-01"
      },
      "models": [
        { "name": "claude-opus-4-5", "alias": "claude-opus-4-5-20251101" },
        { "name": "claude-opus-4-5", "alias": "claude-sonnet-4-5-20250929" }
      ]
    }
  ],
  "geminiApiKeys": [],
  "codexApiKeys": [],

  "thinkingBudgetMode": "custom",
  "thinkingBudgetCustom": 16000,
  "reasoningEffortLevel": "xhigh",

  "copilot": {
    "enabled": false,
    "port": 4141,
    "accountType": "individual"
  },

  "closeToTray": true,
  "maxRetryInterval": 0
}
```

---

## Workflow tổng quan

```
┌─────────────────────────────────────────────────────────────┐
│  Coding Tool (Cursor, Cline, Claude Code, Amp Code, etc.)   │
│  Request: model="claude-opus-4-5-20251101"                  │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  ProxyPal (localhost:8317)                                  │
│                                                             │
│  1. Check Model Aliasing (Claude API Keys)                  │
│     - claude-opus-4-5-20251101 → claude-opus-4-5 ✓          │
│                                                             │
│  2. Check Model Mappings (Amp)                              │
│     - sonnet → opus? ✓                                      │
│                                                             │
│  3. Route to Provider                                       │
│     - Azure AI Foundry (with aliased model name)            │
│     - OAuth account (Claude, OpenAI, Gemini)                │
│     - Custom Provider (OpenRouter, etc.)                    │
└──────────────────────┬──────────────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  Azure AI Foundry                                           │
│  Deployment: claude-opus-4-5                                │
│  Response → Client                                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Hỗ trợ

- GitHub Issues: https://github.com/nicepkg/proxypal/issues
- Buy me a coffee: https://buymeacoffee.com/nicepkg

---

*Cập nhật: 2025-12-18*
*Phiên bản: 0.1.63*
