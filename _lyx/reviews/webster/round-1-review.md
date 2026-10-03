---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: internal/battencli/wire.go:394
    summary: Batten's Shutdown fails with ErrPairNotFound on re-entry after a completed removal, so the row halts at teardown for good and never reaches the done post-condition finishRemoval grants
  - id: F2
    severity: LOW
    location: internal/fabricengine/remove.go:586
    summary: commitPendingRecords resolves the Warp-SHA trailer before its empty-pathspec return, so a sibling location that is not a git checkout fails Remove even when there is nothing to commit
  - id: F3
    severity: LOW
    location: internal/fabricengine/remove.go:87
    summary: RemoteSkippedReason's doc comment still says its only cause is a weft repo with no origin, but card 2's finish path now sets it for an origin-only sibling branch kept without remote
  - id: F4
    severity: LOW
    location: internal/battencli/wire.go:161
    summary: New and changed Go comments across the diff are fixed-column wrapped mid-sentence, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F5
    severity: NIT
    location: internal/pairteardown/teardown.go:58
    summary: SessionResult.Ended says a session was ended, but the present-worktree arm reports true after Engine.Down even when the pair had no session, so session_ended overstates
---

Review range: `_lyx/shed/pair-lifecycle/status.json` gives `product.parent = main`, and `_lyx/loom/rework/` does not exist, so the subject is `git diff $(git merge-base main HEAD)..HEAD` (merge base `5805c58`, cards 1-13 plus `d4d9f6b96`).
The plan's verify block passed: `go build ./...`, `go vet ./...`, `go test ./...`, the plan's integration list, `go vet -tags smoke` over the three packages, and `go vet -tags integration ./...`.

### [BLOCKING] Batten's Shutdown halts a re-entered teardown on a pair that is already gone

**Location:** `internal/battencli/wire.go:394` (the `Shutdown` closure), with `internal/fabricengine/remove.go:517` (`RemoveRefusal` calls `refuseWhenNothingRemains`).

**Issue:** Card 11 makes `ErrPairNotFound` the Worktree-Teardown row's done post-condition, and the rewritten comment above `Teardown:` claims a process killed right after `Remove` succeeded re-enters the row, "and RemovePair reports that as ErrPairNotFound, which is done".
That re-entry never reaches `RemovePair`.
`battenshed`'s producer calls `Shutdown` first and never calls `Remove` when it fails (`internal/battenshed/teardown.go:109`, pinned by `TestWorktreeTeardown_ShutdownFailureNeverCallsRemove`).
`Shutdown` calls `EndSession`, which finds the task worktree gone, skips the quiet wait and runs the refusal probe.
`RemoveRefusal` now includes `refuseWhenNothingRemains`, so with nothing of the pair left it returns an error wrapping `ErrPairNotFound`.
`teardownRefusal` passes that error through unchanged (it is neither `ErrArchiveFailed` nor `ErrPairSiblingDirty`), `Shutdown` returns it, and the row is Stuck at session shutdown on every resume.
The old code skipped shutdown for a gone task worktree, which was exactly what made the row idempotent; `TestWire_TeardownIsIdempotentAgainstAnAlreadyRemovedWorktree` pinned that through `Shutdown` and was replaced by a pure `finishRemoval` unit test that never calls the closure, so the regression is uncovered.
`manifest/designs/pair-teardown.md` and `internal/battenshed/doc.go` repeat the same false claim.

**Fix:** In `Shutdown`, treat `errors.Is(err, fabricengine.ErrPairNotFound)` from `EndSession` as nothing left to shut down: log it and return `"", nil`, so `Remove` runs and `finishRemoval` reports done.
Add an integration test in `internal/battencli/lifecycle_integration_test.go` that tears a created pair down through both closures and then runs both again, asserting the second pass returns nil from each.

### [LOW] The pending-record commit resolves its trailer before checking there is anything to commit

**Location:** `internal/fabricengine/remove.go:586`.

**Issue:** `commitPendingRecords` builds `present`, then calls `recordTrailerSHA` unconditionally, and only `CommitWeftPaths` notices `present` is empty.
With the task worktree and the local task branch both gone, `recordTrailerSHA` runs `git log HEAD` in the sibling location.
When that location is not a git checkout (the state `TestRemove_StatusFailureNamesPathAndCommandOnce` builds), `Remove --force` fails with "read the sibling branch's log" although no record path exists to commit.
It is also a wasted git call on every removal with nothing pending.

**Fix:** Return `false, nil` when `present` is empty, before resolving the trailer.

### [LOW] RemoteSkippedReason's doc names only one cause, and card 2 added a second

**Location:** `internal/fabricengine/remove.go:87`.

**Issue:** The field's comment says it carries a reason "today only a weft repo with no origin remote configured".
`removeSibling`'s origin-only arm now sets it to "the sibling branch ... exists only on origin and was kept; pass --remote to delete it", which card 2 asks for ("the result says so").
A reader of the field, or of batten's teardown log line that prints it, is told the wrong cause.

**Fix:** Reword the comment to state both causes without a "today only" tally.

### [LOW] New and changed comments are fixed-column wrapped

**Location:** `internal/battencli/wire.go:161`, plus the sites listed below.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line for new and changed Go comments, with no column limit.
These new or changed comments break mid-sentence at a fixed column instead:
- `internal/battencli/wire.go`: the `taskWorktreePresent` doc paragraph and the comment above `Teardown:`.
- `internal/loomcli/bootstrap.go`: the `driverStrandRetiring` comment.
- `internal/loomcli/start_driver_test.go`: the doc comments of the two retiring-driver tests.
- `internal/fabriccli/openprheads.go`: the file header and `listOpenPRHeads`' doc.
- `internal/fabriccli/fabric.go`: `sweepRemoteTaskBranches`' doc.
- `internal/fabriccli/removefields_test.go`: the file header.
- `internal/fabricengine/doc.go`: the rewritten `PairComplete` paragraph.
- `internal/fabricengine/fabric.go`: the changed line of `PairComplete`'s doc.
- `internal/fabricengine/remove_refusal_remedy_integration_test.go`: the rewritten doc and inline comment of `TestRemove_StatusFailureNamesPathAndCommandOnce`.
- `internal/reedcli/cli.go`: the `strands`/`spawnRemove` seam comment.
- `internal/reedcli/remove.go`: the detached-remover no-op comment.
- `internal/reedengine/overlay.go`: `reapSessionVia`'s and `EndSessionByName`'s doc comments.
- `internal/pairteardown/testmain_test.go`: the `TestMain` doc.

**Fix:** Reflow each to one sentence per line, breaking further only at an independent-clause boundary.

### [NIT] session_ended reports true for a pair that had no session

**Location:** `internal/pairteardown/teardown.go:58` (`SessionResult.Ended`) and `endReedSession`'s present arm.

**Issue:** `Engine.Down` is idempotent and reports nothing about whether a session existed, so `endReedSession` returns `true` whenever `Down` succeeds.
The field's comment, "true when a session was ended", and the `session_ended` envelope key both read as "a session existed and was ended", which only the gone-worktree arm guarantees.

**Fix:** Reword the comment to what each arm reports: `Down` ran on a present task worktree, or a session was found and reaped by name on a gone one.

## Settled focus items that are not defects

- `endReedSession`'s present arm on a pair with no reed session: `Engine.Down` ignores the failed `kill-session` and treats an empty `list-sessions` as no server, so it does not error; the pre-existing `fabric remove` CLI integration tests run it on sessionless pairs and pass.
- `probeQuiet`/`waitQuiet`: the absent-directory shortcut, the park-marker short-circuit, the `ErrNoSession` tolerance and the attempt cap (`QuietWait/interval`, with `QuietWait=0` probing exactly once) match card 9.
- `RemoveRefusal` and `Remove` share one probe, the record commit stages only the scoped pathspec through `CommitWeftPaths` and runs before the archive with or without `force`, and the leased task-branch deletion and `stray_path` behave as cards 2 and 3 state.
- `CleanupRemoteWarp` classifies every keep reason card 4 names and refuses all deletions on a nil `openPRHeads`; the CLI skips the sweep entirely when GitHub is unreachable, which is what card 5 asks for, and each deletion is leased to the enumerated tip.
- `lyx loom start` removes a live retiring driver strand before computing `mustSpawn`, the detached remover answers ok on `ErrUnknownStrand` only on the `--wait-pid` path, and `EndSessionByName` kills the server only when `list-sessions` comes back empty.
- Card 12's Delete group: `git grep` finds no reference to `RemovePairBranch`, `PairBranchResult` or `PairSiblingRemnant` outside the tripwire and its CONSTRAINTS.md line, and `go vet -tags integration ./...` passes.
  The tripwire flags a four-argument `Remove` call in any non-test file importing `internal/fabricengine` outside the two owners, which is what the invariant's first bullet claims, and its self-test is not vacuous.
- `internal/pairteardown` names no side in its production files.
