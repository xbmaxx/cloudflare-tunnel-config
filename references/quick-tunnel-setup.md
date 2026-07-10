---
title: "Quick Tunnel Setup"
description: "Step-by-step guide to install cloudflared and start a Quick Tunnel"
tags: [cloudflare, tunnel, setup, quick]
---

# Quick Tunnel Setup

## Prerequisites

- Hermes Studio running locally (default port 8748)
- Homebrew installed (`brew` command available)

## Step 1: Install cloudflared

```bash
brew install cloudflared
```

Verify installation:

```bash
cloudflared --version
```

## Step 2: Verify Hermes Studio Port

Before starting the tunnel, confirm Hermes Studio is listening:

```bash
lsof -iTCP -sTCP:LISTEN -P 2>/dev/null | grep 8748
```

Expected output contains `Hermes` or `node` on `*:8748`.

If no output, start Hermes Studio first.

## Step 3: Start Quick Tunnel

```bash
cloudflared tunnel --url http://localhost:8748
```

Output example:

```
2025/01/15 10:30:00 Inf Cloudflare Tunnel
2025/01/15 10:30:02 Inf Requesting new quick Tunnel...
2025/01/15 10:30:05 Inf + https://random-words.trycloudflare.com
```

The `https://*.trycloudflare.com` URL is your tunnel address. Share this with
your followers — they can open it in any browser to access your Hermes Studio.

## Step 4: Verify from Another Device

Open the tunnel URL on a phone or another computer. You should see the Hermes
Studio login page or dashboard.

```bash
curl -sI https://random-words.trycloudflare.com | head -5
# Expected: HTTP/2 200
```

## Limitations

- The URL changes every time cloudflared restarts
- The URL is public — anyone who knows it can access your Studio
- Recommend enabling Hermes Studio password protection if sharing publicly
- For auto-restart and URL persistence, see the wrapper script and launchd
  setup in the other references
