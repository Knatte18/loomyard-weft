# Batten run records in the prime leak into every new pair

## Goal

A task pair created by `lyx batten run` (or by hand with `lyx fabric add`) starts with no shed run records inherited from its parent's weft branch,
so the child's own loom run never finds a foreign seed at its own `_lyx/shed/<slug>/` and blocks at `Seed-Child`.
Parallel batten runs whose `Worktree-Create` (or `Worktree-Teardown`) rows overlap wait their turn for the prime lock instead of halting `blocked` on contention.

## Scope

In:

- `fabricengine`'s `Add`: a freshly forked weft branch drops its parent's shed run-records directory (the whole `_lyx/shed/` tree under the pair's anchor) in the pair's first weft commit, alongside the origin record.
  Applies to both callers of `Add` — `lyx fabric add` and batten's `Worktree-Create` — and to any parent branch, `main` or a task branch.
- `shedrun`: a new exported anchor-relative constructor naming the run-records root (`_lyx/shed`), so `fabricengine` never names the `shed` segment itself.
- `battenshed`: `Worktree-Create` and `Worktree-Teardown` wait for the prime lock, bounded, instead of returning `Stuck` on the first contended try.
  This absorbs the `batten-prime-lock-wait` item named in the task body.
- Docs and `CONSTRAINTS.md` updates listed under Acceptance criteria.

Out:

- Moving batten's own records out of `_lyx/shed/<slug>/` in prime.
  Batten keeps its current location; see Decisions.
- Cleaning inherited `_lyx/shed/` copies out of pairs that already exist.
  They are inert and never merge back (the weft is never a merge participant).
- Other per-run directories a pair forked from a task branch inherits (`_lyx/loom/`, `_lyx/discussion/`, `_lyx/plan/`, `_lyx/reviews/`).
  Nothing reported a problem with them.
- `.lyx/` ephemeral state: it is untracked and never inherited.

## Decisions

1. **Stop the inheritance at its source, in `fabricengine.Add`.**
   A new weft branch forks from the parent's weft branch; creating it drops the run-records root from the new branch, so the parent's records never reach the new weft worktree's disk, and the removal is committed before the pair is pushed.
   Rationale: it fixes both symptoms (the same-slug seed collision and the clutter of every other in-flight run), for every way a pair is created, with no on-disk migration of in-flight batten runs.
2. **The adopt path is untouched.**
   When `Add` adopts an already-existing weft branch (`weftBranchAlreadyExists`), it drops nothing.
   Rationale: `Remove` and `Add`'s rollback both delete the weft branch, so one that survives them is the pair's own; its `_lyx/shed/<slug>/` holds that pair's own run records.
3. **No batten-specific namespace.**
   Batten's prime-side run directory stays `_lyx/shed/<slug>/`, addressed by the same flat run-id vocabulary `lyx shed seed|step|status` and ly-drive use.
   Rationale: once pairs stop inheriting `_lyx/shed/`, the prime-side batten record and the child's own record live in different worktrees and cannot collide.
4. **`fabricengine` learns the path from `shedrun`, not from its caller.**
   `shedrun` exports an anchor-relative constructor for the run-records root; `Add` calls it directly.
   Rationale: the Shed Run-Directory Invariant keeps `shedrun` the sole declarer of the `shed` segment, and a path `Add` derives itself cannot be forgotten by a future caller the way a caller-told option could.
5. **One weft commit, not two.**
   The drop lands in the same commit as the origin record, under today's message `fabric: record parent branch for <slug>`, and only when the fork point tracks anything under the run-records root.
   Rationale: a pathspec matching nothing is a hard git error, and a fresh hub with no run records must still add pairs.
6. **Rollback covers the drop.**
   A failure after the drop runs `Add`'s existing rollback, which deletes the new weft branch and worktree; the parent's weft branch is never touched.
7. **Prime lock: bounded, cancellable wait inside the producer.**
   Both `Worktree-Create` and `Worktree-Teardown` retry the non-blocking `PrimeLock.Acquire` at a fixed poll interval until acquired, the context is cancelled, or a bound elapses.
   Poll interval 2 s, bound 10 min, both package constants in `battenshed`.
   On the bound the producer returns `Stuck` with the existing contention reason, extended to name how long it waited.
   A cancelled context during the wait returns the producer's existing cancel error.
   The wait uses an injected sleep seam so tests run without real time.
   The producer logs one info line when it starts waiting, naming the lock path and slug.
   Rationale: a lock held for a create or teardown is held for seconds to a couple of minutes, so a bounded wait turns an operator-visible halt into a short delay while still surfacing a lock that never frees.
8. **`PrimeLock`'s contract is unchanged.**
   `Acquire` stays non-blocking; the retry loop lives in the producers (one shared helper), not in `battencli`'s closure.
   Rationale: the producers own the context and the stuck verdict; the closure owns only the lock primitive.

## Constraints

- **Shed Run-Directory Invariant:** `fabricengine` names neither the `shed` segment nor `seed.json`; it uses the new `shedrun` constructor.
- **No run records on disk:** the parent's run records must never sit in the new weft worktree untracked.
  `shedrun` reads seeds and lists runs from disk, so an untracked copy still collides with the child's own seed.
  Today's create path checks the parent's whole tree out (`git worktree add -b`), so an index-only removal after that checkout leaves the files on disk and does not fix the bug.
- **Fabric Destruction Chokepoint Invariant:** the drop uses one of two mechanisms, and the plan states which.
  Either the fork keeps the subtree off disk (a no-checkout fork whose index drops the root before the tree is written out), which deletes nothing and is outside the chokepoint;
  or it deletes the checked-out files, which is a destructive primitive and must live in `internal/fabricengine/destroy.go` with its containment/ownership/dirtiness/force checks.
- **Fabric Git Invariant:** the drop commit goes through `fabricengine`'s scoped-pathspec commit path (`ScopedPathspec` / `CommitWeftPaths`), positive-only file list, never a repo-wide stage.
- **Mutation Record Invariant:** the drop records its mutation(s) on `Add`'s `*Mutations` recorder only after they observably changed state.
- **Batten Bookend Invariant:** unchanged; the lock wait runs in prime like the rest of the batten Shed.
- **Transient-mark rules:** contention exhausted at the bound is a `Stuck` verdict, not a transient-marked hard error.
- **Documentation Lifecycle / semantic line breaks** for every doc touched.

## Auto-mode assumptions

- No operator was available; every pick below and in the support log's Question ledger is an auto-pick.
- The brief's suggested batten namespace is superseded by Decision 1, which fixes the collision and the clutter both; the namespace would fix only the collision.
- `batten-prime-lock-wait` is not a board task of its own (`lyx board get` returns `null` for it); its scope is taken from the brief's one-line description, "parallel creates block on the prime lock", read as: the second create halts `blocked` on contention.
- The lock bound (10 min) and poll interval (2 s) are constants, not config keys; nothing has asked to tune them.
- Pairs created before this fix keep their inherited copies; the hub already cleared the blocked ones by hand.

## Open risks

- A git-level race between a batten status commit on prime's weft and a concurrent `Add` forking from the same local `main-weft` ref.
  Not introduced by this task; the prime lock does not cover prime's status commits.
  The drop makes the fork point's run records irrelevant either way.
- A lock holder that legitimately exceeds 10 min (a very slow archive push during teardown) makes the waiter halt `blocked` exactly as today; resuming retries.
- Nested pairs still inherit the parent task's other per-run directories (out of scope above).
- A crash between the fork and the origin commit leaves an unmodified fork of the parent's weft branch, run records included, with no rollback.
  Batten's retry stops at the warp branch's "already exists" refusal, but an operator who deletes the warp branch and re-adds reaches the adopt path, which keeps those records.
  Remedy: delete the leftover weft branch too before re-adding.

## Acceptance criteria

- `fabricengine` integration: `Add` on a hub whose parent weft tracks `_lyx/shed/<a>/` and `_lyx/shed/<b>/` produces a pair whose weft branch is exactly one commit ahead of the fork point, tracks nothing under `_lyx/shed/`, has no such files on disk, carries the origin record, and is pushed; the parent's weft branch still tracks both directories.
- `fabricengine` integration: `Add` on a hub whose parent weft tracks nothing under `_lyx/shed/` succeeds with the same single origin-record commit as today.
- `fabricengine` integration: `Add` adopting an existing weft branch leaves that branch's `_lyx/shed/` content untouched.
- `fabricengine` integration: `Add` from a task pair (non-`main` parent) drops the parent task's own `_lyx/shed/<parent>/`.
- `fabricengine`: a failure injected after the drop rolls the pair back fully; the parent weft branch is unchanged.
- `battencli` integration: a batten run seeded and committed in prime for slug `B`, plus a committed batten record for another slug `C`, then `Worktree-Create` and `Seed-Child` for `B`: `Seed-Child` is `Done`, the child's seed is recipe `loom`, and the child has no `_lyx/shed/C/`.
  This reproduces the reported failure (disagreeing batten seed at Seed-Child) on today's code.
- `battencli` integration: `ReadStatus` on the fresh child reports absent even though prime tracks a batten status for the same slug.
- `battenshed` unit: `Worktree-Create` with the lock contended for N polls then free acquires it and returns `Done`; contended past the bound returns `Stuck` with a reason naming the lock path and the wait; context cancelled mid-wait returns the cancel error; the release runs on every exit after acquisition.
- `battenshed` unit: the same three wait cases for `Worktree-Teardown`.
- `battenshed` unit: an `Acquire` error during the wait is a returned hard error, as today.
- `docs/overview.md`: the batten paragraph's "every child's weft branch … frozen copies … inert" sentence is replaced by the new behaviour (a fresh pair starts without parent run records), and the prime-lock wait is stated.
- `CONSTRAINTS.md`: the Shed Run-Directory Invariant (or the Fabric Git Invariant) gains a bullet: a freshly forked weft branch never carries its parent's shed run records; `fabricengine.Add` drops the root `shedrun` names, and the adopt path drops nothing.
- Package docs updated: `internal/fabricengine` (Add's drop step), `internal/shedrun` (the new constructor and its one consumer), `internal/battenshed` (the lock wait).
- `go test ./...` and the integration suite pass.

## Notes for the plan writer

- Reported failure text: `task worktree already seeded with a disagreeing seed: run self is already seeded with {Recipe:batten ...}; refusing to overwrite with {Recipe:loom ...}`.
- This worktree itself shows the leak: its `_lyx/shed/` holds other slugs' batten directories inherited from `main-weft`.
- Places to start: `internal/fabricengine/add.go` (fork at step 8, origin commit at step 10c, rollback), `internal/fabricengine/commitweftpaths.go`, `internal/fabricengine/destroy.go`, `internal/fabricengine/pathtracked.go`, `internal/shedrun/paths.go`, `internal/battenshed/create.go`, `internal/battenshed/teardown.go`, `internal/battenshed/deps.go` (`PrimeLock`), `internal/battencli/wire.go` (the `Acquire` closure), `internal/battencli/lifecycle_integration_test.go`, `docs/overview.md` batten bullet.
- `internal/battenshed/innerrun.go` already has a sleep seam precedent (`InnerRunDeps`) for the lock-wait test seam.
