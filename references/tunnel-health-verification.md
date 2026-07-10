---
title: "Tunnel Health Verification"
description: "Commands to verify the tunnel is working after setup"
tags: [verification, health, tunnel, curl]
---

# Tunnel Health Verification

After setting up the tunnel, verify it's working end-to-end.

## Quick Check

```bash
# Read the current URL
TUNNEL_URL=$(cat ~/.hermes/tunnel-url.txt)

# Check HTTP status
curl -s -o /dev/null -w "%{http_code}" --connect-timeout 10 "$TUNNEL_URL"
# Expected: 200
```

## Full Diagnostics

```bash
TUNNEL_URL=$(cat ~/.hermes/tunnel-url.txt)

echo "=== Tunnel URL ==="
echo "$TUNNEL_URL"

echo "=== HTTP Status ==="
curl -sI "$TUNNEL_URL" | head -5

echo "=== HTML Loads ==="
curl -s "$TUNNEL_URL" | head -3
# Should show <!doctype html>

echo "=== JS Bundle ==="
curl -sI "$TUNNEL_URL/assets/js/index-"*.js | grep -E 'HTTP|content-length|cache'

echo "=== API ==="
curl -sI "$TUNNEL_URL/api/" | head -3
```

## Check cloudflared Process

```bash
ps aux | grep cloudflared | grep -v grep
```

Expected: exactly one cloudflared process running with `--url http://localhost:8748`.

## Check launchd (if configured)

```bash
launchctl list | grep cloudflared
```

Expected output shows a PID (not `-` or `0`).

## Common Failure Modes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| curl returns 502 | Hermes Studio not running | Start Studio first, then tunnel |
| curl returns 530 / 1033 | Stale cloudflared process | Kill all cloudflared, restart |
| curl timeout (no response) | Network issue or tunnel not started | Check cloudflared process exists |
| tunnel-url.txt is empty | Wrapper script not running | Check launchd or start wrapper manually |
| Mobile browser shows blank page | PWA cache pointing to old URL | Open Safari, manually type new URL |
