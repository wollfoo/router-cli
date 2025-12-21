# Cấu Hình Coding Tools

Hướng dẫn chi tiết cấu hình các coding tools (Cursor, Continue, Claude Code, Cline, Aider, OpenCode) để sử dụng ProxyPal.

---

## Mục Lục

1. [Tổng Quan](#1-tổng-quan)
2. [Cursor](#2-cursor)
3. [Continue (VS Code)](#3-continue-vs-code)
4. [Claude Code CLI](#4-claude-code-cli)
5. [Cline (VS Code)](#5-cline-vs-code)
6. [Aider](#6-aider)
7. [OpenCode](#7-opencode)
8. [Zed Editor](#8-zed-editor)
9. [Amp Code](#9-amp-code)
10. [Environment Variables](#10-environment-variables)

---

## 1. Tổng Quan

### Endpoint Mặc Định

| Type | URL | Dùng cho |
|------|-----|----------|
| **OpenAI-compatible** | `http://localhost:8317/v1` | Cursor, Continue, Cline, OpenCode |
| **Anthropic-compatible** | `http://localhost:8317` | Claude Code, Aider |

### API Key

ProxyPal sử dụng API key mặc định:

| Key | Sử dụng khi |
|-----|------------|
| `proxypal-local` | Kết nối từ localhost |
| `proxypal-remote-xxxxx` | Kết nối từ máy remote (Server Mode) |

**Lưu ý:** Nhiều tools chấp nhận bất kỳ giá trị API key nào (dummy key) khi kết nối localhost.

---

## 2. Cursor

### Cấu Hình Qua UI

1. Mở **Settings** (`Ctrl+,`)
2. Tìm **"Models"** hoặc **"OpenAI API"**
3. Điền:

| Field | Giá trị |
|-------|---------|
| **Base URL** | `http://localhost:8317/v1` |
| **API Key** | `proxypal-local` (hoặc bất kỳ giá trị) |

### Cấu Hình Qua File

```json
// ~/.cursor/settings.json (hoặc Cursor settings)
{
  "openai.apiBaseUrl": "http://localhost:8317/v1",
  "openai.apiKey": "proxypal-local"
}
```

### Models Khuyên Dùng

| Model | Chức năng |
|-------|-----------|
| `claude-sonnet-4-5-20250929` | General coding, balanced |
| `claude-opus-4-5-20251101` | Complex tasks |
| `gemini-2.5-pro` | Fast, good for refactoring |
| `gemini-2.5-flash` | Quick completions |

### Test

1. Mở file code bất kỳ trong Cursor
2. Press `Ctrl+K` hoặc `Cmd+K`
3. Nhập prompt và kiểm tra response

---

## 3. Continue (VS Code)

### Cài Đặt Extension

1. VS Code → Extensions (`Ctrl+Shift+X`)
2. Search **"Continue"**
3. Install **"Continue - Codestral, Claude, and more"**

### Cấu Hình

Chỉnh sửa file `~/.continue/config.json`:

```json
{
  "models": [
    {
      "title": "Claude Sonnet (ProxyPal)",
      "provider": "openai",
      "model": "claude-sonnet-4-5-20250929",
      "apiBase": "http://localhost:8317/v1",
      "apiKey": "proxypal-local"
    },
    {
      "title": "Claude Opus (ProxyPal)",
      "provider": "openai",
      "model": "claude-opus-4-5-20251101",
      "apiBase": "http://localhost:8317/v1",
      "apiKey": "proxypal-local"
    },
    {
      "title": "Gemini Pro (ProxyPal)",
      "provider": "openai",
      "model": "gemini-2.5-pro",
      "apiBase": "http://localhost:8317/v1",
      "apiKey": "proxypal-local"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Gemini Flash (ProxyPal)",
    "provider": "openai",
    "model": "gemini-2.5-flash",
    "apiBase": "http://localhost:8317/v1",
    "apiKey": "proxypal-local"
  }
}
```

### Cấu Hình Nâng Cao

```json
{
  "models": [
    {
      "title": "Claude Sonnet",
      "provider": "openai",
      "model": "claude-sonnet-4-5-20250929",
      "apiBase": "http://localhost:8317/v1",
      "apiKey": "proxypal-local",
      "contextLength": 200000,
      "completionOptions": {
        "temperature": 0.7,
        "maxTokens": 4096
      }
    }
  ],
  "slashCommands": [
    {
      "name": "explain",
      "description": "Explain code",
      "prompt": "Explain the following code in detail:\n\n{{{ input }}}"
    }
  ]
}
```

### Vị Trí File Config

| OS | Đường dẫn |
|----|-----------|
| **Windows** | `%USERPROFILE%\.continue\config.json` |
| **macOS** | `~/.continue/config.json` |
| **Linux** | `~/.continue/config.json` |

---

## 4. Claude Code CLI

### Environment Variables

#### Session-only (PowerShell)

```powershell
$env:ANTHROPIC_BASE_URL = "http://localhost:8317"
$env:ANTHROPIC_API_KEY = "proxypal-local"
```

#### Session-only (Bash/Zsh)

```bash
export ANTHROPIC_BASE_URL="http://localhost:8317"
export ANTHROPIC_API_KEY="proxypal-local"
```

### Permanent (Windows)

```powershell
# User level (recommended)
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "http://localhost:8317", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "proxypal-local", "User")

# System level (all users)
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "http://localhost:8317", "Machine")
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "proxypal-local", "Machine")
```

### Permanent (macOS/Linux)

Thêm vào `~/.bashrc`, `~/.zshrc`, hoặc `~/.profile`:

```bash
export ANTHROPIC_BASE_URL="http://localhost:8317"
export ANTHROPIC_API_KEY="proxypal-local"
```

Sau đó reload:

```bash
source ~/.bashrc  # hoặc ~/.zshrc
```

### Model Routing (Optional)

```bash
# Route tất cả Opus requests về Sonnet
export ANTHROPIC_DEFAULT_OPUS_MODEL="claude-sonnet-4-5-20250929"

# Hoặc route về Gemini
export ANTHROPIC_DEFAULT_OPUS_MODEL="gemini-2.5-pro"
export ANTHROPIC_DEFAULT_SONNET_MODEL="gemini-2.5-flash"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="gemini-2.5-flash-lite"
```

### Test

```bash
claude-code --version
claude-code chat "Hello, can you help me?"
```

---

## 5. Cline (VS Code)

### Cài Đặt

1. VS Code → Extensions
2. Search **"Cline"**
3. Install

### Cấu Hình

1. Open Cline sidebar
2. Click Settings icon (⚙️)
3. Chọn **"API Provider"** → **"OpenAI Compatible"**

| Field | Giá trị |
|-------|---------|
| **Base URL** | `http://localhost:8317/v1` |
| **API Key** | `proxypal-local` |
| **Model** | `claude-sonnet-4-5-20250929` |

### Cấu Hình File

```json
// VS Code settings.json
{
  "cline.apiProvider": "openai-compatible",
  "cline.openaiApiBase": "http://localhost:8317/v1",
  "cline.openaiApiKey": "proxypal-local",
  "cline.model": "claude-sonnet-4-5-20250929"
}
```

---

## 6. Aider

### Installation

```bash
pip install aider-chat
```

### Environment Variables

```bash
export ANTHROPIC_BASE_URL="http://localhost:8317"
export ANTHROPIC_API_KEY="proxypal-local"
```

### Cấu Hình File

Tạo `~/.aider.conf.yml`:

```yaml
anthropic-api-base: http://localhost:8317
anthropic-api-key: proxypal-local
model: claude-sonnet-4-5-20250929
```

### Command Line Options

```bash
# Sử dụng Claude qua ProxyPal
aider --anthropic-api-base http://localhost:8317 --anthropic-api-key proxypal-local

# Với model cụ thể
aider --model claude-sonnet-4-5-20250929
```

### Test

```bash
cd /path/to/your/project
aider
# Sau đó nhập: /help
```

---

## 7. OpenCode

### Environment Variables

```powershell
# PowerShell
$env:OPENAI_BASE_URL = "http://localhost:8317/v1"
$env:OPENAI_API_KEY = "proxypal-local"
```

```bash
# Bash/Zsh
export OPENAI_BASE_URL="http://localhost:8317/v1"
export OPENAI_API_KEY="proxypal-local"
```

### Permanent (Windows)

```powershell
[Environment]::SetEnvironmentVariable("OPENAI_BASE_URL", "http://localhost:8317/v1", "User")
[Environment]::SetEnvironmentVariable("OPENAI_API_KEY", "proxypal-local", "User")
```

### Test

```bash
opencode --help
opencode "Explain this code"
```

---

## 8. Zed Editor

### Cấu Hình

Chỉnh sửa `~/.config/zed/settings.json`:

```json
{
  "assistant": {
    "default_model": {
      "provider": "openai",
      "model": "claude-sonnet-4-5-20250929"
    },
    "openai": {
      "base_url": "http://localhost:8317/v1",
      "api_key": "proxypal-local"
    }
  }
}
```

### Anthropic Provider

```json
{
  "assistant": {
    "default_model": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-5-20250929"
    },
    "anthropic": {
      "base_url": "http://localhost:8317",
      "api_key": "proxypal-local"
    }
  }
}
```

---

## 9. Amp Code

### Cấu Hình

Chỉnh sửa `~/.config/amp/settings.json`:

```json
{
  "amp.url": "http://localhost:8317",
  "amp.apiKey": "proxypal-local",
  "amp.anthropic.thinking.enabled": true,
  "amp.tools.stopTimeout": 600
}
```

### Chi Tiết

Xem hướng dẫn đầy đủ tại: [AMP_CODE.md](./AMP_CODE.md)

---

## 10. Environment Variables

### Tổng Hợp Variables

#### OpenAI-Compatible (Cursor, Continue, Cline, OpenCode)

| Variable | Giá trị |
|----------|---------|
| `OPENAI_BASE_URL` | `http://localhost:8317/v1` |
| `OPENAI_API_KEY` | `proxypal-local` |

#### Anthropic-Compatible (Claude Code, Aider)

| Variable | Giá trị |
|----------|---------|
| `ANTHROPIC_BASE_URL` | `http://localhost:8317` |
| `ANTHROPIC_API_KEY` | `proxypal-local` |

### Script Tự Động (Windows PowerShell)

```powershell
# setup-proxypal.ps1
Write-Host "Setting up ProxyPal environment variables..."

# OpenAI-compatible
[Environment]::SetEnvironmentVariable("OPENAI_BASE_URL", "http://localhost:8317/v1", "User")
[Environment]::SetEnvironmentVariable("OPENAI_API_KEY", "proxypal-local", "User")

# Anthropic-compatible
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "http://localhost:8317", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "proxypal-local", "User")

Write-Host "Done! Please restart your terminal/IDE."
```

### Script Tự Động (Bash)

```bash
#!/bin/bash
# setup-proxypal.sh

echo "Setting up ProxyPal environment variables..."

# Add to .bashrc or .zshrc
SHELL_RC="$HOME/.bashrc"
[ -n "$ZSH_VERSION" ] && SHELL_RC="$HOME/.zshrc"

cat >> "$SHELL_RC" << 'EOF'

# ProxyPal Configuration
export OPENAI_BASE_URL="http://localhost:8317/v1"
export OPENAI_API_KEY="proxypal-local"
export ANTHROPIC_BASE_URL="http://localhost:8317"
export ANTHROPIC_API_KEY="proxypal-local"
EOF

echo "Done! Run: source $SHELL_RC"
```

### Kết Nối Remote (Server Mode)

Khi sử dụng ProxyPal từ máy khác:

```bash
# Thay localhost bằng IP của máy server
export ANTHROPIC_BASE_URL="http://192.168.1.100:8317"
export ANTHROPIC_API_KEY="proxypal-remote-xxxxx"

# Hoặc qua Tailscale Funnel
export ANTHROPIC_BASE_URL="https://your-pc.tail12345.ts.net"
export ANTHROPIC_API_KEY="proxypal-remote-xxxxx"
```

Xem chi tiết: [Server Mode](./SERVER_MODE.md)

---

## Xử Lý Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Giải pháp |
|-----|-------------|-----------|
| `Connection refused` | Proxy chưa start | Start proxy trong ProxyPal |
| `401 Unauthorized` | API key sai | Kiểm tra lại key |
| `No models available` | Chưa connect provider | Connect ít nhất 1 provider |
| `Model not found` | Model name sai | Kiểm tra tên model đúng |
| `Timeout` | Firewall hoặc mạng chậm | Mở port, kiểm tra mạng |

---

## Tham Khảo

- [Model Mapping](./MODEL_MAPPING.md)
- [Custom Providers](./CUSTOM_PROVIDERS.md)
- [Xử Lý Lỗi](./TROUBLESHOOTING.md)
- [Server Mode](./SERVER_MODE.md)
