---
type: handoff
created: 2025-12-18 16:25
from_session: proxypal-azure-integration
goal: ProxyPal hoạt động với Azure AI Foundry và Amp Code
status: ready
---

# Handoff: Azure AI Foundry Integration cho ProxyPal

## Origin Context

Session này tập trung vào việc fix bugs để ProxyPal có thể route requests qua Azure AI Foundry. Nhiều issues đã được phát hiện và fix, bao gồm Management API 404, model aliasing, và config generation.

## Extracted Context

### 🎯 Goal
Sử dụng ProxyPal làm proxy để route AI requests từ Amp Code/Claude Code đến Azure AI Foundry (thay vì Anthropic API trực tiếp).

### ✅ Completed
- [x] Fix 404 error khi add Claude API keys (Management API không tồn tại trong bundled CLIProxyAPI)
- [x] Chuyển API key functions sang đọc/ghi local config + restart proxy
- [x] Fix model dropdown không hiển thị models (filter `owned_by === "anthropic"` → thêm `"claude"`)
- [x] Fix models không được ghi vào `proxy-config.yaml`
- [x] Fix `set_force_model_mappings` 404 error
- [x] Test thành công qua curl với Azure AI Foundry
- [x] Cấu hình Amp Code settings
- [x] Tạo example configs trong `examples/` folder
- [x] Cập nhật tài liệu `docs/HUONG_DAN_SU_DUNG.md`

### 🔄 In Progress
- [ ] Test Amp Code với ProxyPal (user cần verify)
- [ ] Build và release version mới

### 📁 Modified Files

**Rust Backend:**
- `src-tauri/src/lib.rs` – Fix API key functions, thêm models vào proxy-config generation

**Frontend:**
- `src/pages/Settings.tsx` – Fix model dropdown filter

**Documentation:**
- `docs/HUONG_DAN_SU_DUNG.md` – Cập nhật hướng dẫn Azure AI Foundry, Amp Code integration

**New Files:**
- `examples/README.md` – Hướng dẫn sử dụng example configs
- `examples/config.example.json` – ProxyPal config mẫu
- `examples/proxy-config.example.yaml` – Proxy config mẫu
- `examples/amp-settings.example.json` – Amp Code settings mẫu

### 💡 Key Decisions

1. **Local config thay vì Management API**: CLIProxyAPI bundled không có Management API endpoints. Giải pháp: đọc/ghi trực tiếp vào `config.json` và restart proxy.

2. **Model Aliasing**: Azure AI Foundry deployment names khác Anthropic standard (e.g., `claude-opus-4-5` vs `claude-opus-4-5-20251101`). Cần map alias → name trong config.

3. **Headers tự động**: ProxyPal tự thêm `x-api-key` và `anthropic-version` headers khi gọi Azure.

4. **Proxy restart**: Mỗi khi save API keys hoặc settings → stop proxy → wait 500ms → start lại để load config mới.

### ⚠️ Notes

**Cấu hình Azure AI Foundry đang hoạt động:**
```yaml
claude-api-key:
  - api-key: "YOUR_KEY"
    base-url: "https://resource.services.ai.azure.com/anthropic"
    models:
      - alias: "claude-opus-4-5-20251101"
        name: "claude-opus-4-5"
```

**Config files location:**
- ProxyPal: `%APPDATA%\proxypal\config.json`
- Proxy: `%APPDATA%\proxypal\proxy-config.yaml` (auto-generated)
- Amp: `~/.config/amp/settings.json`

**Test command thành công:**
```bash
curl -X POST "http://localhost:8317/v1/messages" \
  -H "Content-Type: application/json" \
  -H "x-api-key: proxypal-local" \
  -H "anthropic-version: 2023-06-01" \
  -d '{"model":"claude-opus-4-5-20251101","max_tokens":100,"messages":[{"role":"user","content":"Hello"}]}'
```

## Next Steps

1. **Test Amp Code** – Verify Amp Code hoạt động qua ProxyPal
2. **Build release** – `pnpm tauri build` để tạo installer mới
3. **Commit changes** – Commit các file đã thay đổi
4. **Test các provider khác** – Gemini, Codex, OpenAI Compatible

## Relevant Files

@src-tauri/src/lib.rs
@src/pages/Settings.tsx
@docs/HUONG_DAN_SU_DUNG.md
@examples/config.example.json
@examples/proxy-config.example.yaml
@examples/amp-settings.example.json

## Resume Prompt

```
Tiếp tục task: **ProxyPal Azure AI Foundry Integration**

### Context
Session trước đã fix bugs cho ProxyPal để route requests qua Azure AI Foundry. Các vấn đề chính: Management API 404, model aliasing, config generation đã được giải quyết. Test curl thành công.

### Files cần xem
@src-tauri/src/lib.rs @docs/HUONG_DAN_SU_DUNG.md @examples/

### Next action
1. Verify Amp Code hoạt động
2. Build và release version mới
3. Commit changes

### Reference
Xem chi tiết: `.handoff/checkpoints/handoff-azure-ai-foundry-integration-251218-1625.md`
```
