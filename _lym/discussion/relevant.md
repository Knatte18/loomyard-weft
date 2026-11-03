# Relevant code — batten-remote-order

- `internal/battencli/resumereconcile.go` `reconcileRun`, `reconcileResume`, `resumeRefusal` — the resume reconcile whose error is a bare pre-run refusal (bug 1)
- `internal/battencli/owner.go` — the pre-run owner checks and the Worktree-Teardown exception for a gone task worktree
- `internal/battencli/cancel.go` `removePairAndBranch`, `childReachedParent` — cancel's synchronous pair removal and task-branch discard, and its reached-parent test
- `internal/battencli/cancelcmd.go` — cancel wiring, the spawned prime records push behind `push_spawned`
- `internal/fabricengine/teardownlock.go` `DetachedPush`, `lockPairTeardown`, `TeardownLockPath`, `respawnPairPush` — the per-slug teardown lock D7 extends, and the push that exits when a removal holds it
- `internal/fabricengine/remove.go` `removePair`, `removeSibling` — the removal and its weft teardown, origin-only path included
- `internal/fabricengine/weftwiring.go` `removeWeftWorktree` — the unleased weft delete on origin
- `internal/fabricengine/cleanup.go` `deleteWeftBranchOnRemote` — the cleanup sweep's unleased weft delete
- `internal/fabricengine/archive.go` `archiveWeftTip`, `tagAndPushArchiveTip` — the archive tag push D4 reorders
- `internal/fabricengine/remoteleftover.go` — `Add`'s origin probes and liveness rule D1 relies on
- `internal/fabricengine/add.go` `Topology.Add` — makes or adopts a pair; D1 calls it, D7 locks it
- `internal/fabricengine/taskbranchdiscard.go` `Topology.DiscardTaskBranch` — archives and leased-deletes the task branch
- `internal/fabricengine/originreconcile.go` `judgeBranch` — where a branch origin lacks reads gone
- `internal/landingshed/finalize_owner.go` — Finalize's stop that names `lym fabric remove`
- `internal/dobbyengine/passes.go` — dobby's failing-pass count and pass planning for a `running` status
- `internal/shedengine/failedstop.go` `WriteFailedStop` — the status writer D2 uses
- `contracts/stencils/shed/shed-template-tackler.md` — the tackler stencil D5 extends
