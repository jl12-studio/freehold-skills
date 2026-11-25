# Freehold Skills

Agent skills for [Freehold](https://freeholders.org) software. They teach any agent what Freehold is, how to build each Freehold grade, and how to put a Freehold app on infrastructure the person owns, safely, with exact steps.

Freehold apps point here instead of each one re-explaining cloud setup, sync and security. The app's own guide covers what's specific to that app; these skills cover everything shared.

## The skills

| Skill | What it does |
|---|---|
| [freehold-standard](skills/freehold-standard/SKILL.md) | What Freehold is, the 10 points, the grades (desktop, cloud hub, web portal, PWA), and how to check an app against them. |
| [freehold-builder](skills/freehold-builder/SKILL.md) | How to build each grade: local core, MCP surface, hub-and-spoke sync, conflict stories, remote MCP, web portal, PWA. |
| [freehold-deploy](skills/freehold-deploy/SKILL.md) | The setup interview, then provider-by-provider steps with exact URLs and commands: home hub with a tunnel, Cloudflare, Hetzner, DigitalOcean, AWS, Google Cloud, Azure. |
| [freehold-security](skills/freehold-security/SKILL.md) | Browser login over keys, narrowly scoped and expiring keys, secrets never in chat, a locked-down hub, budget alerts. |
| [freehold-admin](skills/freehold-admin/SKILL.md) | Backups, restore tests, upgrades, devices, health and cost checks, moving and shutting down. |

## Install

**Claude Code** (as a plugin):

```bash
claude plugin marketplace add jl12-studio/freehold-skills
```

```bash
claude plugin install freehold@freehold-skills
```

**Any other agent:** the skills are plain Markdown in the open `SKILL.md` format. Clone this repo and point your agent's skills folder at `skills/`, or have the agent read the files straight from GitHub.

## For app makers: how to reference this

In your app's `INSTALL.md` or `AGENT-GUIDE.md`, add a section like this, with the version you tested against:

```markdown
## Infrastructure, sync and security

This app follows the Freehold skills, version 0.1.0:
https://github.com/jl12-studio/freehold-skills/tree/v0.1.0

For setting up a hub, sync, backups or keys, use the freehold-deploy, freehold-security
and freehold-admin skills from that version. App-specific details (ports, data directory,
release names, MCP admin tools) are below.
```

Then list the app-specific facts the skills need: the release artifacts per target, the port, the data directory, the hub-mode command, and the names of the app's admin MCP tools.

**Pin a tag, never `main`.** Your app must keep working if this repo changes or disappears, so your own guide still carries the app-specific steps. The leader is not the movement.

## Adding a provider

Copy the shape of an existing file in `skills/freehold-deploy/references/providers/`:

1. **Good for:** one line.
2. **Check first:** links to the provider's official docs for the CLI install and login.
3. **Account:** signup URL, turning on two-factor login.
4. **CLI:** the official CLI, install command per OS, browser login if it has one.
5. **Budget first:** where to set a spending alert.
6. **Create the hub:** copy-paste commands, smallest sensible size, firewall with only the needed ports, a static IP.
7. **If a key is unavoidable:** exact console path, the narrowest permissions, expiry.

Then add it to the list in `freehold-deploy/SKILL.md`. Only official CLIs and official provider MCP servers.

## Versioning

Semantic versions, tagged (`v0.1.0`). Changes are in [CHANGELOG.md](CHANGELOG.md). Provider details go stale; every provider file tells the agent to check the provider's current docs before running commands.

## License

MIT. See [LICENSE](LICENSE).
