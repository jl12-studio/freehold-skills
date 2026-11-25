---
name: freehold-admin
description: Operating a Freehold app the person owns - backups, restore tests, verifying data, upgrades with migrations, pinning versions, managing paired devices, and checking costs. Use when a person asks to back up, restore, upgrade, check on, move, or shut down their Freehold app or hub, or after freehold-deploy finishes a setup.
---

# Running a Freehold app

The agent is the sysadmin (point 6 of the standard). Owning your data must not mean owning your data loss.

Use the app's own MCP admin tools when it has them (`backup`, `restore`, `verify`, `upgrade`, `list_devices`, `revoke_device` or similar; check the app's `AGENT-GUIDE.md`). Fall back to the steps below only when it doesn't.

## Backups

- **The 3-2-1 idea, kept simple:** the live data, a backup on the hub's disk, and a copy somewhere else the person owns (their own computer, or a storage bucket in their own provider account).
- Back up on a schedule (daily is a good default) and before every upgrade.
- Back up with the app's own export or a consistent database snapshot, never by copying a database file while the app writes to it.
- Encrypt any copy that leaves the hub. Keep the encryption key in the person's password manager, and say so: lose that key and the backup is unreadable.
- Keep a sensible number (for example 14 daily, 8 weekly) and delete older ones so storage costs stay flat.

## Restore test

A backup nobody has restored is a guess. After setup, and every few months:
1. Restore the latest backup into a scratch location (never over the live data).
2. Run the app's `verify` against it.
3. Tell the person it worked, and the date of the backup you tested.

## Upgrades

1. Read the release notes for every version between current and target. Call out anything marked breaking.
2. Back up.
3. Upgrade to an exact version (never `latest`). The app runs its own migrations.
4. Run `verify`, then make one test change from a spoke.
5. If anything fails: restore the backup and go back to the previous version. Then report what happened.

Pinning a version forever is allowed (point 8). If the person wants to stay put, respect it, and only flag security fixes.

## Devices

- List paired devices now and then; offer to revoke ones the person doesn't recognize or no longer uses.
- Lost phone or laptop: revoke that device's token straight away.

## Health and cost check

When asked to check on things:
1. Is the hub up (HTTPS address responds and asks for auth)?
2. When did the last backup run, and did it succeed?
3. Disk space on the hub.
4. OS security updates pending.
5. This month's bill against the budget alert.

## Moving or shutting down

- **Moving providers:** back up, deploy the hub on the new provider (`freehold-deploy`), restore, re-point spokes, verify, then delete the old resources.
- **Shutting down:** export everything to the person's own computer first, confirm they have it, then delete resources so billing stops. Say what will be deleted and that it can't be undone, and wait for a yes.
