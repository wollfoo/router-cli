# Hướng Dẫn Sử Dụng ProxyPal

> **ProxyPal** - Sử dụng AI subscriptions (Claude, ChatGPT, Gemini, GitHub Copilot) với bất kỳ coding tool nào thông qua một endpoint duy nhất.

---

## Giới Thiệu

ProxyPal là ứng dụng desktop giúp hợp nhất nhiều AI subscriptions thành một OpenAI-compatible endpoint. Bạn có thể:

- **Dùng 1 endpoint** cho tất cả coding tools (Cursor, Continue, Claude Code, Cline, Aider)
- **Chia sẻ subscriptions** với nhiều máy qua Server Mode
- **Route models** linh hoạt với Model Mapping
- **Kết nối enterprise** như Azure AI Foundry

```
┌─────────────────────────────────────────────────────────────┐
│  Coding Tools (Cursor, Claude Code, Cline, Aider, etc.)     │
│            ↓                                                │
│  http://localhost:8317/v1  ← ProxyPal endpoint              │
│            ↓                                                │
│  ProxyPal routes đến provider tốt nhất:                     │
│  • Claude Pro (OAuth)                                       │
│  • Gemini Advanced (OAuth)                                  │
│  • Azure AI Foundry (API Key)                               │
│  • GitHub Copilot (OAuth)                                   │
│  • Custom providers (OpenRouter, Groq, etc.)                │
└─────────────────────────────────────────────────────────────┘
```

---

## Mục Lục Tài Liệu

### Bắt Đầu

| Tài liệu | Mô tả |
|----------|-------|
| [**Cài Đặt**](./CAI_DAT.md) | Cài đặt trên Windows, macOS, Linux. Build từ source |
| [**Khởi Động**](./KHOI_DONG.md) | Khởi động app, kết nối providers, start proxy |

### Cấu Hình

| Tài liệu | Mô tả |
|----------|-------|
| [**Cấu Hình Tools**](./CAU_HINH_TOOLS.md) | Thiết lập Cursor, Continue, Claude Code, Cline, Aider |
| [**Model Mapping**](./MODEL_MAPPING.md) | Route requests giữa các models |
| [**Custom Providers**](./CUSTOM_PROVIDERS.md) | Thêm OpenAI-compatible providers, Azure AI Foundry |

### Nâng Cao

| Tài liệu | Mô tả |
|----------|-------|
| [**Server Mode**](./SERVER_MODE.md) | Chia sẻ proxy với máy remote qua Tailscale |
| [**Amp Code**](./AMP_CODE.md) | Tích hợp Amp Code của Sourcegraph |
| [**Ví Dụ Endpoint**](./VI_DU_ENDPOINT.md) | Cấu hình mẫu cho nhiều providers |

### Hỗ Trợ

| Tài liệu | Mô tả |
|----------|-------|
| [**Xử Lý Lỗi**](./TROUBLESHOOTING.md) | Troubleshooting các lỗi thường gặp |

---

## Quick Start (5 phút)

### 1. Cài Đặt

```powershell
# Windows (winget)
winget install nicepkg.proxypal

# Hoặc tải installer từ:
# https://github.com/nicepkg/proxypal/releases
```

### 2. Kết Nối Provider

1. Mở ProxyPal
2. Dashboard → Click **"Connect"** bên cạnh provider (Claude, Gemini, etc.)
3. Đăng nhập OAuth trong browser
4. Đợi status chuyển ✓ Connected

### 3. Start Proxy

- Proxy tự động start nếu **Auto Start** được bật
- Hoặc click **"Start Proxy"** trên Dashboard
- Endpoint: `http://localhost:8317/v1`

### 4. Cấu Hình Coding Tool

#### Claude Code

```powershell
$env:ANTHROPIC_BASE_URL = "http://localhost:8317"
$env:ANTHROPIC_API_KEY = "proxypal-local"
```

#### Cursor

```
Settings → Models → OpenAI API Base URL
URL: http://localhost:8317/v1
API Key: proxypal-local
```

#### Continue (VS Code)

```json
// ~/.continue/config.json
{
  "models": [{
    "provider": "openai",
    "model": "claude-sonnet-4-5-20250929",
    "apiBase": "http://localhost:8317/v1",
    "apiKey": "proxypal-local"
  }]
}
```

### 5. Test

```bash
curl http://localhost:8317/v1/models
# → Danh sách models có sẵn
```

**Done!** Bắt đầu code với AI assistant.

---

## Các Trang Trong App

| Trang | Chức năng | Shortcut |
|-------|-----------|----------|
| **Dashboard** | Tổng quan, start/stop proxy, providers | `Ctrl+1` |
| **Settings** | Port, auto-start, model mappings | `Ctrl+,` |
| **API Keys** | Quản lý API keys | `Ctrl+3` |
| **Logs** | Request logs real-time | `Ctrl+L` |
| **Analytics** | Thống kê usage, tokens | `Ctrl+5` |

---

## Models Có Sẵn

Phụ thuộc vào providers đã kết nối:

| Provider | Models |
|----------|--------|
| **Claude OAuth** | `claude-opus-4-5-20251101`, `claude-sonnet-4-5-20250929` |
| **Gemini OAuth** | `gemini-2.5-pro`, `gemini-2.5-flash`, `gemini-2.5-flash-lite` |
| **GitHub Copilot** | `gpt-4o`, `claude-sonnet-4`, `gemini-2.5-pro`, etc. |
| **Azure AI Foundry** | Tùy deployments |

Xem đầy đủ: [Khởi Động → Models](./KHOI_DONG.md#6-các-trang-chính)

---

## Config Files

| File | Vị trí (Windows) |
|------|------------------|
| **config.json** | `%APPDATA%\proxypal\config.json` |
| **proxy-config.yaml** | `%APPDATA%\proxypal\proxy-config.yaml` |

Xem example configs trong thư mục `examples/`:

```
examples/
├── config.example.json          # ProxyPal config
├── proxy-config.example.yaml    # CLIProxyAPI config
└── amp-settings.example.json    # Amp Code settings
```

---

## Hỗ Trợ

- **Issues**: [github.com/nicepkg/proxypal/issues](https://github.com/nicepkg/proxypal/issues)
- **Discussions**: [github.com/nicepkg/proxypal/discussions](https://github.com/nicepkg/proxypal/discussions)
- **Troubleshooting**: [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)

---

## Tham Khảo Nhanh

### Environment Variables

```bash
# OpenAI-compatible (Cursor, Continue, Cline)
export OPENAI_BASE_URL="http://localhost:8317/v1"
export OPENAI_API_KEY="proxypal-local"

# Anthropic-compatible (Claude Code, Aider)
export ANTHROPIC_BASE_URL="http://localhost:8317"
export ANTHROPIC_API_KEY="proxypal-local"
```

### API Endpoints

| Endpoint | Format | Dùng cho |
|----------|--------|----------|
| `http://localhost:8317/v1/chat/completions` | OpenAI | Cursor, Continue, Cline |
| `http://localhost:8317/v1/messages` | Anthropic | Claude Code, Aider |
| `http://localhost:8317/v1/models` | - | Liệt kê models |

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Command Palette |
| `Ctrl+,` | Settings |
| `Ctrl+L` | Logs |
