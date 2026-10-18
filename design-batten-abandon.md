# A batten verb that ends a paused or blocked run and removes its records

`batten-abandon` · Note · enhancement, batten, fabric

## Seen

On 2026-10-18 the records held four batten run records with no pair and no board entry:

- `orch-rules-in-weft`, blocked at Worktree-Create because its slug ended in the reserved `-weft` suffix; the task reran as `orch-rules-files` and landed.
- `plan-gate-quality` and `fabric-net-and-config-warn`, paused at Run-Shed on 2026-10-08 a minute after their start, then re-scoped on the board into other entries.
- `webster-pack-end-of-prompt`, paused at Run-Shed on 2026-10-15, then replaced by `webster-pack-layout-switch`, which landed.

Their pairs were removed with `lyx fabric remove`, but no verb ends a paused or blocked run, and `lyx batten cleanup --apply` skips every paused, blocked, awaiting or failed run.
So the records stayed, showed as runs in the reed overview, and were removed by hand with `git rm` in the records repository.

## Idea

A batten verb, for example `lyx batten abandon <slug>`, ends a run that will not go on:
it refuses while the run's driver or run lock is live, tears down the pair if one is left, releases the run's board lock, removes the run's records directory and scratch, and commits and pushes the records.
It leaves the board entry itself alone, so a re-scope or rerun stays the caller's own step.
