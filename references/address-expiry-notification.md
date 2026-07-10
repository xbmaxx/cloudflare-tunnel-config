---
title: "Address Expiry Notification"
description: "Notify user when the tunnel URL changes via Feishu or WeChat"
tags: [notification, feishu, wechat, url, expiry]
---

# Address Expiry Notification

Quick Tunnel URLs change every time cloudflared restarts (system reboot, crash,
or manual restart). The address becomes stale until the wrapper script captures
the new one.

This reference covers how to:

1. Have the wrapper script automatically send the new URL
2. Ask your Hermes Agent for the current URL on demand
3. Choose between Feishu and WeChat notification channels

## Automatic Notification in Wrapper Script

Extend the wrapper script to include `hermes send` calls. The pattern:

- Try WeChat first (lower latency, direct to user)
- Fallback to Feishu if WeChat fails (no rate limits on Feishu)
- Add a throttle to avoid spamming on rapid restarts

### Notification-Enabled Wrapper Snippet

Add this block after the URL capture logic in the wrapper script:

```bash
# After URL is captured to URL_FILE
NOTIFY_THROTTLE_SEC=300  # 5 minutes between notifications
NOW=$(date +%s)
LAST=$(cat "$HERMES_HOME/.last-notify-time" 2>/dev/null || echo 0)

if [ $((NOW - LAST)) -ge $NOTIFY_THROTTLE_SEC ]; then
    echo "$NOW" > "$HERMES_HOME/.last-notify-time"

    MSG="🔗 Hermes Studio 新地址
$(cat "$URL_FILE")"

    # Try WeChat first, then Feishu
    if hermes send --to "weixin:YOUR_WECHAT_ID" "$MSG" 2>/dev/null; then
        echo "[$(date)] WeChat notified" >> "$LOG"
    elif hermes send --to "feishu:YOUR_FEISHU_CHAT" "$MSG" 2>/dev/null; then
        echo "[$(date)] Feishu notified" >> "$LOG"
    else
        echo "[$(date)] Both notification channels failed" >> "$LOG"
    fi
fi
```

Replace `YOUR_WECHAT_ID` and `YOUR_FEISHU_CHAT` with the actual chat/channel
IDs configured in the user's Hermes Gateway.

## Asking Your Agent for the Current URL

If the user does not have automatic notifications set up (or both channels
failed), they can ask their Hermes Agent directly:

> "Tunnel 地址是什么？"

The agent reads `~/.hermes/tunnel-url.txt` and returns the current URL.

## Platform Considerations

| Platform | hermes send target | Rate Limit | Setup Required |
|----------|-------------------|------------|----------------|
| WeChat | `weixin:USER_ID@im.wechat` | Yes (testing can trigger) | Hermes Gateway WeChat channel |
| Feishu | `feishu:CHAT_ID` | None | Hermes Gateway Feishu channel |

- If the user only has one platform configured, use that one
- If both are configured, WeChat is preferred for speed, Feishu as fallback
- If neither is configured, the user can still ask the agent for the URL
  manually through any channel (CLI, Studio, etc.)

## Important: Not Everyone Has Feishu or WeChat

This skill documents the notification mechanism as a convenience. Whether to
configure it is entirely up to the user. The skill never assumes a particular
messaging platform is available. The bare minimum requirement is:

- `cat ~/.hermes/tunnel-url.txt` to read the current URL
- Share it manually with followers
