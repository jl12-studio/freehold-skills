# Google Cloud

Good for: people who already have a Google Cloud account, or who want the always-free small VM. For a hub, use **Compute Engine** (a VM). If the app ships a serverless container build, **Cloud Run** is an option; check the app's guide.

**Check first:**
- CLI install: https://cloud.google.com/sdk/docs/install (Homebrew: https://cloud.google.com/sdk/docs/downloads-homebrew)
- Free tier limits (they change): https://cloud.google.com/free/docs/free-cloud-features#compute
- `gcloud compute instances create`: https://cloud.google.com/sdk/gcloud/reference/compute/instances/create

## Account

1. Sign up: https://console.cloud.google.com/freetrial. The person enters their own card.
2. Google account security (two-step verification): https://myaccount.google.com/security

## CLI

macOS:

```bash
brew install --cask gcloud-cli
```

Windows and Linux: the installer from the install docs above.

Log in through the browser and pick defaults:

```bash
gcloud init
```

## A project just for the hub

A dedicated project keeps permissions and the bill contained. Project IDs are globally unique, so add a few random characters:

```bash
gcloud projects create freehold-hub-<6 random letters> --name="Freehold hub"
```

```bash
gcloud config set project freehold-hub-<same letters>
```

Link billing (the person picks their billing account from the first command's output):

```bash
gcloud billing accounts list
```

```bash
gcloud billing projects link freehold-hub-<same letters> --billing-account=<ACCOUNT_ID>
```

```bash
gcloud services enable compute.googleapis.com
```

## Budget first

https://console.cloud.google.com/billing/budgets, **Create budget**, scope it to the hub project, the person's monthly amount, alerts at 50%, 90% and 100%.

## Create the hub

The always-free tier has historically covered one `e2-micro` in `us-west1`, `us-central1` or `us-east1` with a 30 GB standard disk. Confirm on the free tier page before promising $0, and say a public IPv4 address may bill separately.

```bash
gcloud compute instances create freehold-hub --zone=us-central1-a --machine-type=e2-micro --image-family=ubuntu-2404-lts-amd64 --image-project=ubuntu-os-cloud --boot-disk-size=30GB --boot-disk-type=pd-standard --tags=freehold-hub
```

Allow HTTPS to the hub (the default network already allows SSH):

```bash
gcloud compute firewall-rules create freehold-hub-https --allow=tcp:443 --target-tags=freehold-hub
```

The default network also opens Windows remote desktop (port 3389) to the whole internet. Close it:

```bash
gcloud compute firewall-rules delete default-allow-rdp --quiet
```

Reserve the current IP as static so DNS doesn't break:

```bash
gcloud compute addresses create freehold-hub-ip --region=us-central1 --addresses=$(gcloud compute instances describe freehold-hub --zone=us-central1-a --format='get(networkInterfaces[0].accessConfigs[0].natIP)')
```

Connect (gcloud creates and manages the SSH key):

```bash
gcloud compute ssh freehold-hub --zone=us-central1-a
```

Then follow the **server baseline** in [vps.md](vps.md) (skip step 2; gcloud already made a non-root sudo user).

## Keys

Avoid service account keys. If something must deploy without a person present, use the VM's own attached service account with only the roles it needs, or Workload Identity Federation from CI. A downloaded JSON key is the last resort, and it gets an expiry and deletion date.
