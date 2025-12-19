# ProxyPal

Use your AI subscriptions (Claude, ChatGPT, Gemini, GitHub Copilot) with any coding tool. Native desktop app wrapping [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI).

![ProxyPal Dashboard](src/assets/dashboard.png)

## Features

- **Multiple AI Providers** - Connect Claude, ChatGPT, Gemini, Qwen, iFlow, Vertex AI, and custom OpenAI-compatible endpoints
- **GitHub Copilot Bridge** - Use Copilot models via OpenAI-compatible API
- **Antigravity Support** - Access thinking models through Antigravity proxy
- **Server Mode** - Share your proxy with remote machines as a central model router
- **Model Mapping** - Route model requests to different providers (e.g., Claude → Gemini)
- **Works Everywhere** - Cursor, Cline, Continue, Claude Code, OpenCode, and any OpenAI-compatible client
- **Usage Analytics** - Track requests, tokens, success rates, and estimated savings
- **Request Monitoring** - View all API requests with response times and status codes
- **Auto-Configure** - Detects installed CLI agents and sets them up automatically

## Quick Start

1. Download from [Releases](https://github.com/heyhuynhgiabuu/proxypal/releases)
2. Launch ProxyPal and start the proxy
3. Connect your AI accounts (OAuth or auth files)
4. Point your coding tool to `http://localhost:8317/v1`

## Server Mode - Share with Remote Machines

ProxyPal can act as a **central model router** for multiple machines in your network.

### Enable Server Mode

1. Go to **Settings** → **Server Mode**
2. Toggle **Enable Server Mode**
3. Copy the **Remote Endpoint** (e.g., `http://10.0.0.4:8317/v1`)
4. Copy the **Remote API Key**
5. **Restart proxy** to apply changes
6. **Open firewall** for port 8317 (Windows: `New-NetFirewallRule -DisplayName "ProxyPal" -Direction Inbound -Protocol TCP -LocalPort 8317 -Action Allow`)

### Expose via Tailscale Funnel (Recommended)

Use **Tailscale Funnel** to expose ProxyPal with a free, permanent domain (no domain purchase needed):

```powershell
# Install Tailscale
winget install tailscale.tailscale

# Login (free account)
tailscale up

# Expose ProxyPal
tailscale funnel 8317
```

You'll get a permanent URL like: `https://your-pc.tail12345.ts.net`

| Feature | Value |
|---------|-------|
| Domain | Free `*.ts.net` subdomain |
| HTTPS | Automatic |
| Bandwidth | Unlimited |
| Cost | **$0** |

### Configure Remote Machines

On each remote machine, set environment variables:

```bash
# For Claude Code / Aider (via Tailscale Funnel)
export ANTHROPIC_BASE_URL=https://<YOUR_TAILSCALE_DOMAIN>
export ANTHROPIC_AUTH_TOKEN=<REMOTE_API_KEY>

# Or via local network
export ANTHROPIC_BASE_URL=http://<SERVER_IP>:8317
export ANTHROPIC_AUTH_TOKEN=<REMOTE_API_KEY>

# Model routing - use any available model
export ANTHROPIC_DEFAULT_OPUS_MODEL=gemini-2.5-pro
export ANTHROPIC_DEFAULT_SONNET_MODEL=gemini-2.5-flash
export ANTHROPIC_DEFAULT_HAIKU_MODEL=gemini-2.5-flash-lite

# For OpenAI-compatible clients (Cursor, Continue, etc.)
export OPENAI_BASE_URL=http://<SERVER_IP>:8317/v1
export OPENAI_API_KEY=<REMOTE_API_KEY>
```

### Available Models

Models depend on your configured providers. Example setup:

| Provider | Models |
|----------|--------|
| **Claude OAuth/API** | `claude-opus-4-5-20251101`, `claude-sonnet-4-5-20250929`, `claude-haiku-4-5` |
| **Gemini OAuth** | `gemini-2.5-pro`, `gemini-2.5-flash`, `gemini-2.5-flash-lite` |
| **Antigravity** | `gemini-claude-opus-4-5-thinking`, `gemini-claude-sonnet-4-5`, `gemini-3-pro-preview` |
| **Codex OAuth** | `gpt-5`, `gpt-5.1`, `gpt-5.1-codex`, `gpt-5.2` |
| **GitHub Copilot** | `gpt-5`, `claude-sonnet-4.5`, `gemini-2.5-pro` (via copilot-api) |

View all available models in **Logs** page (DEBUG level) after proxy starts.

### Model Mapping

Route requests from one model to another in **Settings** → **Amp Model Mappings**:

```
claude-opus-4-5-20251101 → gemini-claude-opus-4-5-thinking
gpt-5.1 → claude-sonnet-4-5-20250929
```

### macOS Users

The app is not signed with an Apple Developer certificate yet. If macOS blocks the app, run:

```bash
xattr -cr /Applications/ProxyPal.app
```

Then open the app again.

## Supported Platforms

| Platform | Architecture          | Status |
| -------- | --------------------- | ------ |
| macOS    | Apple Silicon (ARM64) | ✅     |
| macOS    | Intel (x64)           | ✅     |
| Windows  | x64                   | ✅     |
| Linux    | x64 (.deb)            | ✅     |

## Development

```bash
pnpm install
pnpm tauri dev
```

## Tech Stack

- **Frontend**: SolidJS + TypeScript + Tailwind CSS
- **Backend**: Rust + Tauri v2
- **Proxy**: CLIProxyAPI (bundled)

## Contributing

We welcome contributions! Here's how to submit a good PR:

1. **One feature per PR** - Keep changes focused and atomic
2. **Clean commits** - Don't include unrelated changes (watch your lockfiles!)
3. **Rebase before submitting** - Ensure no merge conflicts with main
4. **Test your changes** - Run `pnpm tauri dev` and verify functionality
5. **Follow existing patterns** - Check how similar features are implemented

### Adding a New Agent

If adding support for a new coding agent:

- Add detection logic in `src-tauri/src/lib.rs`
- Add logo to `public/logos/` (use `currentColor` for dark mode support)
- Update the agents array in relevant components
- Test the auto-configuration flow

### Code Style

- **TypeScript**: Follow existing patterns, use `type` imports
- **Rust**: Use `cargo check` before committing
- **Commits**: Clear, descriptive messages

## Support

If you find ProxyPal useful, consider [buying me a coffee](https://buymeacoffee.com/heyhuynhgiabuu).

## License

MIT
