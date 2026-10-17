# Preflight pushes the weft branch synchronously and races the board sync

`preflight-weft-push-sync` · Note · enhancement, fabric, shed

## Seen (orch, 2026-10-17)

The darn run `creel-c13-stacked-rewrite` spent 11.1 s in its Preflight row (trace `acc61c4a8e5651cb` in the pair's `.lyx/logs/`), almost all of it in a synchronous push of the weft branch to GitHub:

- 13:43:50.98: the row commits the weft (`fabric: mutation kind=commit_created`); 0.2 s earlier the same step spawned a detached board sync, which pushes to the same records repository.
- 13:43:58.91: the first push of `creel-c13-stacked-rewrite-weft` is refused after 8 s: `! [remote rejected] ... (failed)`.
- 13:44:01.89: the retry succeeds after 3 s, and the row is Done.

The operator saw about 22 s for the row; the trace accounts for 11 s of it.

## Idea

- Nothing in the run reads the remote weft branch until Publish, so Preflight could push it detached, as the board sync already is, and leave the row at local work only.
- The 8 s refusal looks like two pushes to the same records repository at once (the board sync and the weft push); if so, serialising them, or the detached push, would remove the retry.

## A pair begun on another machine

Operator question 2026-10-17: what if the branches already exist because the task was begun on another machine?
That case is settled before Preflight, at `fabricengine.Add` (Worktree-Create), per `internal/fabricengine/doc.go`: it probes both origins read-only, adopts a live pair (fast-forwards a local weft branch behind origin, or makes origin's weft and warp branches local tracking branches), refuses a true divergence with `ErrRemoteLeftover`, and never force-pushes or deletes a live pair's branches.
`Add` itself already pushes nothing and leaves the push to the pair's detached push (`SpawnPairPush`).
So a detached Preflight push changes nothing for that case; it only gives up seeing at once a push the remote refuses, which the next detached push logs as a Warn.
The bare `(failed)` refusal is a known server-side race: every plain fabric push retries it with backoff.

## Open

- Whether the remote refusal is a GitHub race between the two pushes or something else; check other runs' Preflight traces for the same warning.
