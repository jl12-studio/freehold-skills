---
name: freehold-security
description: Security rules for setting up and running Freehold software - protecting the person's keys and logins, preferring browser login over API keys, scoping any unavoidable key to exactly what the setup needs, storing secrets safely, and securing the hub (HTTPS, auth on every endpoint, closed ports, updates). Use before any step that creates an account, logs in to a provider, creates or uses an API key or token, opens a network port, or exposes a Freehold app to the internet. Load alongside freehold-deploy.
---

# Freehold security

The people using Freehold software are often beginners. The defaults must protect them even when they don't know what to ask for. When in doubt, choose the safer option and say why in one sentence.

## Credentials: the order of preference

1. **Browser login through the official CLI.** The person signs in on the provider's own site; the CLI gets a short-lived token. Examples: `wrangler login`, `aws login`, `gcloud auth login`, `az login`. Nobody types or pastes a secret.
2. **A scoped, expiring API token the person creates** in the provider's console, when the CLI has no browser login or the task needs one (a server that deploys itself, a tunnel token).
3. **Never** the account's root or owner keys, a global API key, or the person's password.

## When a key can't be avoided

Give the person the exact recipe from the provider file: URL, button names, which permissions to tick, and nothing else.

- **Narrowest scope.** Only the permissions this setup needs, only on the one account, project, zone or resource group involved.
- **Expiry.** Set an expiry date where the provider allows (a day for setup-only keys; a reasonable rotation period for long-lived ones).
- **IP restriction** where the provider allows and the person has a stable IP.
- **One key per purpose.** The setup key and the server's running key are different keys.
- **Delete setup keys when setup is done.** Do it, or tell the person exactly where to click to do it, before you finish.

## Handling secrets

- **Never ask the person to paste a secret into chat.** Chat logs persist. Instead, have them paste it straight into a terminal prompt that doesn't echo, or into a file you then reference by path. For example, to save a token into a file without it showing on screen or in shell history:

  ```bash
  mkdir -p ~/.config/freehold && (umask 077 && read -rs TOKEN && printf '%s' "$TOKEN" > ~/.config/freehold/token)
  ```

- **Never print a secret** in your own output, logs or files you write. Refer to it by name or path.
- **Store secrets** in the OS keychain (macOS Keychain, Windows Credential Manager, a Linux secret service), the provider's secret manager, or a file with `chmod 600` outside any git repo.
- **Never commit secrets.** Before any commit in a setup folder, check `.gitignore` covers `.env`, token files and `MY-SETUP.md` secrets (there should be none in it anyway).
- **Generate server secrets on the server.** For example `openssl rand -base64 32` run there, written straight to the config file.
- If a secret was exposed (pasted in chat, committed, printed), **say so plainly and rotate it now**: revoke the old one, create a new one.

## Securing the hub

- **HTTPS only.** Valid certificates from the tunnel, platform, or Caddy/Let's Encrypt. No plain HTTP except a redirect to HTTPS.
- **Auth on every endpoint**: MCP, sync API, web portal. No "open on the local network" exceptions once it's reachable from outside.
- **Ports:** only 443 (and 22 for SSH on a VPS; 80 only if Let's Encrypt needs it for certificates, redirecting to HTTPS). Use the provider's firewall as well as the server's.
- **SSH:** key login only, password login off, root login off. Use the provider's console to add the SSH key, never email it.
- **Updates:** turn on automatic security updates for the server OS (`unattended-upgrades` on Debian and Ubuntu).
- **Least privilege on the server:** run the app as its own non-root user; the data directory is readable only by that user.
- **Backups are encrypted** if they leave the hub (`freehold-admin`).

## Money safety

- Set a **budget alert** on every paid provider before creating resources (each provider file has the steps).
- Say the expected monthly cost before creating anything that bills.
- Never enter payment details for the person. They add their card in the provider's console themselves.

## Things to refuse

- Disabling TLS, auth, or the firewall "just to test" on anything reachable from the internet.
- Using a third-party service that would hold the person's provider credentials.
- Installing a CLI or package from anywhere except the provider's official source or a mainstream package manager (Homebrew, winget, apt, npm for npm-published official tools).
