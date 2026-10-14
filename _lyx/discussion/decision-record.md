# Decision record — run-window-early

## Goal

`lyx batten run <slug> --window` puts the operator's terminal window up within about two seconds of the start, and every write to a new pair goes one way: a local commit followed by a detached push, as `lyx fabric sync` does.

## Scope

- Batten opens the terminal window, running `lyx reed attach` in the pair, as soon as the Worktree-Create row has the pair on disk, before the seed and the inner shed spawn.
- `fabricengine`'s pair creation no longer pushes the code and records branches synchronously at its last two steps; the push leaves the engine.
- Batten's Seed-Child row commits the child seed locally and pushes it detached instead of waiting on the push.
- `lyx fabric add` and batten's Worktree-Create row push the new pair detached.
- Out of scope: the pre-flight `ls-remote` probes of pair creation, the board claim, and the hub-config reconcile of the new pair.

## Decisions

### Scope: early window plus async pushes for pair creation and child seed (by: parent, 2026-10-14)

Decision: the task builds three changes: (1) the early terminal window; (2) the child seed committed locally and pushed detached like `lyx fabric sync`; (3) pair creation's code and records branch pushes detached, a failed push retried by the next sync, never rolling the pair back or leaving it broken.
It supersedes the board entry's "Scope" and "Out of scope" sections.
Rationale: the operator wants every pair write to go one way, local commit plus detached push as fabric sync does, and the window up within about two seconds; the run reads only local files, so nothing waits on GitHub.

### Pair creation stops pushing; its callers push

Decision: `Topology.Add` keeps its pre-flight probes and the leased deletion of an archived leftover records branch, but no longer pushes either branch.
The production callers, `lyx fabric add` and batten's Worktree-Create row, push the new pair through one new engine entry that spawns the existing detached push child for the pair's code and records worktrees.
`AddResult.Pushed` and the add envelope's `pushed` key are replaced by the caller's report of whether a push was spawned.
Rationale: a detached spawn from inside the engine would fork the test binary for every in-process caller; leaving the push to the CLI layers keeps the engine local-only and lets test fixtures push synchronously.
Bound: with no push inside `Add`, its rollback never sees a pushed branch, so the rollback step that deleted the code branch from origin is removed, along with the push-retry seam only `Add` used.
A failed or never-run detached push leaves both branches committed locally and unpushed; the next detached push from that pair (every fabric commit and sync spawns one) carries them, and `push.autoSetupRemote` creates the upstream on that first push.

### Seed push goes detached

Decision: the Seed-Child row's `PushSeed` seam spawns the same detached pair push instead of the synchronous records push; `CommitSeed` is unchanged.
Rationale: the seed is read locally by the child bootstrap, so nothing waits on the remote.
Bound: a spawn failure stays a warning that never changes the row's verdict, as a push failure is today.

### Window opens from the Worktree-Create row

Decision: batten's Worktree-Create seam opens the terminal window once the pair is complete and its config reconciled, through the same once-marker the Run-Shed row uses; the Run-Shed row keeps its open after a spawn, which the marker turns into a no-op unless the window was never opened in this run.
Rationale: one marker keeps the window once per run, and a run resumed past Worktree-Create with `--window` still gets its window at Run-Shed.
`lyx reed attach` boots the pair's reed session when none exists, and reed serialises boots under its op lock, so the later `lyx loom start --no-attach` joins the same session.

### Test callers push synchronously

Decision: `hubforge.AddPairWith` pushes the new pair synchronously after `Add` unless its options skip pushing; `fabriccli` and `battencli` route their detached pair push through a package seam their tests replace with a synchronous push.
Rationale: in-process tests must never fork the test binary, and their assertions about remote branches stay deterministic.

## Constraints

- `PATTERN-fabric-git`: git on the pair runs only through `internal/fabricengine`; the new push entry lives there.
- `PATTERN-fabric-vocabulary`: callers outside the fabric owner set reach the pair push from a `*lyxcwd.Location` alone, never naming the records worktree.
- `PATTERN-spawn-observability`: the detached push child is already logged at spawn by `SpawnDetachedPush`.
- `PATTERN-test-economy`: rollback and push-retry tests for the removed `Add` push paths are deleted, not kept against dead code.
- Docs in the same commits: `internal/fabricengine/doc.go`, `internal/battenshed/doc.go`, `internal/battencli` help text, and the CLAUDE.md "Watching runs" sentence.

## Auto-mode assumptions

- The pre-flight probes of `Add` (`ls-remote` of both origins) stay synchronous; they are network reads, not pushes, and decide refusals before the first mutation.
- The archived-leftover deletion of the records branch stays synchronous, since a detached push onto the leftover would be rejected; it runs only when a leftover exists.
- A reed config override from the task's run settings, written by Seed-Child after the window opens, can miss the reed session the window boots; run settings target per-strand keys read at spawn, so this is accepted.

## Open risks

- A pair whose first detached push fails stays local-only until its next commit or sync; another machine cannot adopt it meanwhile.
- A run resumed from a status file written by an older binary finds its once-marker under the same name, so no duplicate window opens.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - batten-create-and-seed-push-spawns-unasserted (class scope, severity MEDIUM, open in rounds 1)
  - fabric-add-bypasses-spawnpairpush (class consistency, severity MEDIUM, open in rounds 1)
  - fabric-add-envelope-and-spawnpairpush-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- `--window`: the terminal open happens in the Worktree-Create row after a successful create or an already-complete pair, and at most once per run with the Run-Shed row's open.
- A failed terminal open is warned about and never changes the Worktree-Create verdict.
- `Topology.Add` pushes nothing: both new branches exist locally and are absent from origin until a push runs.
- `lyx fabric add` spawns one detached push of the pair and reports it in its envelope; under `FABRIC_SKIP_PUSH` or `FABRIC_SKIP_GIT` it spawns none.
- Seed-Child commits the seed locally and its push seam spawns the detached pair push; a push-seam error is warned, the row still Done.
- `hubforge.AddPairWith` with zero options leaves the records branch on the bare remote.
- `lyx gate test` of the touched packages, untagged and `--tags integration`, is green.

## Notes for the plan writer

- `internal/fabricengine/add.go` steps 11 and 12, `rollbackAdd` step 5's remote deletion, `addWarpBranch`, `pushretry.go`, `weftwiring.go`'s `pushWeftBranch`, `Topology.push`.
- `internal/fabricengine/spawn.go` holds `SpawnDetachedPush`; `commit.go` holds the `spawnDetachedPushFn` test seam.
- `internal/battencli/wire.go`: `CreateWorktree`, `PushSeed`, `openTerminal`; `internal/battenshed/innerrun.go`: `openTerminalOnce`.
