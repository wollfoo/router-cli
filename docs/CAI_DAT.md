# Hướng Dẫn Cài Đặt ProxyPal

Hướng dẫn chi tiết cài đặt ProxyPal trên Windows, macOS và Linux.

---

## Mục Lục

1. [Yêu Cầu Hệ Thống](#1-yêu-cầu-hệ-thống)
2. [Cài Đặt Bản Build Sẵn](#2-cài-đặt-bản-build-sẵn)
3. [Build Từ Source](#3-build-từ-source)
4. [Cấu Hình Sau Cài Đặt](#4-cấu-hình-sau-cài-đặt)
5. [Xác Minh Cài Đặt](#5-xác-minh-cài-đặt)
6. [Gỡ Cài Đặt](#6-gỡ-cài-đặt)

---

## 1. Yêu Cầu Hệ Thống

### Hệ Điều Hành Hỗ Trợ

| OS | Version | Kiến trúc |
|----|---------|-----------|
| **Windows** | 10, 11 | x64, arm64 |
| **macOS** | 11+ (Big Sur) | Intel, Apple Silicon |
| **Linux** | Ubuntu 20.04+, Fedora 35+ | x64 |

### Yêu Cầu Phần Cứng

| Thành phần | Tối thiểu | Khuyến nghị |
|------------|-----------|-------------|
| **RAM** | 2 GB | 4 GB |
| **Disk** | 200 MB | 500 MB |
| **Network** | Internet để OAuth | Băng thông tốt cho AI streaming |

### Dependencies

- **WebView2 Runtime** (Windows) - Tự động cài nếu chưa có
- **OpenSSL** (Linux) - `libssl-dev` hoặc `openssl-devel`

---

## 2. Cài Đặt Bản Build Sẵn

### Windows

#### Cách 1: Installer (.msi)

1. Truy cập: https://github.com/nicepkg/proxypal/releases
2. Tải file `proxypal_x.x.x_x64-setup.msi`
3. Chạy installer
4. Chọn thư mục cài đặt (mặc định: `C:\Program Files\proxypal`)
5. Hoàn tất

```powershell
# Hoặc dùng winget
winget install nicepkg.proxypal
```

#### Cách 2: Portable (.exe)

1. Tải file `proxypal_x.x.x_x64.exe`
2. Di chuyển đến thư mục mong muốn
3. Chạy trực tiếp (không cần cài đặt)

### macOS

#### Cách 1: DMG

1. Tải file `proxypal_x.x.x_x64.dmg` (Intel) hoặc `_aarch64.dmg` (Apple Silicon)
2. Mở file DMG
3. Kéo ProxyPal vào thư mục Applications
4. Lần đầu mở: Right-click → Open (bypass Gatekeeper)

#### Cách 2: Homebrew

```bash
# Sẽ hỗ trợ trong tương lai
brew install --cask proxypal
```

### Linux

#### AppImage

```bash
# Tải và chạy
wget https://github.com/nicepkg/proxypal/releases/download/vX.X.X/proxypal_x.x.x_amd64.AppImage
chmod +x proxypal_*.AppImage
./proxypal_*.AppImage
```

#### Debian/Ubuntu (.deb)

```bash
wget https://github.com/nicepkg/proxypal/releases/download/vX.X.X/proxypal_x.x.x_amd64.deb
sudo dpkg -i proxypal_*.deb
sudo apt-get install -f  # Fix dependencies nếu cần
```

#### Fedora/RHEL (.rpm)

```bash
wget https://github.com/nicepkg/proxypal/releases/download/vX.X.X/proxypal_x.x.x_amd64.rpm
sudo rpm -i proxypal_*.rpm
```

---

## 3. Build Từ Source

### Prerequisites

| Tool | Version | Kiểm tra |
|------|---------|----------|
| **Node.js** | 18+ | `node -v` |
| **pnpm** | 8+ | `pnpm -v` |
| **Rust** | 1.70+ | `rustc --version` |
| **Git** | 2.0+ | `git --version` |

### Cài Đặt Prerequisites

#### Windows

```powershell
# Node.js
winget install OpenJS.NodeJS.LTS

# pnpm
npm install -g pnpm

# Rust
winget install Rustlang.Rustup
rustup default stable

# Visual Studio Build Tools (cần cho native dependencies)
winget install Microsoft.VisualStudio.2022.BuildTools
```

#### macOS

```bash
# Node.js
brew install node

# pnpm
brew install pnpm

# Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# Xcode Command Line Tools
xcode-select --install
```

#### Linux (Ubuntu/Debian)

```bash
# Dependencies
sudo apt update
sudo apt install -y build-essential libssl-dev pkg-config \
    libgtk-3-dev libayatana-appindicator3-dev librsvg2-dev \
    libwebkit2gtk-4.1-dev

# Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# pnpm
npm install -g pnpm

# Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

### Build Steps

```bash
# Clone repository
git clone https://github.com/nicepkg/proxypal.git
cd proxypal

# Cài dependencies
pnpm install

# Development mode (hot reload)
pnpm tauri dev

# Production build
pnpm tauri build
```

### Build Output

```
src-tauri/target/release/bundle/
├── msi/           # Windows installer
├── nsis/          # Windows NSIS installer
├── dmg/           # macOS disk image
├── macos/         # macOS app bundle
├── appimage/      # Linux AppImage
├── deb/           # Debian package
└── rpm/           # RPM package
```

---

## 4. Cấu Hình Sau Cài Đặt

### Vị Trí File Cấu Hình

| OS | Đường dẫn |
|----|-----------|
| **Windows** | `%APPDATA%\proxypal\config.json` |
| **macOS** | `~/Library/Application Support/proxypal/config.json` |
| **Linux** | `~/.config/proxypal/config.json` |

### Khởi Tạo Cấu Hình

Lần đầu chạy app, file `config.json` được tạo tự động với giá trị mặc định:

```json
{
  "port": 8317,
  "autoStart": true,
  "launchAtLogin": false,
  "debug": false,
  "configVersion": 1
}
```

### Cấu Hình Firewall

#### Windows

```powershell
# Cho phép ProxyPal qua Windows Firewall
New-NetFirewallRule -DisplayName "ProxyPal" -Direction Inbound -Protocol TCP -LocalPort 8317 -Action Allow
```

#### macOS

ProxyPal sẽ yêu cầu quyền mạng khi khởi động lần đầu. Click "Allow".

#### Linux

```bash
# UFW
sudo ufw allow 8317/tcp

# firewalld
sudo firewall-cmd --permanent --add-port=8317/tcp
sudo firewall-cmd --reload
```

### Cấu Hình Auto-Start

#### Windows

1. Mở ProxyPal Settings
2. Bật toggle **"Launch at Login"**

Hoặc thủ công:

```powershell
# Tạo shortcut trong Startup folder
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\ProxyPal.lnk")
$Shortcut.TargetPath = "C:\Program Files\proxypal\ProxyPal.exe"
$Shortcut.Save()
```

#### macOS

1. System Preferences → Users & Groups → Login Items
2. Add ProxyPal

#### Linux (systemd)

```bash
# ~/.config/systemd/user/proxypal.service
[Unit]
Description=ProxyPal AI Proxy
After=graphical-session.target

[Service]
ExecStart=/usr/bin/proxypal
Restart=on-failure

[Install]
WantedBy=default.target
```

```bash
systemctl --user enable proxypal
systemctl --user start proxypal
```

---

## 5. Xác Minh Cài Đặt

### Kiểm Tra App

1. Mở ProxyPal
2. Dashboard hiển thị: "Proxy Stopped" hoặc "Proxy Running"
3. Không có error messages

### Kiểm Tra Proxy

```bash
# Start proxy trong app, sau đó test:
curl http://localhost:8317/v1/models

# Kết quả khi chưa connect provider:
# {"error": "no auth available"}

# Kết quả sau khi connect provider:
# {"data": [{"id": "claude-sonnet-4-5-20250929", ...}]}
```

### Kiểm Tra Version

```bash
# Trong app: Settings → About
# Hoặc:
ProxyPal.exe --version
```

---

## 6. Gỡ Cài Đặt

### Windows

#### Cách 1: Settings

1. Settings → Apps → Installed apps
2. Tìm "ProxyPal"
3. Click Uninstall

#### Cách 2: Command line

```powershell
# Nếu cài qua winget
winget uninstall proxypal

# Xóa data còn lại
Remove-Item -Recurse "$env:APPDATA\proxypal"
```

### macOS

```bash
# Xóa app
rm -rf /Applications/ProxyPal.app

# Xóa data
rm -rf ~/Library/Application\ Support/proxypal
rm -rf ~/Library/Caches/proxypal
```

### Linux

```bash
# Debian/Ubuntu
sudo apt remove proxypal
rm -rf ~/.config/proxypal

# Fedora/RHEL
sudo rpm -e proxypal
rm -rf ~/.config/proxypal

# AppImage
rm proxypal_*.AppImage
rm -rf ~/.config/proxypal
```

---

## Tham Khảo

- [Khởi Động và Kết Nối](./KHOI_DONG.md)
- [Cấu Hình Coding Tools](./CAU_HINH_TOOLS.md)
- [Xử Lý Lỗi](./TROUBLESHOOTING.md)
- [README](../README.md)
