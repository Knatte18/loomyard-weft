# run-pushes: Go pushes the task branch with the run records

All five batches finished with `status: done` and no deviations from the plan's declared file-ops.
The per-batch `record-batch` digests carried only informational scope-outside-plan warnings, for helpers and tests the cards needed.

## Batches

1. `01-lock-bounded-wait` (`fc22c26`): `internal/lock` gained `AcquireWriteLockWithin`, a write-lock acquisition that gives up when its wait expires, with a table-driven test.
2. `02-gitrepo-has-unpulled` (`7ae3f97`): `Repo.HasUnpulled` in `internal/gitrepo/push.go` counts commits on the upstream that HEAD lacks, and treats a missing upstream as true.
   It is registered in the gitrepo doc and the CLI-bound method list in `cmd/lyx`, and is covered by `TestHasUnpulled`.
3. `03-fabric-pair-anchored-push` (`1df0ac6`): `fabricengine` gained `PushPairAnchored`, `IsPushRejected`, `ErrPushLockBusy`, `StatusPushLockWait` and `LockWaitUnbounded`.
   `PushAnchored` now takes a lock wait, runs under the absorbing push lock and retries once on a rejection.
   The `battencli` and `loomcli` callers pass `StatusPushLockWait`.
4. `04-loom-pushes-both-sides` (`2e05011`): loom's status seam pushes through `PushPairAnchored`.
   The `commit-records` push failure splits into a rejected and a failed refusal, each with its own way forward and spec row.
   `docs/overview.md` and `PATTERN.md` (`PATTERN-push-both-sides`) record the change.
5. `05-fabric-push-sync-both-sides` (`d2855f3`): `lyx fabric push` pushes both sides through `PushPairAnchored` with an unbounded lock wait.
   `lyx fabric sync` spawns one detached push per side.
   The scenario tests cover both bares and the `WEFT_SKIP_PUSH` case.
