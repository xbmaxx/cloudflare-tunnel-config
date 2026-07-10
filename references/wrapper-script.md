---
title: "Wrapper Script"
description: "Bash wrapper script that captures tunnel URL and persists it to a file"
tags: [wrapper, script, url-capture, bash]
---

# Wrapper Script

The wrapper script replaces direct cloudflared execution. It captures the
`trycloudflare.com` URL from cloudflared's stdout and writes it to a file so
other tools (launchd, notification scripts, agent queries) can read it.

## Script Template

Save this as `~/.hermes/cloudflared-tunnel.sh`:

```bash
#!/bin/bash
# Cloudflare Quick Tunnel wrapper — captures URL to a file
# Usage: ./cloudflared-tunnel.sh [port] [hermes_home]
#   port        — local port (default: 8748)
#   hermes_home — data directory (default: ~/.hermes)

PORT="${1:-8748}"
HERMES_HOME="${2:-$HOME/.hermes}"
LOG="$HERMES_HOME/cloudflared-tunnel.log"
URL_FILE="$HERMES_HOME/tunnel-url.txt"

mkdir -p "$HERMES_HOME/logs"
echo "[$(date)] Starting tunnel on port $PORT..." >> "$LOG"

/opt/homebrew/bin/cloudflared tunnel --url "http://localhost:$PORT" 2>&1 \
  | while IFS= read -r line; do
    echo "$line" >> "$LOG"
    if echo "$line" | grep -q "trycloudflare.com"; then
        URL=$(echo "$line" | grep -oE 'https://[a-z0-9-]+\.trycloudflare\.com')
        if [ -n "$URL" ]; then
            echo "[$(date)] TUNNEL URL: $URL" >> "$LOG"
            echo "$URL" > "$URL_FILE"
        fi
    fi
done
```

## Make It Executable

```bash
chmod +x ~/.hermes/cloudflared-tunnel.sh
```

## Test Run

```bash
bash ~/.hermes/cloudflared-tunnel.sh
```

After 5-10 seconds, in another terminal:

```bash
cat ~/.hermes/tunnel-url.txt
# Expected: https://some-words.trycloudflare.com
```

Press Ctrl+C to stop the test run.

## Key Details

- The script accepts port and hermes_home as parameters for reuse with other
  services (e.g., a different port or a different data directory)
- The URL is written atomically (overwrites the file) so readers always get
  the latest address
- Logs are written to `$HERMES_HOME/logs/cloudflared-tunnel.log`
- For production use, pair this with launchd (see launchd plist reference)
