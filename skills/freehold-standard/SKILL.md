---
name: freehold-standard
description: What Freehold software is and the 10-point Freehold Standard, plus the Freehold grades (desktop, cloud hub, web portal, phone install as a PWA). Use when someone asks what Freehold means, whether an app is Freehold, which grade an app has reached, or what an app still needs to climb to the next grade. Also the starting point before using freehold-builder, freehold-deploy, freehold-security or freehold-admin.
---

# The Freehold Standard

**Freehold software is software the person owns outright:** free license, their data, their deployment, their choice of providers. No landlord, no middleman. Subscription software is leasehold; Freehold is the opposite.

Freehold is a status, not a brand. Any software that meets the standard may call itself Freehold. The canonical text lives at https://freeholders.org/standard. If you can reach it, read it: it wins over this summary when they differ.

## The 10 points

1. **Free, and owned by the person.** Free license, their data, their deployment. No telemetry, no phone-home, no account with the maker.
2. **Deployable anywhere.** Pre-built releases for every reasonable target. The person's agent performs the deployment. The maker never runs infrastructure for anyone.
3. **Speaks MCP natively.** The app ships its own MCP server, local and remote.
4. **Hub-and-spoke sync, self-managed.** The person's own instance is the single source of truth; other devices are clients of it. Sync is set up by the software itself, never a service someone else runs.
5. **A declared conflict story.** The app states how it handles offline edits, even if the answer is "you can't edit offline."
6. **Agent-operable administration.** Backup, restore, verify and upgrade are MCP tools.
7. **Secure by default.** Authenticated endpoints and encrypted transport out of the box.
8. **Self-carrying upgrades.** Upgrades carry their own data migrations; pinning a version forever keeps working.
9. **Usable by conversation.** The main UX is the person telling their agent what they need.
10. **Teaches while it works.** It builds the person's understanding, not just does the task.

**The self-contained rule.** An app adds no API keys or outside accounts of its own. The only setup cost is infrastructure the person chooses (their machine, their server). The agent's own key is the one key the person brings.

## The Freehold grades

The grades are the road; all ten points are the top of it. A project can start on the first rung and climb.

| Grade | What it means | Points checked |
|---|---|---|
| 1. Desktop | Open source, runs on the person's own computer, the checklist is published. | Every point except 4, at desktop scope: point 2 is a desktop release the agent installs, point 3 is the local MCP server, point 5 can be "one computer, nothing to merge", point 7 can be "nothing listens on the network". Backups (6) and migrations (8) are required already. |
| 2. Cloud | Deploys to a server the person controls. That instance becomes the hub their devices sync to. A sync hub, not a public website. | Adds point 4, plus the server side of 2 (server releases), 3 (remote MCP), 5 (a conflict story across devices) and 7 (authenticated, encrypted endpoints). |
| 3. Web portal | The hub serves a site the person can open from any browser, so nothing needs installing. | Reach from any device |
| 4. Phone install (PWA) | The web portal installs onto the phone's home screen from the browser, free, no store. Store listings are optional extras, never required. | Phone use |

## Checking an app

When asked whether an app is Freehold or what grade it has reached:

1. Read the app's own `FREEHOLD.md`, `README.md`, install guide and agent guide if they exist.
2. Go point by point. For each, answer **met**, **not met**, or **not applicable at this grade** (at Desktop, only point 4 is not applicable), with one line of evidence (a file, a command, a behavior you saw). Never mark a point met on the maker's word alone when you can check it.
3. Name the grade reached and the single next step to the next grade.
4. For the next step, hand off: building a rung is `freehold-builder`, putting it on a provider is `freehold-deploy`, keys and auth are `freehold-security`, backups and upgrades are `freehold-admin`.

## How to talk about it

- Freehold is a philosophy and a movement. Never call it a manifesto.
- Apps are named Freehold + plain noun (Freehold Ledger, Freehold Invoices).
- Teach as you go (point 10): say what you are doing and why in plain words, so the person could do it themselves next time.
