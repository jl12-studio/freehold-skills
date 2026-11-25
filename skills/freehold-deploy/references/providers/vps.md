# A small rented server (VPS): Hetzner or DigitalOcean

Good for: an always-on hub at a low, flat monthly price, with nobody else in the network path. This file also holds the **server baseline** that the AWS, Google Cloud and Azure files reuse once their VM exists.

## Before either provider: an SSH key

Check for one, and make one if missing (the person presses Enter to accept the default path and sets a passphrase):

```bash
ls ~/.ssh/id_ed25519.pub || ssh-keygen -t ed25519 -C "freehold-hub"
```

## Hetzner

**Check first:** https://docs.hetzner.com/cloud/ and https://github.com/hetznercloud/cli (the official `hcloud` CLI).

1. Sign up: https://accounts.hetzner.com/signUp. Turn on two-factor login in the account settings.
2. In the Hetzner Console, create a **new project** just for this hub (for example `freehold`). The API token only reaches its own project, so a dedicated project keeps the token's reach small.
3. In that project: **Security**, **API tokens**, **Generate API token**, permission **Read & Write**, description `freehold-setup`. Hetzner tokens can't be scoped tighter than one project, which is why step 2 matters.
4. Install the CLI (macOS: `brew install hcloud`; others: the GitHub releases page above).
5. Save the token into an `hcloud` context. It prompts for the token without echoing it; the person pastes it there, not in chat:

   ```bash
   hcloud context create freehold
   ```

6. Add the SSH key, pick the smallest current server type, and create the server (fill in the type from the list):

   ```bash
   hcloud ssh-key create --name freehold --public-key-from-file ~/.ssh/id_ed25519.pub
   ```

   ```bash
   hcloud server-type list
   ```

   ```bash
   hcloud server create --name freehold-hub --type <smallest type from the list> --image ubuntu-24.04 --ssh-key freehold
   ```

7. Firewall, only SSH and HTTPS:

   ```bash
   hcloud firewall create --name freehold-hub-fw
   ```

   ```bash
   hcloud firewall add-rule freehold-hub-fw --direction in --protocol tcp --port 22 --source-ips 0.0.0.0/0 --source-ips ::/0
   ```

   ```bash
   hcloud firewall add-rule freehold-hub-fw --direction in --protocol tcp --port 443 --source-ips 0.0.0.0/0 --source-ips ::/0
   ```

   ```bash
   hcloud firewall apply-to-resource freehold-hub-fw --type server --server freehold-hub
   ```

8. Cost: Hetzner bills monthly with an hourly cap; show the person the price `hcloud server-type list` reports. Hetzner has no budget alert; the flat price is the guard.
9. After setup, delete the setup token in the Console if the server doesn't need it (it usually doesn't).

## DigitalOcean

**Check first:** https://docs.digitalocean.com/reference/doctl/ (the official `doctl` CLI).

1. Sign up: https://cloud.digitalocean.com/registrations/new. Turn on two-factor login: https://cloud.digitalocean.com/account/security
2. Token: https://cloud.digitalocean.com/account/api/tokens, **Generate New Token**, name `freehold-setup`, **Expiration** 1 day (or the shortest offered), **Custom Scopes**: only `droplet`, `ssh_key`, `firewall` (and `domain` if setting DNS here). Click **Generate Token**.
3. Install the CLI (macOS: `brew install doctl`; others: the docs above).
4. Save the token (prompts without echo):

   ```bash
   doctl auth init --context freehold
   ```

   Make it the active context, so nothing lands in another DigitalOcean account doctl may already know:

   ```bash
   doctl auth switch --context freehold
   ```

5. Add the SSH key and create the droplet (check `doctl compute size list` for the current smallest size and price):

   ```bash
   doctl compute ssh-key import freehold --public-key-file ~/.ssh/id_ed25519.pub
   ```

   ```bash
   doctl compute droplet create freehold-hub --region nyc3 --size s-1vcpu-1gb --image ubuntu-24-04-x64 --ssh-keys $(doctl compute ssh-key list --format Name,FingerPrint --no-header | awk '$1=="freehold"{print $2}') --wait
   ```

6. Firewall, only SSH and HTTPS in, everything out:

   ```bash
   doctl compute firewall create --name freehold-hub-fw --droplet-ids $(doctl compute droplet list freehold-hub --format ID --no-header) --inbound-rules "protocol:tcp,ports:22,address:0.0.0.0/0,address:::/0 protocol:tcp,ports:443,address:0.0.0.0/0,address:::/0" --outbound-rules "protocol:tcp,ports:all,address:0.0.0.0/0,address:::/0 protocol:udp,ports:all,address:0.0.0.0/0,address:::/0"
   ```

7. Budget alert: https://cloud.digitalocean.com/account/billing, **Billing alerts**, set a monthly amount.

## Server baseline (every VM, any provider)

Run these over SSH as the provider's default user (`ssh root@<ip>` on Hetzner and DigitalOcean; the AWS, Google and Azure files give their own login). Check each step against the current Ubuntu and Docker docs.

1. **Updates now, and automatic security updates from now on:**

   ```bash
   sudo apt update && sudo apt -y upgrade && sudo apt -y install unattended-upgrades && sudo dpkg-reconfigure -plow unattended-upgrades
   ```

2. **A non-root admin user** with the same SSH key (skip if the provider already made one, as AWS, Google and Azure do):

   ```bash
   adduser --disabled-password --gecos "" freehold && usermod -aG sudo freehold && rsync --archive --chown=freehold:freehold ~/.ssh /home/freehold
   ```

   Then set a sudo password for that user (`passwd freehold`, typed on the server, saved in the person's password manager).

3. **SSH: keys only, no root login.** Open a second SSH session as the new user first, so a mistake can't lock the person out. Then:

   ```bash
   printf 'PasswordAuthentication no\nPermitRootLogin no\n' | sudo tee /etc/ssh/sshd_config.d/00-freehold.conf && sudo systemctl reload ssh
   ```

   The file starts with `00` on purpose: sshd keeps the first value it reads, and cloud images often ship a later file (such as `50-cloud-init.conf`) that turns passwords back on. Confirm both settings took:

   ```bash
   sudo sshd -T | grep -E '^(passwordauthentication|permitrootlogin) '
   ```

   Both lines must say `no`.

4. **Host firewall** as a second layer behind the provider's firewall:

   ```bash
   sudo apt -y install ufw && sudo ufw allow OpenSSH && sudo ufw allow 443/tcp && sudo ufw --force enable
   ```

   If the app's guide uses Caddy for certificates, also allow `80/tcp` (Let's Encrypt checks on port 80), here and in the provider firewall.

5. **Run the app** the way its `INSTALL.md` says (a container with a named volume, or a systemd service as its own non-root user). Pin the exact version. If it runs in Docker, publish its port on loopback only (`-p 127.0.0.1:<port>:<port>`): Docker-published ports skip ufw, and Caddy reaches the app on localhost anyway.

6. **HTTPS** with Caddy (automatic Let's Encrypt certificates) in front of the app, if the app doesn't do TLS itself. Point a DNS A record (and AAAA for IPv6) for the hub's name at the server's IP first. Install Caddy from its official instructions (https://caddyserver.com/docs/install), then the whole config is two lines in `/etc/caddy/Caddyfile`:

   ```
   ledger.example.com {
       reverse_proxy localhost:<the app's port>
   }
   ```

   ```bash
   sudo systemctl reload caddy
   ```

7. Visit `https://<hub name>` and confirm the app asks for auth.
