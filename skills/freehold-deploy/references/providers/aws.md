# AWS

Good for: people who already have an AWS account. For a hub, use **Lightsail** (a simple VM with a flat monthly price) rather than raw EC2: fewer moving parts, and the price includes storage and some data transfer.

**Check first:**
- CLI install: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
- Browser login (`aws login`): https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sign-in.html
- Lightsail CLI: https://docs.aws.amazon.com/cli/latest/reference/lightsail/

## Account

1. Sign up: https://aws.amazon.com/free/ (**Create a Free Account**). The person enters their own card.
2. Protect the root user with MFA: https://console.aws.amazon.com/iam/home#/security_credentials, **Assign MFA device**.
3. **Never create access keys for the root user.**
4. Make a day-to-day user instead of using root. In IAM (https://console.aws.amazon.com/iam/home#/users), **Create user**:
   - Name `freehold-admin`, tick **Provide user access to the AWS Management Console**.
   - Permissions: **Attach policies directly**, attach `SignInLocalDevelopmentAccess` (lets the CLI use browser login), then **Create policy** with the JSON below and attach it too. It allows only Lightsail.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       { "Effect": "Allow", "Action": "lightsail:*", "Resource": "*" }
     ]
   }
   ```

   Name the policy `freehold-lightsail-only`.
5. **Still signed in as root**, give this user MFA: IAM, **Users**, `freehold-admin`, **Security credentials**, **Assign MFA device**. (The user can't do this itself; its policy only allows Lightsail.) The person signs in as `freehold-admin` from now on.

## CLI

Install AWS CLI v2 (version 2.32.0 or later is needed for `aws login`).

macOS:

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg" && sudo installer -pkg AWSCLIV2.pkg -target / && rm AWSCLIV2.pkg
```

Windows (PowerShell):

```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

Linux: the zip installer from the install docs above.

Log in through the browser. No access keys are stored; the CLI gets short-lived credentials that it refreshes itself:

```bash
aws login
```

On a machine with no browser, `aws login --remote` prints a URL to open on another device. Confirm:

```bash
aws sts get-caller-identity
```

When setup is finished, `aws logout` clears the cached credentials.

If the person's organization uses IAM Identity Center, use `aws configure sso` and `aws sso login` instead, per the docs.

Official MCP servers: https://github.com/awslabs/mcp (published by AWS Labs). Optional.

## Budget first

Before creating anything: https://console.aws.amazon.com/billing/home#/budgets, **Create budget**, **Use a template**, **Monthly cost budget**, the person's monthly amount, their email. (Budgets need billing access: do this as root, or grant the admin user billing access.)

## Create the hub on Lightsail

Pick a region near the person and the smallest bundle that meets the app's requirements:

Fill in `<region>` with the person's choice (for example `us-east-1` or `eu-central-1`) everywhere below; the availability zone is the region plus `a`.

```bash
aws lightsail get-bundles --region <region> --query "bundles[?isActive].[bundleId,price,ramSizeInGb]" --output table
```

```bash
aws lightsail create-instances --region <region> --instance-names freehold-hub --availability-zone <region>a --blueprint-id ubuntu_24_04 --bundle-id <bundle from the list>
```

Lightsail opens ports 22 and 80 by default. Set exactly 22 and 443 (add 80 back only if the app uses Caddy for certificates):

```bash
aws lightsail put-instance-public-ports --region <region> --instance-name freehold-hub --port-infos fromPort=22,toPort=22,protocol=tcp fromPort=443,toPort=443,protocol=tcp
```

Give it a fixed IP so DNS doesn't break on restart:

```bash
aws lightsail allocate-static-ip --region <region> --static-ip-name freehold-hub-ip && aws lightsail attach-static-ip --region <region> --static-ip-name freehold-hub-ip --instance-name freehold-hub
```

SSH key: download the default key pair for the region from the Lightsail console (**Account**, **SSH keys**) or connect with the browser SSH button. Login user is `ubuntu`.

Then follow the **server baseline** in [vps.md](vps.md) (skip step 2; `ubuntu` is already a non-root sudo user).

## Backups

Lightsail snapshots are a convenient extra (`aws lightsail create-instance-snapshot`), but they bill per GB and they aren't a substitute for the app's own backups in `freehold-admin`.
