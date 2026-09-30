# Stop batten run records leaking into new pairs; wait on prime lock

A pair created by `lyx batten run` or `lyx fabric add` forked its weft branch from the parent's, so it inherited every committed `_lyx/shed/<slug>/` run directory.
The child's own loom run then found a foreign batten seed for its slug and blocked at `Seed-Child` with "already seeded with a disagreeing seed".
Every other in-flight run's records also cluttered each new pair.
Separately, a second `Worktree-Create` or `Worktree-Teardown` halted `blocked` whenever another run held the prime lock.

## What changes

- `fabricengine.Add` forks the new weft branch without the parent's run-records root.
  The records never reach the new worktree's disk, and the pair's first weft commit (the origin record, same message as before) records their deletion.
  This applies to both callers of `Add` and to any parent branch, `main` or a task branch.
- `shedrun` exports `RunsRootRel()`, the anchor-relative run-records root, so `fabricengine` does not name the `shed` segment itself.
- `battenshed`: `Worktree-Create` and `Worktree-Teardown` wait for a contended prime lock, polling every 2 s for up to 10 min, through one shared helper (`acquirePrimeLock`).
  After the bound they return `Stuck` with a reason naming the lock path and the wait.
  A cancelled context during the wait returns the producer's cancel error, and an `Acquire` error stays a hard error.
  The wait logs one info line when it starts.
- `docs/overview.md`, `CONSTRAINTS.md` (Shed Run-Directory Invariant bullet) and the `fabricengine`, `shedrun` and `battenshed` package docs describe the new behaviour.

## Design decisions

- **Drop at the source, no batten namespace.**
  Batten's records stay at `_lyx/shed/<slug>/` in prime.
  Once pairs stop inheriting `_lyx/shed/`, the prime-side record and the child's own record live in different worktrees and cannot collide.
  A separate namespace would fix only the collision, not the clutter.
- **No-checkout fork, not a delete.**
  The fork is `worktree add --no-checkout`, then the index is written out with the run-records root removed and restored from `HEAD` afterwards.
  The files never exist on disk, which matters because `shedrun` reads seeds from disk.
  Nothing is deleted, so the Fabric Destruction Chokepoint Invariant does not apply.
- **Adopt path untouched.**
  When `Add` adopts a surviving weft branch, it drops nothing; that branch's records are the pair's own.
- **One commit, conditional.**
  The deletions are staged only when the fork point tracks something under the root, since a pathspec matching nothing is a hard git error and a fresh hub must still add pairs.
  Rollback covers the drop.
- **`PrimeLock` contract unchanged.**
  `Acquire` stays non-blocking; the retry loop lives in the producers.
  Poll interval and bound are package constants, and the wait uses an injected sleep seam so tests run without real time.

## Not covered

- Existing pairs keep their inherited copies; they are inert and never merge back.
- Other per-run directories a task-branch pair inherits (`_lyx/loom/`, `_lyx/discussion/`, `_lyx/plan/`, `_lyx/reviews/`) are untouched.
- A lock holder exceeding 10 min (for example a very slow archive push during teardown) still halts the waiter `blocked`, as today; resuming retries.

## Checks

`go build ./...`, `go test ./...` and the integration-tagged tests for `shedrun`, `fabricengine`, `battenshed` and `battencli` passed.
New integration tests cover the drop (tracked and untracked root, adopt, non-`main` parent, rollback) and the reported `Seed-Child` collision.
Nothing was run by hand; to see it end to end, start several `lyx batten run` in a row and confirm none blocks at `Seed-Child` or on the prime lock.
