# Model Mapping

Hướng dẫn chi tiết cấu hình Model Mapping để chuyển hướng requests từ model này sang model khác.

---

## Mục Lục

1. [Giới Thiệu](#1-giới-thiệu)
2. [Cách Hoạt Động](#2-cách-hoạt-động)
3. [Cấu Hình Qua UI](#3-cấu-hình-qua-ui)
4. [Cấu Hình Qua Config File](#4-cấu-hình-qua-config-file)
5. [Predefined Slots](#5-predefined-slots)
6. [Custom Mappings](#6-custom-mappings)
7. [Use Cases](#7-use-cases)
8. [Force Mapping Mode](#8-force-mapping-mode)
9. [Amp Model Mappings](#9-amp-model-mappings)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Giới Thiệu

### Model Mapping Là Gì?

Model Mapping cho phép bạn **chuyển hướng** request từ một model sang model khác mà client không cần biết.

```
┌──────────────────────────────────────────────────────────┐
│  Client Request: "gpt-4o"                                │
└──────────────────────┬───────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────┐
│  ProxyPal Model Mapping                                  │
│  Rule: gpt-4o → claude-sonnet-4-5-20250929              │
└──────────────────────┬───────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────┐
│  Actual Request: "claude-sonnet-4-5-20250929"            │
│  Provider: Claude                                        │
└──────────────────────────────────────────────────────────┘
```

### Lợi Ích

| Lợi ích | Mô tả |
|---------|-------|
| **Tiết kiệm chi phí** | Route model đắt về model rẻ hơn |
| **Tối ưu hiệu suất** | Dùng model nhanh cho tasks đơn giản |
| **Không cần sửa client** | Client vẫn request model cũ |
| **A/B Testing** | Test model mới mà không sửa code |
| **Fallback** | Khi model chính unavailable |

---

## 2. Cách Hoạt Động

### Luồng Xử Lý

```
1. Client gửi request với model "gpt-4o"
          ↓
2. ProxyPal nhận request
          ↓
3. Kiểm tra Model Mappings
   - Có rule "gpt-4o" → "claude-sonnet-4-5-20250929"? ✓
          ↓
4. Thay đổi model trong request
   - model: "gpt-4o" → "claude-sonnet-4-5-20250929"
          ↓
5. Route đến provider phù hợp (Claude)
          ↓
6. Trả response về client
```

### Thứ Tự Ưu Tiên

Khi có nhiều nguồn mapping:

```
1. Amp Model Mappings (nếu request từ Amp)
          ↓
2. Custom Model Mappings (Settings)
          ↓
3. Predefined Slots (Settings)
          ↓
4. Claude API Keys → models[] aliasing
          ↓
5. Không mapping → giữ nguyên model
```

---

## 3. Cấu Hình Qua UI

### Mở Settings

1. Sidebar → **Settings**
2. Hoặc shortcut `Ctrl+,`
3. Scroll đến section **"Model Mappings"**

### Giao Diện

```
┌─────────────────────────────────────────────────────────┐
│  Model Mappings                                         │
│                                                         │
│  ☑ Prioritize Model Mappings                            │
│     (Ưu tiên mapping thay vì local API keys)            │
│                                                         │
│  ═══ Predefined Slots ═══                               │
│                                                         │
│  [gpt-4      ] → [claude-sonnet-4-5-20250929 ▼] ☑       │
│  [gpt-4o     ] → [gemini-2.5-flash           ▼] ☑       │
│  [gpt-4-turbo] → [                           ▼] ☐       │
│                                                         │
│  ═══ Custom Mappings ═══                                │
│                                                         │
│  [custom-model] → [qwen-max                  ▼] ☑ [🗑]  │
│  [my-model    ] → [claude-opus-4-5-20251101  ▼] ☑ [🗑]  │
│                                                         │
│  [+ Add Custom Mapping]                                 │
└─────────────────────────────────────────────────────────┘
```

### Thêm Mapping Mới

1. Click **"+ Add Custom Mapping"**
2. Điền:
   - **From**: Tên model client gọi
   - **To**: Tên model thực tế muốn dùng
3. Bật toggle ☑ để enable
4. Click Save

---

## 4. Cấu Hình Qua Config File

### Vị Trí File

| OS | Đường dẫn |
|----|-----------|
| **Windows** | `%APPDATA%\proxypal\config.json` |
| **macOS** | `~/Library/Application Support/proxypal/config.json` |
| **Linux** | `~/.config/proxypal/config.json` |

### Cấu Trúc

```json
{
  "modelMappings": [
    {
      "from": "gpt-4",
      "to": "claude-sonnet-4-5-20250929",
      "enabled": true
    },
    {
      "from": "gpt-4o",
      "to": "gemini-2.5-flash",
      "enabled": true
    },
    {
      "from": "custom-model",
      "to": "qwen-max",
      "enabled": true
    }
  ],
  "forceModelMappings": false
}
```

### Fields

| Field | Type | Mô tả |
|-------|------|-------|
| `from` | string | Tên model trong request |
| `to` | string | Tên model thực tế |
| `enabled` | boolean | Bật/tắt mapping này |

---

## 5. Predefined Slots

### Các Slots Có Sẵn

ProxyPal cung cấp các mapping slots phổ biến:

| From | Mặc định To | Mô tả |
|------|-------------|-------|
| `gpt-4` | (trống) | GPT-4 base |
| `gpt-4o` | (trống) | GPT-4 Omni |
| `gpt-4-turbo` | (trống) | GPT-4 Turbo |
| `gpt-3.5-turbo` | (trống) | GPT-3.5 |
| `claude-3-opus` | (trống) | Claude 3 Opus |
| `claude-3-sonnet` | (trống) | Claude 3 Sonnet |

### Sử Dụng

1. Chọn dropdown "To" cho slot muốn mapping
2. Chọn model đích từ danh sách
3. Bật toggle ☑

### Ví Dụ

```json
{
  "predefinedMappings": {
    "gpt-4": {
      "to": "claude-sonnet-4-5-20250929",
      "enabled": true
    },
    "gpt-4o": {
      "to": "gemini-2.5-flash",
      "enabled": true
    }
  }
}
```

---

## 6. Custom Mappings

### Thêm Custom Mapping

Cho phép mapping bất kỳ tên model nào:

```json
{
  "modelMappings": [
    {
      "from": "my-custom-model",
      "to": "claude-opus-4-5-20251101",
      "enabled": true
    },
    {
      "from": "cheap-model",
      "to": "gemini-2.5-flash-lite",
      "enabled": true
    },
    {
      "from": "thinking-model",
      "to": "claude-opus-4-5-20251101",
      "enabled": true
    }
  ]
}
```

### Wildcard Mapping (Tương Lai)

```json
{
  "from": "gpt-*",
  "to": "claude-sonnet-4-5-20250929",
  "enabled": true
}
```

**Lưu ý:** Wildcard chưa được hỗ trợ trong version hiện tại.

---

## 7. Use Cases

### Use Case 1: Tiết Kiệm Chi Phí

Route model đắt về model rẻ hơn:

```json
{
  "modelMappings": [
    {
      "from": "claude-opus-4-5-20251101",
      "to": "gemini-2.5-pro",
      "enabled": true,
      "comment": "Opus $15/1M → Gemini $1.25/1M"
    },
    {
      "from": "gpt-4o",
      "to": "gemini-2.5-flash",
      "enabled": true,
      "comment": "GPT-4o $5/1M → Gemini $0.075/1M"
    }
  ]
}
```

### Use Case 2: Prefer Provider

Client dùng tên OpenAI nhưng thực tế dùng Claude:

```json
{
  "modelMappings": [
    { "from": "gpt-4", "to": "claude-sonnet-4-5-20250929", "enabled": true },
    { "from": "gpt-4o", "to": "claude-sonnet-4-5-20250929", "enabled": true },
    { "from": "gpt-4-turbo", "to": "claude-opus-4-5-20251101", "enabled": true }
  ]
}
```

### Use Case 3: Model Aliasing

Tạo tên ngắn gọn cho models dài:

```json
{
  "modelMappings": [
    { "from": "sonnet", "to": "claude-sonnet-4-5-20250929", "enabled": true },
    { "from": "opus", "to": "claude-opus-4-5-20251101", "enabled": true },
    { "from": "flash", "to": "gemini-2.5-flash", "enabled": true },
    { "from": "pro", "to": "gemini-2.5-pro", "enabled": true }
  ]
}
```

Sau đó client chỉ cần:

```bash
curl http://localhost:8317/v1/chat/completions \
  -d '{"model": "sonnet", "messages": [...]}'
```

### Use Case 4: A/B Testing

Test model mới mà không sửa client code:

```json
{
  "modelMappings": [
    {
      "from": "production-model",
      "to": "claude-sonnet-4-5-20250929",
      "enabled": true,
      "comment": "Đang test Sonnet thay vì Opus"
    }
  ]
}
```

### Use Case 5: Fallback

Khi model chính không khả dụng:

```json
{
  "modelMappings": [
    {
      "from": "claude-sonnet-4-5-20250929",
      "to": "gemini-2.5-pro",
      "enabled": true,
      "comment": "Fallback khi Claude down"
    }
  ]
}
```

---

## 8. Force Mapping Mode

### Bật Force Mapping

```json
{
  "forceModelMappings": true
}
```

### Hành Vi

| `forceModelMappings` | Request model có mapping | Request model không mapping |
|----------------------|--------------------------|------------------------------|
| `false` (default) | Route theo mapping | Giữ nguyên model |
| `true` | Route theo mapping | **Reject request** |

### Khi Nào Dùng

- Muốn kiểm soát chặt models được phép
- Chỉ cho phép models đã cấu hình
- Environment production với security requirements

### Ví Dụ

```json
{
  "forceModelMappings": true,
  "modelMappings": [
    { "from": "approved-model-1", "to": "claude-sonnet-4-5-20250929", "enabled": true },
    { "from": "approved-model-2", "to": "gemini-2.5-pro", "enabled": true }
  ]
}
```

Request với model không trong danh sách → Error:

```json
{
  "error": {
    "message": "Model 'unknown-model' is not allowed. Enable a mapping or disable forceModelMappings.",
    "code": "model_not_allowed"
  }
}
```

---

## 9. Amp Model Mappings

### Mục Đích

Riêng cho Amp Code integration, cho phép mapping độc lập:

```json
{
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
    }
  ],
  "ampRoutingMode": "mappings"
}
```

### Amp Routing Modes

| Mode | Hành vi |
|------|---------|
| `mappings` | Dùng `ampModelMappings` |
| `passthrough` | Không mapping, giữ nguyên |
| `default` | Fallback về model mặc định |

### Chi Tiết

Xem: [AMP_CODE.md](./AMP_CODE.md)

---

## 10. Troubleshooting

### Mapping Không Hoạt Động

#### Kiểm tra 1: Mapping đã enabled?

```json
{
  "from": "gpt-4",
  "to": "claude-sonnet-4-5-20250929",
  "enabled": true  // ← Phải là true
}
```

#### Kiểm tra 2: Tên model chính xác?

Model names phải khớp **chính xác** (case-sensitive):

```
❌ "GPT-4" (sai)
✓ "gpt-4" (đúng)

❌ "claude-sonnet" (sai)
✓ "claude-sonnet-4-5-20250929" (đúng)
```

#### Kiểm tra 3: Restart proxy?

Sau khi thay đổi mapping, cần restart proxy:

1. Dashboard → Stop Proxy
2. Dashboard → Start Proxy

### Model Đích Không Tồn Tại

Nếu model trong `to` không có sẵn:

```
Error: Model 'nonexistent-model' not found in any provider
```

**Giải pháp:** Kiểm tra model có trong danh sách `/v1/models`:

```bash
curl http://localhost:8317/v1/models | jq '.data[].id'
```

### Conflict Giữa Mappings

Nếu có nhiều mapping cho cùng 1 model:

```json
{
  "modelMappings": [
    { "from": "gpt-4", "to": "claude-sonnet", "enabled": true },
    { "from": "gpt-4", "to": "gemini-pro", "enabled": true }  // ← Conflict!
  ]
}
```

**Hành vi:** Mapping đầu tiên được áp dụng.

### Debug Mapping

Bật Debug Mode để xem mapping logs:

1. Settings → Enable **"Debug Mode"**
2. Logs page → Filter **"DEBUG"**
3. Xem:

```
[DEBUG] Model mapping: "gpt-4" → "claude-sonnet-4-5-20250929"
[DEBUG] Routing to provider: Claude
```

---

## Tham Khảo

- [Custom Providers](./CUSTOM_PROVIDERS.md) - Thêm providers mới
- [Amp Code](./AMP_CODE.md) - Amp integration
- [Ví Dụ Endpoint](./VI_DU_ENDPOINT.md) - Cấu hình mẫu
- [Xử Lý Lỗi](./TROUBLESHOOTING.md)
