---
name: cloudflare-tunnel-config
description: >
  Use when the user wants to expose their local Hermes Studio Web UI to the
  internet via Cloudflare Tunnel (Quick Tunnel). Covers cloudflared install,
  Quick Tunnel startup, wrapper script with automatic URL capture, launchd
  auto-restart, address expiry notification via Feishu or WeChat, and tunnel
  health verification. Does NOT cover Named Tunnel (requires a domain).
  Use for: setting up remote access from phone/tablet, sharing a Hermes Studio
  link with followers, configuring tunnel auto-recovery after reboot.
license: MIT
metadata:
  author: Hermes Community
  version: "1.0"
---
# Cloudflare Tunnel Config

## Overview

Cloudflare Tunnel (Quick Tunnel) exposes your local Hermes Studio Web UI to the
internet through Cloudflare's edge network. Unlike Tailscale DERP relay
(~250-600ms latency from China mobile networks), Cloudflare Tunnel provides
~50-100ms latency and works reliably on 4G/5G. The tunnel address changes every
time cloudflared restarts, so this skill covers automatic URL capture and
notification workflows.

The target audience is Hermes users who want their followers or themselves to
access their Hermes Studio remotely without buying a VPS or domain.

## Quick Reference

| Concept | Summary | Key Rule |
|---------|---------|----------|
| Quick Tunnel | Free, instant, no domain needed | Address changes on every restart |
| cloudflared | CLI tool from Cloudflare | Install via `brew install cloudflared` |
| Wrapper script | Bash script that captures the URL | Must replace direct cloudflared in launchd |
| launchd | macOS auto-restart service | Plist must point to wrapper, not cloudflared |
| URL notification | Send new address via Feishu/WeChat | Use `hermes send` with fallback chain |
| Hermes Studio port | Default is 8748 | Verify with `lsof -iTCP -sTCP:LISTEN -P` |
| Address expiry | Tunnel URL changes after restart | Every restart generates a new trycloudflare.com URL |

## Common Mistakes

| # | Mistake | Correct Approach |
|---|---------|------------------|
| 1 | Launchd plist runs cloudflared directly | Always point to the wrapper script — direct execution loses the URL |
| 2 | Hardcoding $HOME paths in scripts | Use the user's actual $HOME or HERMES_HOME env var |
| 3 | Skipping port verification before tunnel start | Always confirm Hermes Studio is listening on 8748 first |
| 4 | Notifying only one messaging platform | Try WeChat first, fallback to Feishu (which has no rate limits) |
| 5 | Assuming the tunnel URL persists | Every restart generates a new URL — treat it as ephemeral |
| 6 | Leaving old tunnel-url.txt as stale reference | Always read the fresh URL from the wrapper's output file |
| 7 | Using foreground `&` to background cloudflared | Use `terminal(background=true)` or launchd, never shell `&` |
| 8 | Writing wrapper paths with $USER instead of $HOME | On macOS $HOME and whoami can diverge — always use $HOME |
| 9 | Forgetting to chmod +x the wrapper script | Without execute permission launchd silently fails |
| 10 | Not adding a notification throttle | Rate-limit notifications to avoid WeChat rate limiting |
| 11 | Verifying tunnel with HEAD request | Some tunnel endpoints respond differently to GET vs HEAD |
| 12 | Tunnel 1033 error without checking for stale processes | Kill all cloudflared instances before restarting |
| 13 | Hardcoding the port number in wrapper | Accept port as a parameter so it works with different services |
| 14 | Putting sensitive paths in shared templates | Use placeholder like `$HOME/.hermes/` instead of real usernames |
| 15 | Assuming everyone has both Feishu and WeChat configured | Let the user decide which platform to use; note both options |

## Delegation

This skill covers the Quick Tunnel setup end to end. For advanced topics:

- **Deep debugging, cache-header injection, Hermes Studio process topology,
  mobile blank-page diagnosis, LAN pairing** → `hermes-remote-access` (the
  internal advanced umbrella for all remote access topics)
- **Server-side deployment on a VPS** → `hermes-server-deployment`
- **Named Tunnel (persistent domain-based tunnel)** — requires purchasing a
  domain (cheapest ~3-5 CNY/year) and is out of scope for this skill; refer
  to Cloudflare Tunnel documentation or load `hermes-remote-access`

## References

- `references/quick-tunnel-setup.md` — Full step-by-step Quick Tunnel setup
- `references/wrapper-script.md` — Wrapper script with URL capture + notification
- `references/launchd-plist.md` — macOS launchd auto-restart plist template
- `references/address-expiry-notification.md` — Address change notification via Feishu/WeChat
- `references/tunnel-health-verification.md` — Verification commands after setup
