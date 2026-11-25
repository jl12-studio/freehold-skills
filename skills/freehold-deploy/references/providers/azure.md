# Microsoft Azure

Good for: people who already have an Azure or Microsoft 365 business account. For a hub, use a small **Linux VM** in its own resource group. If the app ships a container build, **Azure Container Apps** is an option; check the app's guide.

**Check first:**
- CLI install: https://learn.microsoft.com/cli/azure/install-azure-cli
- `az vm create`: https://learn.microsoft.com/cli/azure/vm#az-vm-create
- Free account services (they change): https://azure.microsoft.com/pricing/free-services

## Account

1. Sign up: https://azure.microsoft.com/free/. The person enters their own card.
2. Turn on MFA for the Microsoft account: https://account.microsoft.com/security

## CLI

macOS:

```bash
brew update && brew install azure-cli
```

Windows (PowerShell):

```powershell
winget install --exact --id Microsoft.AzureCLI
```

Linux (Debian or Ubuntu):

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Log in through the browser (use `az login --use-device-code` on a machine with no browser):

```bash
az login
```

Official MCP server: Azure MCP Server, published by Microsoft at https://github.com/microsoft/mcp. Optional.

## A resource group just for the hub

Everything goes in one resource group, so permissions and deletion are contained:

```bash
az group create --name freehold-hub-rg --location eastus
```

## Budget first

In the portal (https://portal.azure.com), search **Budgets**, then **Add**, scope it to the subscription, the person's monthly amount, alert at 80% and 100% to their email.

## Create the hub

Check the free services page for which small VM sizes the free account covers (historically `Standard_B1s` for 12 months).

```bash
az vm create --resource-group freehold-hub-rg --name freehold-hub --image Ubuntu2404 --size Standard_B1s --admin-username freehold --generate-ssh-keys --public-ip-sku Standard
```

This opens SSH (22) by default. Open HTTPS:

```bash
az vm open-port --resource-group freehold-hub-rg --name freehold-hub --port 443 --priority 900
```

The Standard public IP is static by default. Get it:

```bash
az vm show --resource-group freehold-hub-rg --name freehold-hub --show-details --query publicIps --output tsv
```

Connect with `ssh freehold@<that IP>`, then follow the **server baseline** in [vps.md](vps.md) (skip step 2; `freehold` is already a non-root sudo user).

## If a key is unavoidable

For a CI job that deploys without a person present, use **federated credentials** first: OpenID Connect from GitHub Actions to an app registration, with no password at all. Follow https://learn.microsoft.com/azure/developer/github/connect-from-azure-openid-connect, scoping the role to the one resource group.

Only if the CI can't do that, create a service principal limited to the one resource group (start with **Contributor** on the group only, never the subscription). Its output contains a `password`, so **the agent must not run this**: the person runs it in their own terminal, and the output goes straight into the CI secret store. For GitHub Actions, piped so it never appears on screen:

```bash
az ad sp create-for-rbac --name freehold-deploy --role Contributor --scopes /subscriptions/$(az account show --query id --output tsv)/resourceGroups/freehold-hub-rg | gh secret set AZURE_CREDENTIALS --repo <owner>/<repo>
```
