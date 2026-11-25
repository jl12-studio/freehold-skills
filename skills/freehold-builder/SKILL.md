---
name: freehold-builder
description: How to build Freehold software and climb the Freehold grades - the local desktop core, the MCP surface, the person's own cloud hub with hub-and-spoke sync, the web portal, and phone install as a PWA. Use when building a new Freehold app, or when a person or maker says "I want this to sync", "make this work on my phone", "add a web version", "put this in the cloud", or asks how an existing app reaches its next grade.
---

# Building Freehold software

Read `freehold-standard` first if you have not. This skill is about **what to build**. Putting it on a provider is `freehold-deploy`.

This is principles and patterns, not a fixed tech stack. Software changes fast: pick the current, well-maintained option for the app's language, and check its docs before you use it.

## Order of operations

Each step works on its own before the next starts.

1. **Write the scene.** One paragraph: a specific person, a specific moment, what they tell their agent, what they get. If you can't write it, don't build it.
2. **Define what the person owns.** The records, what must never be lost, what gets exported. Prefer append-mostly data (ledgers, logs): it makes sync and conflicts easy.
3. **Local single-user core.** One machine, one owned database file, doing the job. No sync yet. Must already be useful.
4. **MCP surface.** Every core operation is an MCP tool. Drive the whole scene through an agent.
5. **Desktop release and administration.** A signed desktop release an agent can install; backup, restore and verify as MCP tools; migrations; a conflict story (on one computer it can be "nothing to merge"). **Ship here: this is grade 1, Desktop, with every point except 4 met.**
6. **Server release.** A container image or a single binary or package for a server, installable by an agent without a human reading docs.
7. **Sync.** The hub and spokes below, with remote MCP and authenticated, encrypted endpoints. Declare the conflict story across devices. **Grade 2, Cloud.**
8. **Web portal, then PWA.** **Grades 3 and 4.**
9. **Two guides.** `HUMAN-GUIDE.md` (what this is, what you own, the scene) and `AGENT-GUIDE.md` (how to deploy, operate, administer). Plus an `INSTALL.md` an agent can follow end to end.

## Hub and spoke: the shape

```
                    the person's hub (their server, or their always-on machine)
                    - the one source of truth
                    - remote MCP endpoint over HTTPS
                    - sync API over HTTPS
                    - serves the web portal and PWA
                               |
        ---------------------------------------------------
        |                 |                 |              |
   desktop app       web portal         PWA on phone    any agent
   (local copy,      (browser, no       (installed from (remote MCP,
    syncs)            install)           the portal)     token auth)
```

- The **hub is the same app** running in server mode, not a separate product. One codebase, two modes: `desktop` and `hub`.
- The hub can be a cloud server or the person's own always-on computer reached through a tunnel (`freehold-deploy` covers both). Either is Freehold; nobody else runs it.
- Spokes keep a local copy and work offline where the conflict story allows.

Details for each part:
- Sync protocol, device pairing and conflict stories: [references/sync.md](references/sync.md)
- Remote MCP, web portal and PWA: [references/web-and-pwa.md](references/web-and-pwa.md)

## When a person asks for a grade the app doesn't have

Example: "I want Freehold Ledger to sync to my phone," and Ledger is only at Desktop.

1. Say plainly what the app supports today and what's missing (check its `FREEHOLD.md` and release notes).
2. If the app has the feature but it isn't set up, switch to `freehold-deploy`.
3. If the app doesn't have the feature, say so. Don't fake it with a third-party sync service (a shared Dropbox folder for a database file corrupts it, and it puts a middleman back in). Offer to open an issue on the app's repo, or, if the person is the maker, build it with this skill.

## Rules that never bend

- No telemetry, no phone-home, no maker account.
- No hosted service run by the maker, ever. The maker ships releases; the person's agent deploys them.
- No extra API keys for features (self-contained rule). If a feature needs an outside service, it isn't in the app.
- Every network endpoint is authenticated and encrypted from the first release that has one (`freehold-security`).
- Every schema change ships a migration, and the app refuses to open data newer than itself instead of damaging it.
