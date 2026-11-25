# Cloudflare

Good for: exposing a home hub with **Cloudflare Tunnel** (free), DNS for a custom address, and apps that ship a Workers or Pages build. Most Freehold setups stay on the free plan.

**Check first:** https://developers.cloudflare.com/workers/wrangler/install-and-update/ and https://developers.cloudflare.com/cloudflare-one/ (Cloudflare Tunnel lives under Networks, Connectors). Confirm commands there before running them.

## Account

- Sign up: https://dash.cloudflare.com/sign-up
- Turn on two-factor login right away: https://dash.cloudflare.com/profile/authentication
- A custom domain is needed for a named tunnel. If the person has one elsewhere, they can add it at https://dash.cloudflare.com (Add a domain) and change the nameservers at their registrar; give them the two nameserver names Cloudflare shows.

## CLIs

- **wrangler** (Workers, Pages, R2, D1). Needs Node.js. Install inside the project, or run without installing:

  ```bash
  npx wrangler@latest login
  ```

  This opens the browser; the person clicks **Allow**. Confirm with:

  ```bash
  npx wrangler@latest whoami
  ```

- **cloudflared** (Tunnel). macOS:

  ```bash
  brew install cloudflared
  ```

  Windows: `winget install --id Cloudflare.cloudflared`. Linux: Cloudflare's package repo, per the docs.

Official MCP servers: https://github.com/cloudflare/mcp-server-cloudflare (published by Cloudflare). Optional; the CLIs are enough.

## Tunnel to a home hub

The easiest path is a dashboard-managed tunnel, which needs no API key:

1. Go to https://one.dash.cloudflare.com, then **Networks**, **Connectors** (or **Tunnels**), **Create a tunnel**, choose **Cloudflared**.
2. Name it after the app (for example `freehold-ledger-hub`).
3. Cloudflare shows an install command containing a token. Have the person run it themselves in a terminal on the hub machine (it installs `cloudflared` as a service). **The token is a secret: they paste it in the terminal, not in chat.**
4. Add a **Public hostname**: subdomain (for example `ledger`), their domain, service type **HTTP**, URL `localhost:<the app's port>`.
5. Visit `https://ledger.<their domain>` and confirm the app asks for auth.

Optional extra lock: put **Cloudflare Access** in front of the hostname (Access, Applications, Add an application, Self-hosted) with an email one-time PIN policy limited to the person's own email. Note this also gates the agent's remote MCP calls; use a service token policy for those, or skip Access and rely on the app's own auth.

## If a key is unavoidable

For example a CI job or a server that deploys itself.

1. Go to https://dash.cloudflare.com/profile/api-tokens and click **Create Token**.
2. Start from a template if one fits (**Edit Cloudflare Workers**), otherwise **Create Custom Token**.
3. Permissions, only what the job needs:
   - Workers deploy: Account, **Workers Scripts**, Edit
   - Pages deploy: Account, **Cloudflare Pages**, Edit
   - DNS record: Zone, **DNS**, Edit
   - Tunnel management: Account, **Cloudflare Tunnel**, Edit
4. **Account Resources:** Include, the one account. **Zone Resources:** Include, Specific zone, the one domain.
5. **Client IP Address Filtering:** the server's IP, if it's fixed.
6. **TTL:** an end date (tomorrow for a setup-only token).
7. Click **Continue to summary**, **Create Token**. The person copies it once into the terminal or secret store (see `freehold-security`), never into chat.

Never use the **Global API Key**.

## Cost guard

Workers, Pages, Tunnel and DNS have free plans that cover a personal hub. If the person upgrades to a paid plan, set a usage notification at https://dash.cloudflare.com under **Notifications**, **Add** (look for the usage-based billing alert).
