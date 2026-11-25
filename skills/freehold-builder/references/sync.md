# Hub-and-spoke sync

The pattern that fits most Freehold apps. Adapt it; don't skip the conflict story.

## Data shape: an operation log

Store changes as an append-only log of operations, not as rows that get overwritten.

Each operation has:
- `op_id`: a random unique id made on the device that created it (UUIDv7 works well; it sorts by time).
- `device_id`: which spoke made it.
- `created_at`: the device's clock, for display only. Never trust it for ordering.
- `hub_seq`: a number the hub assigns when it accepts the operation. **This is the order of truth.**
- `kind` and `payload`: what happened ("record expense", "void transaction 42").

The current state is the log replayed in `hub_seq` order. Keep a materialized table for speed, but it can always be rebuilt from the log.

Ledgers are the easy case: corrections are new operations ("void 42", "correct 42"), so nothing is ever overwritten and two devices can never fight over one row.

## The sync loop

On every spoke, on a timer and on app focus:

1. **Push.** Send local operations that have no `hub_seq` yet. The hub accepts each one exactly once (dedupe by `op_id`), assigns `hub_seq`, and returns it.
2. **Pull.** Ask for every operation with `hub_seq` greater than the last one this spoke has seen. Apply them in order.
3. **Save the cursor** (the highest `hub_seq` seen) only after the pulled operations are applied.

Both calls are idempotent, so a dropped connection just means trying again.

Two rules make the loop safe:
- **The hub hands out `hub_seq` from one serialized writer** (one database transaction at a time for accepting operations), so numbers are committed in order. If the hub ever accepts in parallel, a pull must only return operations up to the highest contiguous committed number; otherwise a spoke can save cursor 11 while 10 is still committing and never receive 10.
- **Spokes skip any `op_id` they already hold.** A pull returns the spoke's own pushed operations too; applying them twice would duplicate rows in the materialized table.

## Pairing a device

Never ask the person to copy a master password between devices.

1. On the hub (or through the agent, with an MCP tool such as `create_pairing_code`), make a one-time code that expires in 10 minutes.
2. The new device sends the code plus its `device_id` and a name ("Marcus's phone").
3. The hub returns a **device token**: long random, stored on the hub only as a hash, revocable on its own.
4. The device keeps the token in the OS keychain (desktop) or as a secure, HTTP-only cookie (browser and PWA).
5. Expose `list_devices` and `revoke_device` as MCP tools so the agent can manage them.

## Conflict stories (pick one and write it down)

Write the chosen story in the app's `FREEHOLD.md` under "Conflict story", in plain words a person can read.

- **Append-only (best).** Every change is a new operation. Two devices offline simply both add theirs; nothing conflicts. Fits ledgers, journals, logs, notes-as-entries.
- **Last writer wins, per field.** For simple settings-like data. The operation with the higher `hub_seq` wins. Say so plainly: "if you change the same field on two devices while offline, the last one to sync wins."
- **Flag for the person.** When two offline edits touch the same thing and losing either would hurt, keep both and ask the person (or their agent) to choose. Expose `list_conflicts` and `resolve_conflict` as MCP tools.
- **No offline edits.** Spokes are read-only while offline. Honest and valid; say it.

## What not to do

- Don't sync the database file itself through a file-sync service. It corrupts, and it adds a middleman.
- Don't order by device clocks.
- Don't let a spoke rewrite history. Corrections are new operations.
