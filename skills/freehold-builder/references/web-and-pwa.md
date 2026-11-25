# Remote MCP, web portal, and PWA

All three are served by the hub, from the same app, over HTTPS.

## Remote MCP

- Serve the app's MCP tools over the current MCP remote transport (Streamable HTTP at the time of writing; check https://modelcontextprotocol.io for the current spec before building).
- Require auth on every request. Two good options:
  - **Device or agent tokens** from the pairing flow in [sync.md](sync.md). Simple, works with every agent client that accepts a bearer token.
  - **OAuth as the MCP spec describes it**, with the hub as its own authorization server. More work, nicer for clients that support it. No outside identity provider is required, which keeps the self-contained rule.
- Give tools that change or remove data clear names and descriptions so the agent asks the person before using them.

## Web portal (grade 3)

- The hub serves a web UI at its own address (for example `https://ledger.example.com`).
- Sign-in is the pairing flow: the person gets a code from their agent or desktop app and enters it once per browser. No passwords to forget, no email service needed.
- The portal is a spoke like any other: it reads and writes through the same sync API.
- Keep the conversation first (point 9). The portal is for looking things over and quick entries, not a replacement for the agent.

## PWA, phone install (grade 4)

The portal becomes installable. No store, no fee.

Checklist:
1. **HTTPS** on the hub (required for install and service workers).
2. **Web app manifest** linked from every page: `name`, `short_name`, `start_url`, `display: "standalone"`, `theme_color`, `background_color`, icons at 192px and 512px plus a maskable icon.
3. **Service worker** that caches the app shell so it opens offline.
4. **Offline writes** (if the conflict story allows): queue operations in IndexedDB and push them on reconnect with the normal sync loop.
5. **Install instructions to give the person:**
   - iPhone or iPad (Safari): open the portal, tap the Share button, then **Add to Home Screen**.
   - Android (Chrome): open the portal, tap the menu, then **Install app** (or **Add to Home screen**).
6. Test on a real phone before calling grade 4 done.

Check current browser support (MDN's PWA docs at https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps) before building, since iOS support changes between releases.
