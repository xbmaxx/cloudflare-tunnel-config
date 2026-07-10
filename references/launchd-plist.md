---
title: "Launchd Plist Template"
description: "macOS launchd plist for auto-starting the tunnel wrapper script"
tags: [launchd, plist, autostart, macos]
---

# Launchd Plist Template

The launchd plist ensures the tunnel wrapper script starts automatically when
macOS boots and restarts if it crashes.

## Plist Template

Save as `~/Library/LaunchAgents/com.cloudflared.tunnel.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.cloudflared.tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>ABSOLUTE_PATH_TO_WRAPPER</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>ABSOLUTE_PATH_TO_LOG</string>
    <key>StandardErrorPath</key>
    <string>ABSOLUTE_PATH_TO_ERROR_LOG</string>
</dict>
</plist>
```

Replace `ABSOLUTE_PATH_TO_WRAPPER` with the full path to the wrapper script
(e.g., `/Users/username/.hermes/cloudflared-tunnel.sh`).

Replace `ABSOLUTE_PATH_TO_LOG` and `ABSOLUTE_PATH_TO_ERROR_LOG` with log file
paths (e.g., `/Users/username/.hermes/logs/cloudflared.log`).

## Important: Path Must Be Absolute

- Do NOT use `~` or `$HOME` in plist files — launchd does not expand them
- Do NOT use relative paths
- Use the full absolute path from `echo $HOME`
- On macOS, `whoami` and `$HOME` can point to different directories — always
  use `$HOME`'s value

## Load and Unload

```bash
# Load (start now + auto-start on boot)
launchctl load ~/Library/LaunchAgents/com.cloudflared.tunnel.plist

# Unload (stop + disable auto-start)
launchctl unload ~/Library/LaunchAgents/com.cloudflared.tunnel.plist
```

## Verify It's Running

```bash
launchctl list | grep cloudflared
# Should show PID, not "-" or "0"
```

## Check the URL File

After loading, wait 10-15 seconds, then:

```bash
cat ~/.hermes/tunnel-url.txt
```

If the file is empty or missing, check the log:

```bash
cat ~/.hermes/logs/cloudflared.log | tail -20
```

## Kill and Recover

If you need to stop and restart the tunnel:

```bash
# 1. Unload to prevent launchd from auto-restarting
launchctl unload ~/Library/LaunchAgents/com.cloudflared.tunnel.plist

# 2. Kill all cloudflared processes
pkill -f cloudflared 2>/dev/null

# 3. Confirm no cloudflared remains
ps aux | grep cloudflared | grep -v grep

# 4. Reload
launchctl load ~/Library/LaunchAgents/com.cloudflared.tunnel.plist
```

## Common launchd Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Plist load returns "Input/output error" | Path in plist uses `~` or `$HOME` | Replace with absolute path |
| Tunnel never starts, no logs | Wrapper script missing execute permission | `chmod +x` the script |
| Multiple cloudflared instances | Previous manual run + launchd instance | Kill all, then load plist |
| URL file not updating | Plist points to cloudflared directly, not wrapper | Change plist to use wrapper script |
