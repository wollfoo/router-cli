# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Full desktop app with hot reload (recommended)
pnpm tauri dev

# Frontend only (Vite dev server on port 1420)
pnpm dev

# Production build
pnpm build              # Frontend bundle to /dist
pnpm tauri build        # Native installers (NSIS/DMG/AppImage)

# Type checking
pnpm tsc --noEmit       # TypeScript
cd src-tauri && cargo check  # Rust
```

No formal test suite - manual testing via TEST_PLAN.md.

## Architecture

**ProxyPal** is a Tauri desktop app that unifies multiple AI subscriptions (Claude, Gemini, Copilot, etc.) into a single OpenAI-compatible proxy endpoint.

```
┌─────────────────────────────────────────────────────────────┐
│  Frontend (SolidJS)                                         │
│  src/pages/*.tsx → src/lib/tauri.ts (IPC wrappers)          │
└─────────────────────┬───────────────────────────────────────┘
                      │ Tauri IPC commands
┌─────────────────────▼───────────────────────────────────────┐
│  Backend (Rust)                                             │
│  src-tauri/src/lib.rs → proxy control, OAuth, config        │
└─────────────────────┬───────────────────────────────────────┘
                      │ spawns/manages process
┌─────────────────────▼───────────────────────────────────────┐
│  CLIProxyAPI (bundled binary)                               │
│  src-tauri/CLIProxyAPI/ → routes to AI providers            │
└─────────────────────────────────────────────────────────────┘
```

### Key Files

| File | Purpose |
|------|---------|
| `src/lib/tauri.ts` | All frontend-to-backend IPC bindings (~29KB) |
| `src/stores/app.ts` | Global SolidJS state (proxy status, auth, config, current page) |
| `src-tauri/src/lib.rs` | Main backend logic (proxy control, OAuth flows, config management) |
| `src-tauri/tauri.conf.json` | Tauri config (window size, bundle settings, updater) |

### Data Flow

1. **User action** → SolidJS component calls function from `lib/tauri.ts`
2. **IPC call** → Tauri invokes `#[tauri::command]` function in `lib.rs`
3. **Backend** → Manages CLIProxyAPI process, reads/writes config, handles OAuth
4. **Events** → Backend emits events (`proxy-status`, `auth-status-changed`) → Frontend listens via `appWindow.listen()`

### Configuration

- **Location**: `%APPDATA%\proxypal\config.json` (Windows) or `~/.config/proxypal/config.json` (Unix)
- **Synced to proxy**: Backend generates `proxy-config.yaml` from AppConfig

## Code Patterns

### TypeScript (SolidJS)

```typescript
// Import order: external → internal aliases → relative
import { createSignal, onMount } from "solid-js";
import type { Provider } from "../lib/tauri";
import { Button } from "./ui/Button";

// Props interface directly above component
interface SettingsProps {
  onSave: () => void;
}

// Use class (not className) for Tailwind
<div class="flex items-center gap-2">
```

### Rust (Tauri)

```rust
// Derive for IPC serialization
#[derive(Clone, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
pub struct ProxyStatus {
    pub is_running: bool,
    pub port: u16,
}

// Commands return Result<T, String>
#[tauri::command]
async fn start_proxy(state: State<'_, AppState>) -> Result<(), String> {
    // ...
}
```

## Adding Features

### New AI Provider

1. Add detection logic in `src-tauri/src/lib.rs`
2. Add logo to `public/logos/` (use `currentColor` for dark mode)
3. Update provider arrays in `src/pages/Settings.tsx` and `src/lib/tauri.ts`
4. Test OAuth/auth file flow via `pnpm tauri dev`

### New Coding Agent (auto-configure target)

1. Add detection in `lib.rs` (check config file paths per OS)
2. Add to agents list in Dashboard/Settings
3. Add configuration writer for that agent's config format
