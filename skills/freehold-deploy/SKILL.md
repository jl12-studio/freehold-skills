---
name: freehold-deploy
description: Walks a person through putting a Freehold app on infrastructure they own - their own computer, a cloud provider (Cloudflare, AWS, Google Cloud, Azure) or a plain server (Hetzner, DigitalOcean) - and setting up hub-and-spoke sync. Starts with an interview (do you have an account, which provider, what budget), installs the provider's official CLI with consent, logs in through the browser instead of keys wherever possible, scopes any unavoidable API key narrowly, and gives exact URLs and copy-paste commands at every step. Use when a person says "set this up in the cloud", "I want this to sync", "deploy this", "put this on my server", or an app's INSTALL.md points here.
---

# Deploying Freehold software

The person owns everything this skill creates: the account, the server, the data, the bill. Nobody else runs any of it. Your job is to do as much as you safely can, and to make every step they do themselves easy and exact.

Before anything touches keys or accounts, load `freehold-security`. Its rules apply to every step here.

## How you behave

- **Helpful from the first message.** Don't hand them a list of docs. Ask, then act.
- **Never lazy.** Every step the person does gets: the exact URL, what to click (button names as they appear), and copy-paste commands with every value filled in. One command per code block.
- **Consent before change.** Say what you are about to install or create and what it will cost, then wait for a yes. Installing a CLI, creating a server, opening a port, and creating a key each need their own yes.
- **Check current docs.** Provider consoles, CLI flags and free tiers change. Before running provider commands, open that provider's official docs (links in each provider file) and confirm the install command, login command and any flag you plan to use. If the docs disagree with this skill, follow the docs and tell the person.
- **Teach while you work.** One plain sentence on what each step is for.

## Step 1: The interview

Ask these together, with a suggested answer for each, so the person can reply in one message:

1. **Where should the hub live?**
   - **Your own always-on computer** (cheapest, data never leaves your home; needs that computer on). Reached from outside through a tunnel.
   - **A cloud server you rent** (always on, about $4 to $7 a month for a small one).
   - **A serverless platform** (Cloudflare, Google Cloud Run, AWS, Azure). Only if the app's `INSTALL.md` says it supports one.
2. **Do you already have an account** with any provider? (Cloudflare, AWS, Google Cloud, Azure, Hetzner, DigitalOcean, or another.) An existing account usually wins: one less signup, one less bill.
3. **Monthly budget**, even if it's $0.
4. **A domain?** Not required. Needed only for a custom address like `ledger.yourname.com`.
5. **Which computer are we on** (macOS, Windows, Linux) and can I run commands in a terminal here?

Then read the app's own `INSTALL.md` and `AGENT-GUIDE.md`. They say which targets the app ships for (container image, binary, serverless build). **Only offer targets the app actually supports.**

## Step 2: Recommend one path

Give one recommendation with one sentence of why, and the monthly cost. Defaults when the person has no preference:

| Situation | Recommend |
|---|---|
| Has an always-on computer, budget $0 | Home hub + Cloudflare Tunnel (free) |
| Wants it always on, cheapest, simplest | A small VPS (Hetzner or DigitalOcean) running the app's container |
| Already has an AWS, Google Cloud or Azure account | That provider's small VM (Lightsail, Compute Engine, Azure VM) |
| App ships a serverless build | That platform, per the app's guide |

## Step 3: Provider setup

Open the matching provider file and follow it. Each one covers signup, the official CLI install, browser login, the narrowest key recipe when a key can't be avoided, and a budget alert.

- [references/providers/home-and-tunnel.md](references/providers/home-and-tunnel.md): the person's own computer, exposed with Cloudflare Tunnel or Tailscale
- [references/providers/cloudflare.md](references/providers/cloudflare.md)
- [references/providers/vps.md](references/providers/vps.md): Hetzner and DigitalOcean
- [references/providers/aws.md](references/providers/aws.md)
- [references/providers/gcp.md](references/providers/gcp.md)
- [references/providers/azure.md](references/providers/azure.md)

A provider that isn't listed: use the same shape (see "Adding a provider" in this repo's README). Find the provider's official CLI and its official docs, prefer browser login, scope keys narrowly.

### Which tool to trust, in order

1. **The provider's own official CLI** (wrangler, aws, gcloud, az, hcloud, doctl). Built by the provider, inspectable, works with any agent, and most log in through the browser so no raw key is handled.
2. **The provider's own official MCP server or agent plugin**, if one exists and the person's agent supports it. Confirm it is published by the provider itself (their domain, their GitHub org).
3. **Never** a third-party CLI, MCP server or website that would hold the person's provider credentials.

## Step 4: Deploy the hub

1. Pull the app's release for the target (pin the exact version; never `latest` in production).
2. Run it in hub mode with a persistent data directory or volume. The data must survive restarts and upgrades.
3. Put HTTPS in front (the tunnel, the platform's own TLS, or a reverse proxy such as Caddy with automatic certificates).
4. Generate the hub's admin secret on the server itself (see `freehold-security`). Never type it into chat.
5. Open only the ports needed (443, and 22 for SSH if it's a VPS). Everything else closed.
6. Check it: visit the HTTPS address, confirm it asks for auth, run the app's `verify` tool.

## Step 5: Connect the spokes

1. Pair each device with a one-time code (the app's pairing flow; see `freehold-builder` references/sync.md).
2. Point the person's agent at the hub's remote MCP address with its device token.
3. For phones: open the web portal and install it as a PWA (exact taps are in `freehold-builder` references/web-and-pwa.md).
4. Make one test change on one device and watch it arrive on another.

## Step 6: Hand over

Before you finish:
1. Turn on backups and run one restore test (`freehold-admin`).
2. Confirm the budget alert exists.
3. Revoke or delete any setup-only key you created (`freehold-security`).
4. Write a short `MY-SETUP.md` in a folder the person chooses: provider, region, server name, hub address, where backups go, how to log in to the console, the monthly cost. **No secrets in it.**
5. Tell the person in three or four sentences what they now own and what it costs.
