# Dead strands from before a reboot hold their names and fail a resumed row

`reclaim-dead-strand-names` · Note · bug, reed, shuttle

Imported from [issue #799](https://github.com/Knatte18/loomyard/issues/799).

## Seen (orch, 2026-10-21, OSL-1046)

After a machine restart, both in-flight runs were resumed from the prime with `lyx batten run <slug> --window`.
Reed still listed the strands from before the restart as dead (`live: false`), holding their names.

- `test-tiers`: Plan-Burler-Fix re-spawned its fixer as `ly:test-tiers:plan-burler-fix-2`, and the row failed with `seatengine: strand "ly:test-tiers:plan-burler-fix" started as "ly:test-tiers:plan-burler-fix-2", because another strand holds that name; way forward: run "lyx reed remove --name ly:test-tiers:plan-burler-fix", then re-step the row`.
- `tuned-defaults-template`: Merge-In's conflict agent came up as `conflict-2` beside the dead `conflict` and carried on.

## Workaround

`lyx reed remove --name <dead strand>` in the task worktree, then `lyx loom resume`.

## Expected

A strand whose pane is gone, such as every strand after a reboot, does not hold its name: a resume or a re-spawn reclaims the name of a dead strand without a manual `lyx reed remove`, and the row does not fail.

## From issue #802

Imported from [issue #802](https://github.com/Knatte18/loomyard/issues/802).

## What was seen

In the quarry hub, the machine slept while task `csharp-alphabet` was at the Webster producer.
The sleep killed the Webster master strand `qry:csharp-alphabet:webster` and its batch fork.
The orch ran `lyx batten run csharp-alphabet --window` and then `lyx loom resume`.
Webster respawned its master as a new strand, `qry:csharp-alphabet:webster-2`.
The dead strand's registration (guid `6726c0ae5bdc360f7e880256ad6153af`, `live: false`, no pane) stayed in `lyx reed list`.
The name `qry:csharp-alphabet:webster` was still taken, so the new strand got the `-2` suffix.

## Workaround

The orch removed the dead registration by hand with `lyx reed remove 6726c0ae5bdc360f7e880256ad6153af` from the task worktree.

## Expected

When a strand is respawned for a role whose previous strand is dead (not live, no pane), reed or the respawning caller drops the dead registration first, and the new strand keeps the canonical name.
A lookup by role name then cannot hit the dead record.


## From issue #811

Imported from [issue #811](https://github.com/Knatte18/loomyard/issues/811).

A `lyx loom` run halted as `failed` (anomaly `producer-hard-failure`) at the `Plan-Burler-Fix` row, round 3 attempt 3, after the plan review loop had already cycled through two stuck rounds.

lyx's own output:

```
shedadapters: Plan-Burler-Fix (burler-fix): round 3 attempt 3: fix: burler: shuttle run: seatengine: strand "ly:<slug>:plan-burler-fix" started as "ly:<slug>:plan-burler-fix-2", because another strand holds that name; way forward: run "lyx reed remove --name ly:<slug>:plan-burler-fix", then re-step the row
```

A second halt note for the same halt carried only `loop-exited`, with the trace pointing at `.lyx/shed/<slug>/steps/loop.log`.

Why this is abnormal:
- The earlier fix attempts of the same row had finished (their history entries are `stuck`), so their strand should have been released.
- A leftover strand from a finished attempt turns a routine retry into a hard failure that ends the whole run.
- The halt needs an operator to run `lyx reed remove` by hand before the run can continue.
- The `loop-exited` note gives no reason, so the real cause is only visible in the other note.

What to do:
- Have the row's attempt teardown remove its strand, or have `seatengine` reclaim a stale strand of the same name instead of renaming and failing.
- If a live holder is genuinely possible, make the retry path run the suggested `lyx reed remove` itself when the holder is provably dead.
- Make the `loop-exited` halt note carry the underlying error.

Provenance: task `test-tiers`; notes `loom-halt-2.md`, `loom-halt.md`.
