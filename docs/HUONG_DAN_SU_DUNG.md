# Hướng dẫn sử dụng ProxyPal

> ProxyPal - Sử dụng AI subscriptions (Claude, ChatGPT, Gemini, GitHub Copilot) với bất kỳ coding tool nào.

---

## Mục lục

1. [Cài đặt](#1-cài-đặt)
2. [Khởi động và kết nối](#2-khởi-động-và-kết-nối)
3. [Cấu hình Coding Tools](#3-cấu-hình-coding-tools)
4. [Các trang chính](#4-các-trang-chính)
5. [Model Mapping](#5-model-mapping)
6. [Server Mode - Chia sẻ với máy Remote](#6-server-mode---chia-sẻ-với-máy-remote)
7. [Custom OpenAI-Compatible Providers](#7-custom-openai-compatible-providers)
8. [Custom Anthropic API (Azure AI Foundry)](#8-custom-anthropic-api-azure-ai-foundry)
9. [Amp Code Integration](#9-amp-code-integration)
10. [Keyboard Shortcuts](#10-keyboard-shortcuts)
11. [Troubleshooting Windows](#11-troubleshooting-windows)
12. [Config Files](#12-config-files)

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

## 6. Server Mode - Chia sẻ với máy Remote

### Mục đích

Biến ProxyPal thành **router trung tâm** cho nhiều máy trong mạng. Các máy remote có thể sử dụng tất cả models đã cấu hình trên máy chủ mà không cần đăng nhập OAuth riêng.

### Bật Server Mode

1. Vào **Settings** → Section **"Server Mode"**
2. Bật toggle **"Enable Server Mode"**
3. Sẽ hiển thị:
   - **Remote Endpoints**: Các địa chỉ IP của máy (ví dụ: `http://10.0.0.4:8317/v1`)
   - **Remote API Key**: Key để xác thực cho máy remote
4. **Restart proxy** để áp dụng thay đổi

### Mở Firewall (Mạng nội bộ)

```powershell
# Windows - Mở port 8317
New-NetFirewallRule -DisplayName "ProxyPal Server Mode" -Direction Inbound -Protocol TCP -LocalPort 8317 -Action Allow

# Linux
sudo ufw allow 8317/tcp
```

### Expose ra Internet với Tailscale Funnel (Khuyên dùng)

Sử dụng **Tailscale Funnel** để expose ProxyPal với domain miễn phí, cố định vĩnh viễn:

#### Cài đặt Tailscale

```powershell
# Windows - Cài qua winget
winget install tailscale.tailscale

# Hoặc tải từ https://tailscale.com/download
```

#### Đăng nhập (miễn phí)

```powershell
# Đăng nhập với Google/Microsoft/GitHub
tailscale up
```

#### Chạy Funnel

```powershell
# Foreground (tắt khi đóng terminal)
tailscale funnel 8317

# Background (tiếp tục chạy sau khi đóng terminal) - KHUYÊN DÙNG
tailscale funnel --bg 8317
```

**Lưu ý Windows:** Nếu `tailscale` không có trong PATH, dùng đường dẫn đầy đủ:
```powershell
& 'C:\Program Files\Tailscale\tailscale.exe' funnel --bg 8317
```

**Kết quả:** Nhận domain dạng `https://your-pc.tail12345.ts.net`

#### Kiểm tra Funnel Status

```powershell
tailscale funnel status

# Hoặc với đường dẫn đầy đủ:
& 'C:\Program Files\Tailscale\tailscale.exe' funnel status
```

Kết quả khi Funnel đang chạy:
```
Available on the internet:

https://your-pc.tail12345.ts.net/
|-- proxy http://127.0.0.1:8317
```

Nếu thấy `No serve config` → Funnel chưa được bật.

#### Tắt Funnel

```powershell
# Tắt Funnel
tailscale funnel off

# Hoặc
& 'C:\Program Files\Tailscale\tailscale.exe' funnel off
```

#### Các phương thức kết nối

| Phương thức | URL | Khi nào dùng |
|-------------|-----|--------------|
| **Tailscale Funnel** | `https://xxx.ts.net` | Máy remote không cài Tailscale |
| **IP Tailscale** | `http://100.x.x.x:8317` | Cả 2 máy đều cài Tailscale (nhanh hơn) |
| **IP LAN** | `http://192.168.x.x:8317` | Cùng mạng nội bộ |

Lấy IP Tailscale: `tailscale ip -4` hoặc `& 'C:\Program Files\Tailscale\tailscale.exe' ip -4`

#### So sánh các phương án Tunneling

| Phương án | Bandwidth | Domain cố định | Chi phí |
|-----------|-----------|----------------|---------|
| **Tailscale Funnel** | Unlimited | ✅ `*.ts.net` | **$0** |
| **Cloudflare Tunnel** | Unlimited | ✅ Cần domain riêng | ~$10/năm |
| **ngrok (Free)** | 1GB/tháng | ❌ Thay đổi | $0 |
| **Zrok** | 250MB/tháng | ✅ `*.zrok.io` | $0 |

#### Ưu điểm Tailscale Funnel

| Tính năng | Giá trị |
|-----------|---------|
| **Domain** | Miễn phí subdomain `*.ts.net` cố định vĩnh viễn |
| **HTTPS** | Tự động, miễn phí |
| **Bandwidth** | Không giới hạn data transfer |
| **Setup** | 1 phút |
| **Chi phí** | **$0** |

#### Giữ Funnel chạy liên tục

**Cách 1: Chạy background (Khuyên dùng)**
```powershell

& 'C:\Program Files\Tailscale\tailscale.exe' funnel --bg 8317

```

**Cách 2: Tạo Scheduled Task** để tự động chạy khi khởi động Windows

#### Troubleshooting Tailscale Funnel

| Lỗi | Nguyên nhân | Giải pháp |
|-----|-------------|-----------|
| `No serve config` | Funnel chưa bật | Chạy `tailscale funnel --bg 8317` |
| `Connection refused` | Proxy không chạy hoặc Server Mode chưa bật | Kiểm tra ProxyPal: Start proxy + Enable Server Mode |
| `ECONNRESET` | Funnel đã tắt | Chạy lại `tailscale funnel --bg 8317` |
| `tailscale: command not found` | Tailscale không có trong PATH | Dùng đường dẫn đầy đủ: `& 'C:\Program Files\Tailscale\tailscale.exe'` |
| Funnel cần approval | Admin chưa approve | Vào [Tailscale Admin Console](https://login.tailscale.com/admin) → approve funnel |

#### Test kết nối

```powershell
# Test từ máy bất kỳ
curl https://your-pc.tail12345.ts.net/v1/models -H "Authorization: Bearer YOUR_REMOTE_API_KEY"

# Nếu thành công, sẽ trả về danh sách models
```

### Cấu hình máy Remote

#### Claude Code / Aider

```bash
# Qua Tailscale Funnel (khuyên dùng)
export ANTHROPIC_BASE_URL=https://<YOUR_TAILSCALE_DOMAIN>
export ANTHROPIC_AUTH_TOKEN=<REMOTE_API_KEY>

# Hoặc qua mạng nội bộ
export ANTHROPIC_BASE_URL=http://<SERVER_IP>:8317
export ANTHROPIC_AUTH_TOKEN=<REMOTE_API_KEY>

# Chọn model routing
export ANTHROPIC_DEFAULT_OPUS_MODEL=gemini-2.5-pro
export ANTHROPIC_DEFAULT_SONNET_MODEL=gemini-2.5-flash
export ANTHROPIC_DEFAULT_HAIKU_MODEL=gemini-2.5-flash-lite

# Hoặc dùng model khác
export ANTHROPIC_MODEL=claude-opus-4-5-20251101
```

#### OpenAI-compatible clients (Cursor, Continue, Cline)

```bash
# Qua Tailscale Funnel
export OPENAI_BASE_URL=https://<YOUR_TAILSCALE_DOMAIN>/v1
export OPENAI_API_KEY=<REMOTE_API_KEY>

# Hoặc qua mạng nội bộ
export OPENAI_BASE_URL=http://<SERVER_IP>:8317/v1
export OPENAI_API_KEY=<REMOTE_API_KEY>
```

#### Ví dụ thực tế với Tailscale Funnel

```bash
# Domain thực tế: https://rdp-telstechs.tailba88a3.ts.net
export ANTHROPIC_BASE_URL="https://rdp-telstechs.tailba88a3.ts.net"
export ANTHROPIC_API_KEY="proxypal-remote-xxxxx"

# Hoặc cho OpenAI-compatible
export OPENAI_BASE_URL="https://rdp-telstechs.tailba88a3.ts.net/v1"
export OPENAI_API_KEY="proxypal-remote-xxxxx"
```

### Models có sẵn

Tùy thuộc vào providers đã cấu hình trên máy chủ. Danh sách đầy đủ (cập nhật 2025-12-19):

| Provider | Số lượng | Models |
|----------|----------|--------|
| **Claude OAuth/API** | 3 | `claude-opus-4-5-20251101`, `claude-sonnet-4-5-20250929`, `claude-haiku-4-5` |
| **Gemini OAuth** | 3 | `gemini-2.5-pro`, `gemini-2.5-flash`, `gemini-2.5-flash-lite` |
| **Antigravity** | 10 | `gemini-2.5-flash`, `gemini-2.5-flash-lite`, `gemini-2.5-computer-use-preview-10-2025`, `gemini-3-pro-preview`, `gemini-3-pro-image-preview`, `gemini-3-flash-preview`, `gemini-claude-sonnet-4-5`, `gemini-claude-sonnet-4-5-thinking`, `gemini-claude-opus-4-5-thinking`, `gpt-oss-120b-medium` |
| **Codex OAuth** | 8 | `gpt-5`, `gpt-5-codex`, `gpt-5.1`, `gpt-5.1-codex`, `gpt-5.1-codex-mini`, `gpt-5.1-codex-max`, `gpt-5.2` |
| **GitHub Copilot** | 23 | `gpt-4`, `gpt-4.1`, `gpt-4o`, `gpt-4-turbo`, `gpt-5`, `gpt-5-mini`, `gpt-5-codex`, `gpt-5.1`, `gpt-5.1-codex`, `gpt-5.1-codex-mini`, `gpt-5.1-codex-max`, `gpt-5.2`, `o1`, `o1-mini`, `grok-code-fast-1`, `raptor-mini`, `gemini-2.5-pro`, `gemini-3-pro-preview`, `claude-haiku-4.5`, `claude-opus-4.1`, `claude-sonnet-4`, `claude-sonnet-4.5`, `claude-opus-4.5` |

**Tip:** Xem tất cả models trong **Logs** page (filter DEBUG) sau khi proxy start.

### Workflow

```
┌─────────────────────────────────────────┐
│  Máy Remote (Claude Code)               │
│  Request: gemini-2.5-pro                │
│  API Key: proxypal-remote-xxx           │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│  ProxyPal Server (10.0.0.4:8317)        │
│  1. Xác thực Remote API Key ✓           │
│  2. Route đến Gemini OAuth              │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│  Google Gemini API                      │
│  Response → Máy Remote                  │
└─────────────────────────────────────────┘
```

### Quản lý API Key

- **Generate new key**: Click nút refresh bên cạnh API key
- **Copy key**: Click icon copy để sao chép
- Key được lưu trong config và tự động thêm vào proxy khi Server Mode bật

### Lưu ý bảo mật

| Lưu ý | Chi tiết |
|-------|----------|
| **Private network** | Chỉ nên dùng trong mạng nội bộ hoặc VPN |
| **Firewall** | Chỉ mở port cần thiết (8317) |
| **API Key** | Không chia sẻ API key công khai |
| **Restart** | Cần restart proxy sau khi bật/tắt Server Mode |

---

## 7. Custom OpenAI-Compatible Providers

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

## 8. Custom Anthropic API (Azure AI Foundry)

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

## 9. Amp Code Integration

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

## 10. Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Mở Command Palette |
| `Ctrl+,` | Mở Settings |
| `Ctrl+L` | Mở Logs |

---

## 11. Troubleshooting Windows

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

## 12. Config Files

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

