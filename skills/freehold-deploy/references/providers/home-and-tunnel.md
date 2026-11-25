# The person's own computer as the hub

Cheapest option ($0) and the data never leaves their home. The trade: the computer must stay on and awake, and their home internet is the hub's internet.

Be honest about one thing: a tunnel service sits in the network path. It is infrastructure the person chooses, like renting a server, and it never holds their data at rest. If they'd rather have nobody in the path, use a VPS instead (`vps.md`).

## Keep it awake

- **macOS:** System Settings, **Energy** (or **Battery**, **Options** on a laptop), turn on **Prevent automatic sleeping when the display is off**, and keep it plugged in.
- **Windows:** Settings, **System**, **Power & battery**, **Screen and sleep**, set "When plugged in, put my device to sleep after" to **Never**.
- **Linux:** disable suspend in the desktop's power settings.

Run the app in hub mode as a background service that starts at login (the app's `INSTALL.md` says how: a launchd agent on macOS, a Windows service, a systemd unit on Linux).

## Option A: Cloudflare Tunnel (reachable from anywhere)

Public HTTPS address, works from any phone or browser with no extra app. Needs a free Cloudflare account and a domain on Cloudflare. Steps in [cloudflare.md](cloudflare.md), section "Tunnel to a home hub".

## Option B: Tailscale (private, only your own devices)

Nothing is exposed to the public internet; only devices signed in to the person's own Tailscale network can reach the hub. Each spoke (phone included) needs the Tailscale app.

**Check first:** https://tailscale.com/kb

1. Install on the hub and on each device: https://tailscale.com/download
2. Sign in on each (browser login; no keys).
3. On the hub, serve the app over HTTPS inside the private network (check the current command in the Tailscale docs for `tailscale serve`):

   ```bash
   tailscale serve --bg <the app's port>
   ```

4. Tailscale prints an `https://<machine>.<tailnet>.ts.net` address. Use it as the hub address for pairing.

Tailscale's coordination server is run by the Tailscale company. For someone who wants to run that part too, Headscale (https://github.com/juanfont/headscale) is the open source replacement; it needs its own small server, so it's rarely worth it for one person.

## Checks before handing over

- The hub restarts on its own after a reboot (reboot once and confirm).
- Backups also go somewhere other than this computer (`freehold-admin`).
