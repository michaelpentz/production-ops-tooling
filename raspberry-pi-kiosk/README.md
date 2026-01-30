# Raspberry Pi Headless Kiosk (Self-Healing)

**Theme:** Linux Automation, X11 Display Management, Cost Optimization.

## 1. The Operational Challenge
I needed to deploy a 24/7 digital presence for a client using low-cost hardware (Raspberry Pi 4). The constraints were:
*   **Budget:** Could not afford enterprise-grade digital signage players.
*   **Reliability:** The device runs unattended. If it crashes, it must recover without human intervention.
*   **Headless:** The application needed to render a graphical environment (X11) internally without a physical monitor connected for debugging.

## 2. The Solution
I architected a "Headless" X11 environment using `Xvfb` (Virtual Framebuffer) and a custom Bash wrapper.

### Key Operational Controls:
1.  **Idempotency on Boot:** The script runs `rm -f /tmp/.X1-lock` immediately. This ensures that if the Pi lost power and left a stale lock file, the new session won't crash on startup.
2.  **Resource Isolation:** Uses `Xvfb :1` to create a virtual display in RAM, completely decoupled from the physical HDMI output.
3.  **Memory Leak Mitigation:** The target application (Electron-based) had known memory leaks over long durations. Rather than engineering a complex patch for the app, I implemented a scheduled nightly reboot via `Cron` to reset the state to a known "clean" baseline (99.9% effective uptime).

## 3. The Automation Code (`legcord_launcher.sh`)
*This script is triggered via `@reboot` in crontab.*

```bash
#!/bin/bash

# 1. Cleanup Stale Locks (Critical for Idempotency)
# The -f flag ensures this doesn't error out on a clean boot
pkill -f Legcord
rm -f /tmp/.X1-lock

# 2. Set Environment
export DISPLAY=:1

# 3. Start Virtual Display
# -ac disables access control (allows local connections without auth cookies)
/usr/bin/Xvfb :1 -screen 0 1280x720x24 -ac &
sleep 5

# 4. Start Window Manager & App
/usr/bin/openbox &
/path/to/Legcord.AppImage &
```

## 4. Why Cron? (Trade-off Analysis)
I chose standard `Cron` over `systemd` timers for this specific implementation because of **portability**. The target environment was a stripped-down Debian distribution where simplicity was preferred over the complexity of writing custom systemd service units.
