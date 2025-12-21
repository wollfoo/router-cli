# Xử Lý Lỗi (Troubleshooting)

Hướng dẫn chi tiết xử lý các lỗi thường gặp khi sử dụng ProxyPal.

---

## Mục Lục

1. [Lỗi Khởi Động](#1-lỗi-khởi-động)
2. [Lỗi Kết Nối Providers](#2-lỗi-kết-nối-providers)
3. [Lỗi Proxy](#3-lỗi-proxy)
4. [Lỗi API Requests](#4-lỗi-api-requests)
5. [Lỗi Model Mapping](#5-lỗi-model-mapping)
6. [Lỗi Azure AI Foundry](#6-lỗi-azure-ai-foundry)
7. [Lỗi Server Mode](#7-lỗi-server-mode)
8. [Config Files](#8-config-files)
9. [Debug Mode](#9-debug-mode)
10. [Reset và Recovery](#10-reset-và-recovery)

---

## 1. Lỗi Khởi Động

### App Không Mở

#### Triệu chứng

- Click icon không có gì xảy ra
- App crash ngay khi mở

#### Nguyên nhân & Giải pháp

| Nguyên nhân | Giải pháp |
|-------------|-----------|
| WebView2 thiếu (Windows) | Cài [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) |
| File corrupt | Reinstall ProxyPal |
| Antivirus block | Thêm ProxyPal vào exceptions |
| Config file corrupt | Xóa `config.json`, restart |

#### Windows: Kiểm tra WebView2

```powershell
# Kiểm tra WebView2 đã cài
Get-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\EdgeUpdate\Clients\{F3017226-FE2A-4295-8BDF-00C3A9A7E4C5}' -ErrorAction SilentlyContinue

# Nếu không có → cài đặt
winget install Microsoft.EdgeWebView2Runtime
```

### Lỗi "Failed to load config"

#### Giải pháp

1. Backup config hiện tại:

```powershell
# Windows
Copy-Item "$env:APPDATA\proxypal\config.json" "$env:APPDATA\proxypal\config.backup.json"
```

2. Xóa config:

```powershell
Remove-Item "$env:APPDATA\proxypal\config.json"
```

3. Restart app (sẽ tạo config mới)

---

## 2. Lỗi Kết Nối Providers

### OAuth Không Hoạt Động

#### Triệu chứng

- Click "Connect" không mở browser
- Browser mở nhưng không redirect về app

#### Giải pháp

| Bước | Hành động |
|------|-----------|
| 1 | Đảm bảo proxy đang chạy (port 8317) |
| 2 | Kiểm tra browser không block popups |
| 3 | Thử browser khác (Chrome, Firefox) |
| 4 | Disconnect → Connect lại |
| 5 | Restart app |

#### Windows: Kiểm tra URL Protocol Handler

```powershell
# Kiểm tra proxypal:// protocol
Get-ItemProperty -Path 'HKCU:\SOFTWARE\Classes\proxypal' -ErrorAction SilentlyContinue
```

### GitHub Copilot Authentication Failed

#### Triệu chứng

```
Error: Authentication failed for GitHub Copilot
```

#### Giải pháp

1. Settings → GitHub Copilot → Disable
2. Settings → GitHub Copilot → Enable
3. Click "Authenticate" lại
4. Đảm bảo GitHub account có Copilot subscription

### "no auth available"

#### Triệu chứng

```json
{"error": "no auth available"}
```

#### Nguyên nhân

- Không có provider nào connected
- OAuth tokens expired
- Model không có sẵn

#### Giải pháp

1. Dashboard → Kiểm tra ít nhất 1 provider ✓ Connected
2. Nếu expired → Disconnect → Connect lại
3. Kiểm tra model có trong `/v1/models`

---

## 3. Lỗi Proxy

### Port Đã Được Sử Dụng

#### Triệu chứng

```
Error: Port 8317 is already in use
```

#### Giải pháp

```powershell
# Kiểm tra process đang dùng port
netstat -ano | findstr :8317

# Tìm tên process
Get-Process -Id <PID>

# Kill process
taskkill /PID <PID> /F
```

Hoặc đổi port trong Settings → General → Proxy Port.

### Proxy Không Start

#### Triệu chứng

- Click "Start Proxy" không có gì xảy ra
- Status vẫn "Stopped"

#### Giải pháp

| Bước | Hành động |
|------|-----------|
| 1 | Kiểm tra port chưa bị dùng |
| 2 | Kiểm tra firewall |
| 3 | Restart app |
| 4 | Xem Logs page |

#### Firewall Windows

```powershell
# Kiểm tra rule
Get-NetFirewallRule -DisplayName "ProxyPal*"

# Thêm rule mới
New-NetFirewallRule -DisplayName "ProxyPal" -Direction Inbound -Protocol TCP -LocalPort 8317 -Action Allow
```

### CLIProxyAPI Crash

#### Triệu chứng

- Proxy start rồi tự stop
- Logs hiển thị "CLIProxyAPI exited"

#### Giải pháp

1. Kiểm tra `proxy-config.yaml` valid:

```powershell
# Windows
notepad "$env:APPDATA\proxypal\proxy-config.yaml"
```

2. Xóa và regenerate:

```powershell
Remove-Item "$env:APPDATA\proxypal\proxy-config.yaml"
# Restart app, start proxy
```

---

## 4. Lỗi API Requests

### Connection Refused

#### Triệu chứng

```
curl: (7) Failed to connect to localhost port 8317
```

#### Giải pháp

1. Kiểm tra proxy đang chạy (Dashboard → Status: Running)
2. Kiểm tra đúng port (mặc định 8317)
3. Thử `127.0.0.1` thay vì `localhost`

### 401 Unauthorized

#### Triệu chứng

```json
{"error": {"message": "Unauthorized", "code": 401}}
```

#### Giải pháp

| Nguyên nhân | Giải pháp |
|-------------|-----------|
| API key sai | Dùng `proxypal-local` cho localhost |
| Server Mode nhưng dùng local key | Dùng Remote API Key |
| Header sai | `Authorization: Bearer <key>` hoặc `x-api-key: <key>` |

### Model Not Found

#### Triệu chứng

```json
{"error": {"message": "Model 'xxx' not found"}}
```

#### Giải pháp

1. Liệt kê models có sẵn:

```bash
curl http://localhost:8317/v1/models
```

2. Kiểm tra tên model chính xác (case-sensitive)
3. Kiểm tra provider của model đã connected

### Request Timeout

#### Triệu chứng

- Request treo lâu
- Error sau vài phút

#### Giải pháp

1. Kiểm tra internet connection
2. Tăng timeout trong client
3. Thử model khác (có thể provider gặp vấn đề)
4. Xem Logs để biết request đến đâu

---

## 5. Lỗi Model Mapping

### Mapping Không Hoạt Động

#### Triệu chứng

- Request vẫn dùng model gốc
- Mapping không được apply

#### Giải pháp

| Bước | Hành động |
|------|-----------|
| 1 | Kiểm tra mapping `enabled: true` |
| 2 | Kiểm tra tên model chính xác (case-sensitive) |
| 3 | Restart proxy sau khi thay đổi |
| 4 | Bật Debug Mode xem logs |

### Model Đích Không Tồn Tại

#### Triệu chứng

```json
{"error": "Target model 'xxx' not available"}
```

#### Giải pháp

1. Kiểm tra model đích có trong `/v1/models`
2. Kiểm tra provider của model đã connected

---

## 6. Lỗi Azure AI Foundry

### DeploymentNotFound

#### Triệu chứng

```json
{"error": {"code": "DeploymentNotFound", "message": "..."}}
```

#### Nguyên nhân

Model name trong request không khớp deployment name trên Azure.

#### Giải pháp

1. Azure Portal → AI Foundry → Deployments
2. Xem deployment name thực tế (VD: `claude-opus-4-5`)
3. ProxyPal → API Keys → Claude → Add model aliasing:
   - Alias: `claude-opus-4-5-20251101` (client gọi)
   - Name: `claude-opus-4-5` (trên Azure)

### InvalidApiKey

#### Triệu chứng

```json
{"error": {"code": "InvalidApiKey", "message": "..."}}
```

#### Giải pháp

1. Azure Portal → AI Services → Keys and Endpoint
2. Copy Key 1 hoặc Key 2
3. Update trong ProxyPal → API Keys → Claude

### InvalidEndpoint

#### Triệu chứng

```json
{"error": {"code": "InvalidEndpoint", "message": "..."}}
```

#### Giải pháp

Kiểm tra Base URL format:

```
✓ https://your-resource.services.ai.azure.com/anthropic
✗ https://your-resource.services.ai.azure.com/anthropic/v1  (sai)
✗ https://your-resource.services.ai.azure.com  (thiếu /anthropic)
```

---

## 7. Lỗi Server Mode

### Remote Không Kết Nối Được

#### Triệu chứng

- `Connection refused` từ máy remote
- Timeout

#### Giải pháp

| Bước | Hành động |
|------|-----------|
| 1 | Kiểm tra Server Mode đã bật |
| 2 | Kiểm tra đã restart proxy sau khi bật |
| 3 | Kiểm tra firewall cho phép port 8317 |
| 4 | Test từ máy server trước |

#### Test từ máy server

```bash
curl http://localhost:8317/v1/models
```

#### Test từ LAN

```bash
curl http://192.168.x.x:8317/v1/models -H "Authorization: Bearer proxypal-remote-xxxxx"
```

### Tailscale Funnel Không Hoạt Động

#### Triệu chứng

```
No serve config
```

#### Giải pháp

```powershell
# Kiểm tra Tailscale status
tailscale status

# Chạy funnel
tailscale funnel --bg 8317

# Kiểm tra funnel status
tailscale funnel status
```

Xem chi tiết: [SERVER_MODE.md](./SERVER_MODE.md#9-troubleshooting)

---

## 8. Config Files

### Vị Trí Files

| File | Windows | macOS/Linux |
|------|---------|-------------|
| **config.json** | `%APPDATA%\proxypal\` | `~/.config/proxypal/` |
| **proxy-config.yaml** | `%APPDATA%\proxypal\` | `~/.config/proxypal/` |
| **Amp settings** | `%USERPROFILE%\.config\amp\` | `~/.config/amp/` |

### Backup Config

```powershell
# Windows
$backupDir = "$env:APPDATA\proxypal-backup"
New-Item -ItemType Directory -Force -Path $backupDir
Copy-Item "$env:APPDATA\proxypal\*" $backupDir -Recurse
```

```bash
# macOS/Linux
cp -r ~/.config/proxypal ~/.config/proxypal-backup
```

### Restore Config

```powershell
# Windows
Copy-Item "$env:APPDATA\proxypal-backup\*" "$env:APPDATA\proxypal\" -Force
```

### Validate Config JSON

```powershell
# Kiểm tra JSON valid
Get-Content "$env:APPDATA\proxypal\config.json" | ConvertFrom-Json
```

Nếu lỗi → config JSON không valid → sửa hoặc xóa.

---

## 9. Debug Mode

### Bật Debug Mode

1. Settings → Enable **"Debug Mode"**
2. Settings → Enable **"Request Logging"**
3. Settings → Enable **"Logging to File"** (optional)

### Xem Debug Logs

1. Sidebar → **Logs**
2. Filter → **"DEBUG"**
3. Xem chi tiết:

```
[10:30:15] DEBUG  Request received: POST /v1/chat/completions
[10:30:15] DEBUG  Model: claude-sonnet-4-5-20250929
[10:30:15] DEBUG  Model mapping: sonnet → opus
[10:30:15] DEBUG  Routing to provider: Claude OAuth
[10:30:16] DEBUG  Response: 200 OK (1.2s, 150 tokens)
```

### Log File Location

```
%APPDATA%\proxypal\logs\
├── proxypal.log
├── proxy.log
└── error.log
```

### Export Logs

1. Logs page → Click **"Export"**
2. Chọn location
3. Chia sẻ khi report bug

---

## 10. Reset và Recovery

### Reset App Settings

```powershell
# Xóa config (giữ auth files)
Remove-Item "$env:APPDATA\proxypal\config.json"
# Restart app
```

### Reset OAuth Tokens

```powershell
# Xóa tất cả OAuth tokens
Remove-Item "$env:APPDATA\proxypal\*-oauth.json"
# Restart app → Connect lại providers
```

### Full Reset

```powershell
# Xóa toàn bộ data
Remove-Item -Recurse "$env:APPDATA\proxypal"
# Restart app (fresh start)
```

### Reinstall

1. Uninstall ProxyPal
2. Xóa data folder:

```powershell
Remove-Item -Recurse "$env:APPDATA\proxypal"
```

3. Reinstall từ [releases](https://github.com/nicepkg/proxypal/releases)

---

## Quick Reference

### Lỗi Phổ Biến → Giải Pháp Nhanh

| Lỗi | Giải pháp nhanh |
|-----|-----------------|
| Port already in use | `taskkill /PID <PID> /F` hoặc đổi port |
| no auth available | Connect ít nhất 1 provider |
| Connection refused | Start proxy |
| 401 Unauthorized | Kiểm tra API key |
| Model not found | Kiểm tra tên model, provider connected |
| OAuth failed | Restart app, connect lại |
| Config error | Xóa `config.json`, restart |

### Lệnh Debug Hữu Ích

```bash
# Kiểm tra proxy
curl http://localhost:8317/v1/models

# Test chat
curl http://localhost:8317/v1/chat/completions \
  -H "Authorization: Bearer proxypal-local" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-sonnet-4-5-20250929","messages":[{"role":"user","content":"Hi"}]}'

# Kiểm tra port
netstat -ano | findstr :8317
```

---

## Tham Khảo

- [Cài Đặt](./CAI_DAT.md)
- [Khởi Động](./KHOI_DONG.md)
- [Server Mode](./SERVER_MODE.md)
- [GitHub Issues](https://github.com/nicepkg/proxypal/issues)
