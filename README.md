# Cloudflare Tunnel Config — Hermes Skill

A Hermes Agent skill that teaches your AI how to expose Hermes Studio Web UI to the internet via **Cloudflare Quick Tunnel** — free, no domain needed, ~50-100ms latency from China mobile networks.

## What It Does

When loaded, this skill tells your Hermes Agent everything it needs to:

1. **Install** `cloudflared` via Homebrew
2. **Start** a Quick Tunnel pointing to Hermes Studio (port 8748)
3. **Capture** the temporary `*.trycloudflare.com` URL to a file
4. **Auto-restart** on macOS boot via launchd
5. **Notify** you when the address changes (WeChat → Feishu fallback)
6. **Verify** the tunnel is working end-to-end

## Who Is This For

- Hermes users who want remote access to their Studio from phone/tablet
- Users whose followers ask how to expose their Hermes instance
- Anyone who wants a free, zero-VPS solution for Hermes remote access

## Structure

```
cloudflare-tunnel-config/
├── SKILL.md                              # Main skill definition
├── README.md                             # This file
└── references/
    ├── quick-tunnel-setup.md             # Install & first run
    ├── wrapper-script.md                 # URL capture bash wrapper
    ├── launchd-plist.md                  # macOS auto-restart plist
    ├── address-expiry-notification.md    # Notify on URL change
    └── tunnel-health-verification.md     # Verification commands
```

## Installation

```bash
# Clone into your Hermes skills directory
git clone https://github.com/xbmaxx/cloudflare-tunnel-config \
  ~/.hermes/skills/hermes/cloudflare-tunnel-config
```

Your Hermes Agent will automatically discover the skill on next session start.

## Requirements

- macOS (for launchd auto-restart)
- Homebrew
- Hermes Studio running locally (default port 8748)
- `hermes send` configured for WeChat and/or Feishu (optional — for notifications)

## License

MIT
