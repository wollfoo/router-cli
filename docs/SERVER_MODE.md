# Server Mode - Chia sẻ ProxyPal với Máy Remote

Hướng dẫn chi tiết biến ProxyPal thành **central model router** cho nhiều máy trong mạng hoặc qua internet.

---

## Mục Lục

1. [Tổng Quan](#1-tổng-quan)
2. [Bật Server Mode](#2-bật-server-mode)
3. [Phương Thức Kết Nối](#3-phương-thức-kết-nối)
4. [Tailscale Funnel (Khuyên Dùng)](#4-tailscale-funnel-khuyên-dùng)
5. [Cấu Hình Máy Remote](#5-cấu-hình-máy-remote)
6. [Models Có Sẵn](#6-models-có-sẵn)
7. [Use Cases](#7-use-cases)
8. [Bảo Mật](#8-bảo-mật)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Tổng Quan

### Server Mode Là Gì?

Server Mode biến ProxyPal thành **router trung tâm** cho nhiều máy:

```
┌─────────────────────────────────────────────────────────────┐
│                    ProxyPal Server                          │
│                    (Máy chính - 10.0.0.4)                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Providers đã kết nối:                              │    │
│  │  • Claude OAuth ✓                                   │    │
│  │  • Gemini OAuth ✓                                   │    │
│  │  • Azure AI Foundry ✓                               │    │
│  │  • GitHub Copilot ✓                                 │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────┬───────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Laptop 1 │    │ Laptop 2 │    │ VPS/VM   │
    │ (LAN)    │    │ (Remote) │    │ (Cloud)  │
    └──────────┘    └──────────┘    └──────────┘
```

### Lợi Ích

| Lợi ích | Mô tả |
|---------|-------|
| **1 OAuth, nhiều máy** | Đăng nhập OAuth 1 lần, dùng mọi nơi |
| **Quản lý tập trung** | Thêm/xóa providers tại 1 điểm |
| **Tiết kiệm quota** | Model mapping để dùng model rẻ hơn |
| **Không cần VPN** | Tailscale Funnel expose qua HTTPS |
| **Team sharing** | Chia sẻ với đồng nghiệp |

---

## 2. Bật Server Mode

### Bước 1: Mở Settings

- Sidebar → **Settings**
- Hoặc `Ctrl+,`

### Bước 2: Enable Server Mode

1. Scroll đến section **"Server Mode"**
2. Bật toggle **"Enable Server Mode"**
3. Sẽ hiển thị:
   - **Remote Endpoints**: Các IP của máy (VD: `http://10.0.0.4:8317/v1`)
   - **Remote API Key**: Key xác thực cho máy remote

### Bước 3: Copy Thông Tin

| Thông tin | Mục đích | Ví dụ |
|-----------|----------|-------|
| **Remote Endpoint** | URL cho máy remote | `http://10.0.0.4:8317/v1` |
| **Remote API Key** | Xác thực requests | `proxypal-remote-a1b2c3d4` |

### Bước 4: Restart Proxy

**Quan trọng:** Phải restart proxy để áp dụng Server Mode.

- Click **Stop Proxy** → **Start Proxy**
- Hoặc toggle proxy off/on

---

## 3. Phương Thức Kết Nối

### So Sánh Các Phương Thức

| Phương thức | URL | Khi nào dùng | Yêu cầu |
|-------------|-----|--------------|---------|
| **LAN IP** | `http://192.168.x.x:8317` | Cùng mạng WiFi/Ethernet | Mở firewall |
| **Tailscale IP** | `http://100.x.x.x:8317` | Cả 2 máy có Tailscale | Cài Tailscale |
| **Tailscale Funnel** | `https://xxx.ts.net` | Máy remote không có Tailscale | Chỉ server cài |
| **Port Forwarding** | `http://public-ip:8317` | Có quyền router | Cấu hình router |

### Khuyến Nghị

```
┌─────────────────────────────────────────────────────────────┐
│  Cùng mạng LAN?                                             │
│      │                                                      │
│      ├── Yes → Dùng LAN IP (192.168.x.x)                    │
│      │                                                      │
│      └── No → Cả 2 máy có Tailscale?                        │
│               │                                             │
│               ├── Yes → Dùng Tailscale IP (100.x.x.x)       │
│               │         (Nhanh hơn, peer-to-peer)           │
│               │                                             │
│               └── No → Dùng Tailscale Funnel                │
│                        (HTTPS, không cần cài Tailscale      │
│                         trên máy remote)                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Tailscale Funnel (Khuyên Dùng)

### Tại Sao Dùng Tailscale Funnel?

| Tính năng | Giá trị |
|-----------|---------|
| **Domain** | Miễn phí subdomain `*.ts.net` cố định vĩnh viễn |
| **HTTPS** | Tự động, certificate miễn phí |
| **Bandwidth** | Không giới hạn |
| **Setup** | ~2 phút |
| **Chi phí** | **$0** |
| **Không cần** | Domain riêng, cấu hình router, VPN |

### So Sánh Với Các Giải Pháp Khác

| Giải pháp | Bandwidth | Domain cố định | Chi phí/năm |
|-----------|-----------|----------------|-------------|
| **Tailscale Funnel** | Unlimited | ✅ `*.ts.net` | **$0** |
| **Cloudflare Tunnel** | Unlimited | ✅ Cần domain | ~$10 |
| **ngrok Free** | 1GB/tháng | ❌ Thay đổi | $0 |
| **ngrok Pro** | Unlimited | ✅ | $96 |
| **Zrok** | 250MB/tháng | ✅ `*.zrok.io` | $0 |

### Cài Đặt Tailscale

#### Windows

```powershell
# Cách 1: winget (khuyên dùng)
winget install tailscale.tailscale

# Cách 2: Tải từ website
# https://tailscale.com/download/windows
```

#### macOS

```bash
# Homebrew
brew install --cask tailscale

# Hoặc tải từ App Store
```

#### Linux

```bash
# Ubuntu/Debian
curl -fsSL https://tailscale.com/install.sh | sh

# Arch
sudo pacman -S tailscale
```

### Đăng Nhập Tailscale

```powershell
# Đăng nhập (miễn phí với Google/Microsoft/GitHub)
tailscale up

# Kiểm tra status
tailscale status
```

### Chạy Funnel

#### Windows

```powershell
# Foreground (tắt khi đóng terminal)
tailscale funnel 8317

# Background (tiếp tục chạy) - KHUYÊN DÙNG
tailscale funnel --bg 8317

# Nếu không tìm thấy tailscale trong PATH
& 'C:\Program Files\Tailscale\tailscale.exe' funnel --bg 8317
```

#### macOS/Linux

```bash
# Background mode
tailscale funnel --bg 8317

# Hoặc với sudo nếu cần
sudo tailscale funnel --bg 8317
```

### Kết Quả

Sau khi chạy, bạn sẽ nhận domain dạng:
```
https://your-pc-name.tail12345.ts.net
```

Domain này **cố định vĩnh viễn** và tự động có HTTPS.

### Kiểm Tra Funnel Status

```powershell
# Kiểm tra status
tailscale funnel status

# Kết quả khi đang chạy:
# Available on the internet:
#
# https://your-pc.tail12345.ts.net/
# |-- proxy http://127.0.0.1:8317

# Kết quả khi chưa chạy:
# No serve config
```

### Tắt Funnel

```powershell
tailscale funnel off
```

### Giữ Funnel Chạy Liên Tục

#### Cách 1: Background Mode (Khuyên dùng)

```powershell
tailscale funnel --bg 8317
```

Funnel sẽ tiếp tục chạy sau khi đóng terminal và restart máy.

#### Cách 2: Windows Scheduled Task

```powershell
# Tạo task tự động chạy khi login
$action = New-ScheduledTaskAction -Execute "C:\Program Files\Tailscale\tailscale.exe" -Argument "funnel --bg 8317"
$trigger = New-ScheduledTaskTrigger -AtLogon
$principal = New-ScheduledTaskPrincipal -UserId $env:USERNAME -RunLevel Highest
Register-ScheduledTask -TaskName "ProxyPal Tailscale Funnel" -Action $action -Trigger $trigger -Principal $principal
```

#### Cách 3: systemd (Linux)

```bash
# /etc/systemd/system/proxypal-funnel.service
[Unit]
Description=ProxyPal Tailscale Funnel
After=tailscaled.service

[Service]
ExecStart=/usr/bin/tailscale funnel 8317
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable proxypal-funnel
sudo systemctl start proxypal-funnel
```

### Lấy Tailscale IP (cho peer-to-peer)

Nếu cả 2 máy đều cài Tailscale, dùng IP trực tiếp nhanh hơn:

```powershell
# Lấy IPv4
tailscale ip -4
# Kết quả: 100.x.x.x

# Hoặc full path Windows
& 'C:\Program Files\Tailscale\tailscale.exe' ip -4
```

---

## 5. Cấu Hình Máy Remote

### Environment Variables

#### Claude Code / Aider

```bash
# Qua Tailscale Funnel (HTTPS)
export ANTHROPIC_BASE_URL="https://your-pc.tail12345.ts.net"
export ANTHROPIC_API_KEY="proxypal-remote-xxxxx"

# Qua Tailscale IP (nhanh hơn, cần Tailscale trên remote)
export ANTHROPIC_BASE_URL="http://100.x.x.x:8317"
export ANTHROPIC_API_KEY="proxypal-remote-xxxxx"

# Qua LAN (cùng mạng)
export ANTHROPIC_BASE_URL="http://192.168.1.100:8317"
export ANTHROPIC_API_KEY="proxypal-remote-xxxxx"
```

#### Model Routing (Optional)

```bash
# Route tất cả models về model cụ thể
export ANTHROPIC_DEFAULT_OPUS_MODEL="gemini-2.5-pro"
export ANTHROPIC_DEFAULT_SONNET_MODEL="gemini-2.5-flash"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="gemini-2.5-flash-lite"

# Hoặc chỉ định model cụ thể
export ANTHROPIC_MODEL="claude-opus-4-5-20251101"
```

#### OpenAI-Compatible Clients (Cursor, Continue, Cline)

```bash
# Qua Tailscale Funnel
export OPENAI_BASE_URL="https://your-pc.tail12345.ts.net/v1"
export OPENAI_API_KEY="proxypal-remote-xxxxx"

# Qua LAN
export OPENAI_BASE_URL="http://192.168.1.100:8317/v1"
export OPENAI_API_KEY="proxypal-remote-xxxxx"
```

### Windows PowerShell (Máy Remote)

```powershell
# Session-only
$env:ANTHROPIC_BASE_URL = "https://your-pc.tail12345.ts.net"
$env:ANTHROPIC_API_KEY = "proxypal-remote-xxxxx"

# Permanent (User level)
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://your-pc.tail12345.ts.net", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "proxypal-remote-xxxxx", "User")
```

### Cấu Hình Cursor (Máy Remote)

```
Settings → Models → OpenAI API Base URL
URL: https://your-pc.tail12345.ts.net/v1
API Key: proxypal-remote-xxxxx
```

### Cấu Hình Continue (Máy Remote)

```json
// ~/.continue/config.json
{
  "models": [{
    "provider": "openai",
    "model": "claude-sonnet-4-5-20250929",
    "apiBase": "https://your-pc.tail12345.ts.net/v1",
    "apiKey": "proxypal-remote-xxxxx"
  }]
}
```

### Test Kết Nối

```bash
# Test từ máy remote
curl https://your-pc.tail12345.ts.net/v1/models \
  -H "Authorization: Bearer proxypal-remote-xxxxx"

# Kết quả thành công: JSON danh sách models
# {"data":[{"id":"claude-opus-4-5-20251101",...}]}
```

---

## 6. Models Có Sẵn

Models phụ thuộc vào providers đã cấu hình trên máy server:

### Danh Sách Models (Cập nhật 2025-12-19)

| Provider | Số lượng | Models |
|----------|----------|--------|
| **Claude OAuth/API** | 3 | `claude-opus-4-5-20251101`, `claude-sonnet-4-5-20250929`, `claude-haiku-4-5` |
| **Gemini OAuth** | 3 | `gemini-2.5-pro`, `gemini-2.5-flash`, `gemini-2.5-flash-lite` |
| **Antigravity** | 10 | `gemini-2.5-flash`, `gemini-2.5-flash-lite`, `gemini-2.5-computer-use-preview-10-2025`, `gemini-3-pro-preview`, `gemini-3-pro-image-preview`, `gemini-3-flash-preview`, `gemini-claude-sonnet-4-5`, `gemini-claude-sonnet-4-5-thinking`, `gemini-claude-opus-4-5-thinking`, `gpt-oss-120b-medium` |
| **Codex OAuth** | 8 | `gpt-5`, `gpt-5-codex`, `gpt-5.1`, `gpt-5.1-codex`, `gpt-5.1-codex-mini`, `gpt-5.1-codex-max`, `gpt-5.2` |
| **GitHub Copilot** | 23 | `gpt-4`, `gpt-4.1`, `gpt-4o`, `gpt-4-turbo`, `gpt-5`, `gpt-5-mini`, `gpt-5-codex`, `gpt-5.1`, `gpt-5.1-codex`, `gpt-5.1-codex-mini`, `gpt-5.1-codex-max`, `gpt-5.2`, `o1`, `o1-mini`, `grok-code-fast-1`, `raptor-mini`, `gemini-2.5-pro`, `gemini-3-pro-preview`, `claude-haiku-4.5`, `claude-opus-4.1`, `claude-sonnet-4`, `claude-sonnet-4.5`, `claude-opus-4.5` |

### Xem Models Thực Tế

1. Mở ProxyPal → **Logs** page
2. Filter: **DEBUG**
3. Start proxy → xem log "Available models"

### Model Mapping

Route model A → model B trong **Settings → Model Mappings**:

| From (Client gọi) | To (Thực tế dùng) | Lý do |
|-------------------|-------------------|-------|
| `claude-opus-4-5-20251101` | `gemini-2.5-pro` | Tiết kiệm quota Claude |
| `gpt-4o` | `claude-sonnet-4-5-20250929` | Prefer Claude |
| `claude-haiku-4-5` | `gemini-2.5-flash-lite` | Nhanh hơn |

---

## 7. Use Cases

### Use Case 1: Laptop + Desktop

```
Desktop (có Claude Pro subscription)
└── ProxyPal Server Mode
    └── Tailscale Funnel: https://desktop.tail123.ts.net

Laptop (di động)
└── ANTHROPIC_BASE_URL=https://desktop.tail123.ts.net
└── Dùng Claude Code mọi nơi có internet
```

### Use Case 2: Team Development

```
Server trung tâm (1 Claude Team subscription)
└── ProxyPal Server Mode
└── API Keys: team-member-1, team-member-2, ...

Team members
└── Mỗi người config ANTHROPIC_BASE_URL + API key riêng
└── Quota tracking qua Analytics page
```

### Use Case 3: Cloud VPS

```
Local PC (có OAuth accounts)
└── ProxyPal Server Mode
└── Tailscale Funnel

Cloud VPS (AWS/GCP/Azure)
└── SSH vào VPS
└── export ANTHROPIC_BASE_URL=https://local-pc.ts.net
└── Chạy Claude Code trên cloud
```

### Use Case 4: CI/CD Pipeline

```yaml
# .github/workflows/ai-review.yml
env:
  ANTHROPIC_BASE_URL: ${{ secrets.PROXYPAL_URL }}
  ANTHROPIC_API_KEY: ${{ secrets.PROXYPAL_KEY }}

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - name: AI Code Review
        run: |
          claude-code review --auto
```

---

## 8. Bảo Mật

### Best Practices

| Khuyến nghị | Chi tiết |
|-------------|----------|
| **API Key riêng** | Tạo key riêng cho từng máy/người dùng |
| **Rotate keys** | Đổi key định kỳ hoặc khi nghi ngờ lộ |
| **HTTPS** | Luôn dùng Tailscale Funnel (HTTPS) cho remote |
| **Firewall** | Chỉ mở port 8317 cho IP cần thiết |
| **Logging** | Bật Request Logging để monitor usage |

### Quản Lý API Keys

1. **Generate new key**: Settings → Server Mode → Click refresh icon
2. **Revoke key**: Generate key mới, key cũ tự động invalid
3. **Multiple keys**: Hiện tại 1 key/server, tương lai sẽ hỗ trợ nhiều keys

### Firewall Rules

#### Windows

```powershell
# Cho phép port 8317 từ mạng nội bộ
New-NetFirewallRule -DisplayName "ProxyPal LAN" -Direction Inbound -Protocol TCP -LocalPort 8317 -RemoteAddress 192.168.0.0/16 -Action Allow

# Cho phép từ Tailscale network
New-NetFirewallRule -DisplayName "ProxyPal Tailscale" -Direction Inbound -Protocol TCP -LocalPort 8317 -RemoteAddress 100.64.0.0/10 -Action Allow

# Block tất cả nguồn khác (mặc định)
```

#### Linux

```bash
# UFW
sudo ufw allow from 192.168.0.0/16 to any port 8317
sudo ufw allow from 100.64.0.0/10 to any port 8317

# iptables
sudo iptables -A INPUT -p tcp --dport 8317 -s 192.168.0.0/16 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8317 -s 100.64.0.0/10 -j ACCEPT
```

---

## 9. Troubleshooting

### Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Giải pháp |
|-----|-------------|-----------|
| `Connection refused` | Proxy không chạy hoặc Server Mode chưa bật | Start proxy + Enable Server Mode |
| `401 Unauthorized` | API key sai | Kiểm tra lại Remote API Key |
| `ECONNRESET` | Funnel đã tắt | Chạy lại `tailscale funnel --bg 8317` |
| `No serve config` | Funnel chưa bật | Chạy `tailscale funnel --bg 8317` |
| `tailscale: command not found` | Tailscale không trong PATH | Dùng full path: `& 'C:\Program Files\Tailscale\tailscale.exe'` |
| `Funnel needs approval` | Admin chưa approve | Vào [Tailscale Admin Console](https://login.tailscale.com/admin) → approve |
| `timeout` | Firewall block | Mở port 8317 |
| `no auth available` | Không có provider nào connected | Connect ít nhất 1 provider |

### Debug Steps

#### Step 1: Kiểm tra ProxyPal

```powershell
# Local test
curl http://localhost:8317/v1/models -H "Authorization: Bearer proxypal-local"

# Nếu fail → Proxy chưa start hoặc port sai
```

#### Step 2: Kiểm tra Server Mode

- Settings → Server Mode → Toggle phải ON
- Remote Endpoint phải hiển thị IP
- Đã restart proxy sau khi enable?

#### Step 3: Kiểm tra Firewall

```powershell
# Test từ máy khác trong LAN
curl http://192.168.x.x:8317/v1/models -H "Authorization: Bearer proxypal-remote-xxx"

# Nếu timeout → Firewall chưa mở
```

#### Step 4: Kiểm tra Tailscale Funnel

```powershell
# Kiểm tra funnel status
tailscale funnel status

# Nếu "No serve config" → chạy:
tailscale funnel --bg 8317

# Test từ internet
curl https://your-pc.ts.net/v1/models -H "Authorization: Bearer proxypal-remote-xxx"
```

#### Step 5: Kiểm tra Logs

- ProxyPal → Logs page → Filter ERROR
- Xem chi tiết request failures

### Reset Server Mode

Nếu gặp vấn đề không giải quyết được:

1. Tắt Server Mode
2. Stop proxy
3. Restart ProxyPal app
4. Start proxy
5. Bật Server Mode
6. Generate new API key
7. Restart proxy

---

## Tham Khảo

- [Tailscale Funnel Docs](https://tailscale.com/kb/1223/funnel/)
- [ProxyPal README](../README.md)
- [Hướng Dẫn Sử Dụng](./HUONG_DAN_SU_DUNG.md)
- [Ví Dụ Endpoint](./VI_DU_ENDPOINT.md)
