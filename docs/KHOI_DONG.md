# Khởi Động và Kết Nối Providers

Hướng dẫn chi tiết khởi động ProxyPal và kết nối các AI providers (Claude, ChatGPT, Gemini, Copilot, Qwen).

---

## Mục Lục

1. [Khởi Động App](#1-khởi-động-app)
2. [Giao Diện Chính](#2-giao-diện-chính)
3. [Kết Nối Providers](#3-kết-nối-providers)
4. [Start Proxy](#4-start-proxy)
5. [Kiểm Tra Kết Nối](#5-kiểm-tra-kết-nối)
6. [Các Trang Chính](#6-các-trang-chính)
7. [Keyboard Shortcuts](#7-keyboard-shortcuts)

---

## 1. Khởi Động App

### Lần Đầu Mở App

1. Chạy ProxyPal từ Start Menu (Windows), Applications (macOS), hoặc App Launcher (Linux)
2. Màn hình **Welcome** hiển thị nếu lần đầu sử dụng
3. Click **"Get Started"** để vào Dashboard

### Các Lần Sau

- App tự động mở vào Dashboard
- Nếu `Auto Start` được bật, proxy tự động chạy

### Chạy Từ Command Line

```bash
# Windows
& "C:\Program Files\proxypal\ProxyPal.exe"

# macOS
open /Applications/ProxyPal.app

# Linux
/usr/bin/proxypal
```

---

## 2. Giao Diện Chính

### Layout

```
┌──────────────────────────────────────────────────────────┐
│  ProxyPal                          [_] [□] [X]           │
├─────────┬────────────────────────────────────────────────┤
│ Sidebar │  Main Content                                  │
│         │                                                │
│ [⌂]     │  ┌─────────────────────────────────────────┐  │
│ Dashboard│  │  Proxy Status: ● Running on :8317       │  │
│         │  │  [Stop Proxy]                            │  │
│ [⚙]     │  │                                         │  │
│ Settings│  │  Connected Providers:                   │  │
│         │  │  ✓ Claude    ✓ Gemini                   │  │
│ [🔑]    │  │  ✗ ChatGPT   ✗ Copilot                  │  │
│ API Keys│  │                                         │  │
│         │  │  [Connect Claude] [Connect Gemini] ...  │  │
│ [📁]    │  └─────────────────────────────────────────┘  │
│ Auth    │                                                │
│ Files   │                                                │
│         │                                                │
│ [📊]    │                                                │
│ Logs    │                                                │
│         │                                                │
│ [📈]    │                                                │
│ Analytics│                                               │
└─────────┴────────────────────────────────────────────────┘
```

### Sidebar Navigation

| Icon | Trang | Mô tả |
|------|-------|-------|
| ⌂ | **Dashboard** | Tổng quan, start/stop proxy, providers status |
| ⚙ | **Settings** | Cấu hình port, auto-start, model mappings |
| 🔑 | **API Keys** | Quản lý API keys (Gemini, Claude, Codex, OpenAI) |
| 📁 | **Auth Files** | Xem các file xác thực OAuth |
| 📊 | **Logs** | Request logs real-time |
| 📈 | **Analytics** | Thống kê usage, tokens, costs |

---

## 3. Kết Nối Providers

### Tổng Quan Providers

| Provider | Phương thức | Subscription cần |
|----------|-------------|------------------|
| **Claude** | OAuth | Claude Pro/Team |
| **ChatGPT** | OAuth | ChatGPT Plus |
| **Gemini** | OAuth | Gemini Advanced |
| **GitHub Copilot** | OAuth + Settings | Copilot Individual/Business |
| **Qwen** | OAuth | Qwen account |
| **Azure AI Foundry** | API Key | Azure subscription |
| **Custom Providers** | API Key | Tùy provider |

### Claude

#### Bước 1: Click Connect

Dashboard → Dòng "Claude" → Click **"Connect"**

#### Bước 2: OAuth Login

1. Cửa sổ trình duyệt mở ra
2. Đăng nhập với tài khoản Claude Pro/Team
3. Click **"Authorize"** để cấp quyền cho ProxyPal

#### Bước 3: Xác nhận

- Status chuyển thành ✓ (màu xanh)
- Badge "Connected" hiển thị

### ChatGPT/OpenAI

#### Bước 1: Click Connect

Dashboard → Dòng "ChatGPT" → Click **"Connect"**

#### Bước 2: OAuth Login

1. Đăng nhập tài khoản OpenAI (ChatGPT Plus)
2. Authorize cho ProxyPal

#### Bước 3: Xác nhận

- Status ✓ Connected

### Gemini

#### Bước 1: Click Connect

Dashboard → Dòng "Gemini" → Click **"Connect"**

#### Bước 2: Google OAuth

1. Chọn tài khoản Google có Gemini Advanced
2. Cho phép truy cập

#### Bước 3: Xác nhận

- Status ✓ Connected

### GitHub Copilot

GitHub Copilot yêu cầu cấu hình thêm trong Settings:

#### Bước 1: Enable Copilot

1. Settings → Section **"GitHub Copilot"**
2. Bật toggle **"Enable Copilot"**

#### Bước 2: Chọn Account Type

| Type | Mô tả |
|------|-------|
| **Individual** | Tài khoản cá nhân ($10/tháng) |
| **Business** | Tài khoản tổ chức |

#### Bước 3: Authenticate

1. Click **"Authenticate"**
2. Đăng nhập GitHub trong browser
3. Authorize cho ProxyPal

#### Bước 4: Xác nhận

- Dashboard hiển thị Copilot ✓ Connected

### Qwen

#### Bước 1: Click Connect

Dashboard → Dòng "Qwen" → Click **"Connect"**

#### Bước 2: OAuth

1. Đăng nhập tài khoản Qwen
2. Authorize

### Azure AI Foundry (Claude)

Xem chi tiết tại: [Custom Providers](./CUSTOM_PROVIDERS.md#azure-ai-foundry)

---

## 4. Start Proxy

### Cách 1: Tự Động

Nếu **Auto Start** được bật (Settings → General):
- Proxy tự động start khi mở app
- Status bar hiển thị: `Proxy Running on :8317`

### Cách 2: Thủ Công

1. Dashboard → Click nút **"Start Proxy"**
2. Đợi vài giây để khởi động
3. Status chuyển thành **"Running"**

### Thông Tin Endpoint

```
┌─────────────────────────────────────────────────────────┐
│  Proxy Status: ● Running                                │
│                                                         │
│  Local Endpoint: http://localhost:8317/v1               │
│  [Copy]                                                 │
│                                                         │
│  [Stop Proxy]                                           │
└─────────────────────────────────────────────────────────┘
```

| Endpoint | Mục đích |
|----------|----------|
| `http://localhost:8317/v1` | OpenAI-compatible API |
| `http://localhost:8317` | Anthropic-compatible API |
| `http://localhost:8317/v1/models` | Danh sách models |

### Đổi Port

1. Settings → Section **"General"**
2. Thay đổi **"Proxy Port"** (mặc định: 8317)
3. Restart proxy để áp dụng

---

## 5. Kiểm Tra Kết Nối

### Test Endpoint

```bash
# Liệt kê models có sẵn
curl http://localhost:8317/v1/models \
  -H "Authorization: Bearer proxypal-local"

# Kết quả thành công:
{
  "data": [
    {"id": "claude-opus-4-5-20251101", "object": "model"},
    {"id": "claude-sonnet-4-5-20250929", "object": "model"},
    {"id": "gemini-2.5-pro", "object": "model"},
    ...
  ]
}
```

### Test Chat Completion

```bash
curl http://localhost:8317/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer proxypal-local" \
  -d '{
    "model": "claude-sonnet-4-5-20250929",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 100
  }'
```

### Test Anthropic Format

```bash
curl http://localhost:8317/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: proxypal-local" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-sonnet-4-5-20250929",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 100
  }'
```

### Xem Logs

Logs page hiển thị real-time:

```
[2024-12-20 10:30:15] INFO  POST /v1/chat/completions
[2024-12-20 10:30:15] DEBUG Model: claude-sonnet-4-5-20250929 → Provider: Claude OAuth
[2024-12-20 10:30:16] INFO  Response: 200 OK (1.2s)
```

---

## 6. Các Trang Chính

### Dashboard

| Thành phần | Chức năng |
|------------|-----------|
| **Proxy Status** | Trạng thái hiện tại (Running/Stopped) |
| **Start/Stop Button** | Điều khiển proxy |
| **Endpoint Info** | Copy URL endpoint |
| **Providers Grid** | Trạng thái từng provider |
| **Connect Buttons** | Kết nối OAuth |

### Settings

| Section | Cấu hình |
|---------|----------|
| **General** | Port, Auto Start, Launch at Login |
| **Debug** | Debug Mode, Request Logging |
| **Model Mappings** | Route model A → model B |
| **Server Mode** | Chia sẻ proxy với máy remote |
| **Amp Code** | Integration với Amp Code |
| **GitHub Copilot** | Enable/Auth Copilot |

### API Keys

| Tab | Mục đích |
|-----|----------|
| **Gemini** | API keys cho Gemini |
| **Claude** | API keys cho Claude (bao gồm Azure AI Foundry) |
| **Codex** | API keys cho Codex/OpenAI |
| **OpenAI Compatible** | Custom providers (OpenRouter, etc.) |

### Auth Files

Xem các file xác thực OAuth đã lưu:

| File | Provider |
|------|----------|
| `claude-oauth.json` | Claude OAuth tokens |
| `openai-oauth.json` | OpenAI/ChatGPT tokens |
| `gemini-oauth.json` | Gemini tokens |
| `copilot-oauth.json` | GitHub Copilot tokens |

### Logs

| Filter | Hiển thị |
|--------|----------|
| **ALL** | Tất cả logs |
| **INFO** | Thông tin chung |
| **DEBUG** | Chi tiết debug |
| **ERROR** | Chỉ lỗi |
| **WARN** | Cảnh báo |

### Analytics

| Metric | Mô tả |
|--------|-------|
| **Total Requests** | Tổng số requests |
| **Tokens Used** | Tokens đã sử dụng |
| **Cost Estimate** | Chi phí ước tính |
| **By Provider** | Phân tích theo provider |
| **By Model** | Phân tích theo model |

---

## 7. Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Mở Command Palette |
| `Ctrl+,` | Mở Settings |
| `Ctrl+L` | Mở Logs |
| `Ctrl+1` | Chuyển đến Dashboard |
| `Ctrl+2` | Chuyển đến Settings |
| `Ctrl+3` | Chuyển đến API Keys |
| `Ctrl+4` | Chuyển đến Logs |
| `Ctrl+5` | Chuyển đến Analytics |

### Command Palette

Press `Ctrl+K` để mở, sau đó gõ:

| Command | Action |
|---------|--------|
| `start` | Start proxy |
| `stop` | Stop proxy |
| `restart` | Restart proxy |
| `settings` | Mở Settings |
| `logs` | Mở Logs |
| `connect claude` | Connect Claude OAuth |
| `connect gemini` | Connect Gemini OAuth |

---

## Tiếp Theo

- [Cấu Hình Coding Tools](./CAU_HINH_TOOLS.md) - Thiết lập Cursor, Continue, Claude Code, Cline
- [Model Mapping](./MODEL_MAPPING.md) - Route requests giữa các models
- [Server Mode](./SERVER_MODE.md) - Chia sẻ với máy remote

---

## Tham Khảo

- [Cài Đặt](./CAI_DAT.md)
- [Xử Lý Lỗi](./TROUBLESHOOTING.md)
- [README](../README.md)
