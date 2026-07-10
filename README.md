# Cloudflare Tunnel Config — Hermes Skill

A Hermes Agent skill that teaches your AI how to expose Hermes Studio Web UI to the internet via **Cloudflare Quick Tunnel** — free, no domain needed, ~50-100ms latency from China mobile networks.

一个 Hermes Agent 技能，让你的 AI 学会通过 **Cloudflare Quick Tunnel** 把 Hermes Studio Web UI 暴露到公网——免费、无需域名，国内移动网络延迟约 50-100ms。

---

## What It Does / 它能做什么

When loaded, this skill tells your Hermes Agent everything it needs to:

加载后，你的 Agent 就能掌握以下全套操作：

1. **Install** `cloudflared` via Homebrew / 通过 Homebrew 安装 cloudflared
2. **Start** a Quick Tunnel pointing to Hermes Studio (port 8748) / 启动 Quick Tunnel，指向 Hermes Studio（默认端口 8748）
3. **Capture** the temporary `*.trycloudflare.com` URL to a file / 把临时 `*.trycloudflare.com` 地址写入文件
4. **Auto-restart** on macOS boot via launchd / 通过 launchd 实现开机自启
5. **Notify** you when the address changes (WeChat → Feishu fallback) / 地址变更时自动通知（微信 → 飞书降级）
6. **Verify** the tunnel is working end-to-end / 验证隧道端到端连通性

---

## Who Is This For / 适用人群

- **English:** Hermes users who want remote access to their Studio from phone/tablet; users whose followers ask how to expose their Hermes instance; anyone who wants a free, zero-VPS solution for Hermes remote access
- **中文：** 想在手机/平板上远程访问自己 Hermes Studio 的用户；粉丝问你「怎么把你的 Hermes 暴露到公网」的创作者；任何想要免费、零 VPS 的 Hermes 远程访问方案的人

---

## Structure / 目录结构

```
cloudflare-tunnel-config/
├── SKILL.md                              # Main skill definition / 主技能定义
├── README.md                             # This file / 本文件
└── references/
    ├── quick-tunnel-setup.md             # Install & first run / 安装与首次运行
    ├── wrapper-script.md                 # URL capture bash wrapper / URL 捕获脚本
    ├── launchd-plist.md                  # macOS auto-restart plist / macOS 开机自启
    ├── address-expiry-notification.md    # Notify on URL change / 地址变更通知
    └── tunnel-health-verification.md     # Verification commands / 健康检查
```

---

## Installation / 安装

```bash
# Clone into your Hermes skills directory
# 克隆到 Hermes 技能目录
git clone https://github.com/xbmaxx/cloudflare-tunnel-config \
  ~/.hermes/skills/hermes/cloudflare-tunnel-config
```

Your Hermes Agent will automatically discover the skill on next session start.

下次启动 Hermes 会话时，Agent 会自动发现这个技能。

---

## Requirements / 前置条件

| 项目 | 说明 |
|------|------|
| macOS | For launchd auto-restart / 用于 launchd 开机自启 |
| Homebrew | Required for cloudflared install / 安装 cloudflared 所需 |
| Hermes Studio | Running locally (default port 8748) / 本地运行（默认端口 8748） |
| `hermes send` | Configured for WeChat/Feishu (optional, for notifications) / 配置了微信/飞书（可选，用于通知） |

---

## License / 许可

MIT
