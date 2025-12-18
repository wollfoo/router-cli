# ProxyPal Configuration Examples

Thư mục này chứa các tệp cấu hình mẫu để tham khảo.

## Danh sách tệp

| Tệp | Mô tả | Vị trí thực tế |
|-----|-------|----------------|
| `config.example.json` | Cấu hình chính của ProxyPal | `%APPDATA%\proxypal\config.json` |
| `proxy-config.example.yaml` | Config sinh ra cho CLIProxyAPI | `%APPDATA%\proxypal\proxy-config.yaml` |
| `amp-settings.example.json` | Cấu hình Amp Code | `~/.config/amp/settings.json` |

## Cách sử dụng

### 1. ProxyPal Config (`config.json`)

Tệp này được ProxyPal tự động tạo và quản lý. Bạn có thể chỉnh sửa qua giao diện ứng dụng hoặc trực tiếp.

**Các trường quan trọng:**
- `claudeApiKeys`: Danh sách API keys cho Claude (hỗ trợ Azure AI Foundry)
- `ampApiKey`: API key từ ampcode.com
- `ampModelMappings`: Ánh xạ model (ví dụ: sonnet → opus)
- `thinkingBudgetCustom`: Token budget cho thinking models

### 2. Proxy Config (`proxy-config.yaml`)

Tệp này được **tự động sinh ra** từ `config.json` mỗi khi proxy khởi động. **KHÔNG chỉnh sửa trực tiếp**.

### 3. Amp Settings (`settings.json`)

Để sử dụng ProxyPal với Amp Code:

```json
{
  "amp.url": "http://localhost:8317",
  "amp.apiKey": "proxypal-local"
}
```

## Azure AI Foundry Integration

Để sử dụng với Azure AI Foundry:

1. Lấy API key từ Azure portal
2. Thêm vào ProxyPal qua Settings → Claude API Keys
3. Cấu hình model mapping:
   - `alias`: Tên model yêu cầu (ví dụ: `claude-opus-4-5-20251101`)
   - `name`: Tên deployment thực tế trên Azure (ví dụ: `claude-opus-4-5`)

## Lưu ý bảo mật

**KHÔNG commit các tệp sau vào git:**
- `config.json` (chứa API keys thật)
- `proxy-config.yaml` (chứa API keys thật)
- `~/.config/amp/settings.json` (có thể chứa tokens)

Chỉ sử dụng các tệp `.example` làm mẫu tham khảo.
