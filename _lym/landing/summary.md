# Report: why pushes of the weft repository's main race on one machine

Adds `docs/research/weft-main-push-race.md`, a read-only study of the 09:20 to 09:25 refusals on 2026-11-23.
No code changes.

## Findings

- Weft `main` is the board branch, and its one checkout is the Bolt worktree.
  Three locks guard its pushes, and the first push attempt of most paths takes none:
  `board.push.lock` for `lym board sync`, `lym-push.lock` for `pullFirst` and the rejection replay, and nothing for `Bolt.PushRecorded`.
- The owner claim of a `batten` run, status writes and config commits push through `PushRecorded` in parallel with the detached syncs.
  GitHub refuses the loser with `cannot lock ref`, and `PushWithRetry` reruns the same push four times, 1, 2 and 4 s apart.
- In the incident the claim commit `d359b72` was already an ancestor of origin's tip at the first refusal, so the claim had landed and the failure was spurious.
  `ErrPushRetriesExhausted` is classed as `transport`, which the owner write reports as `origin is unreachable`.
- No other machine wrote the board in the window: every origin tip named in a refusal is in the checkout's own history, and its reflog holds commits only.
- The `push lock busy` refusal is a separate lock, the prime's records `fabric.push.lock`, bounded at 30 s.

## Fixes the report names

1. `Bolt.PushRecorded` takes `board.push.lock` around the push and its retries, `Sync` commits first and then pushes through it, and `pullFirst` takes the same lock in the order write lock, then push lock.
2. A refused push counts as landed when a fetch shows no unpushed commit.
3. An exhausted refusal gets its own failure class instead of `transport`.
4. The catch-up push rides out a busy records push lock as the landing push does.

## Design decisions

The task named no report path, so the report is at `docs/research/weft-main-push-race.md`.
The report claims no cause for the `(failed)` refusals on `-weft` branches and marks that reading as not established.

## Merge danger

**Door:** two-way
**Blast radius:** docs
