---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/webstercli/reset.go:200
    summary: standalone resets to pre-fix, report-head, last-batch-head and batch-start carry no uncommitted envelope key, though card 15 says every standalone reset lists the paths it left
  - id: F2
    severity: LOW
    class: consistency
    location: contracts/specs/refusal-spec.md:151
    summary: the recover-batch uncommitted-paths row names only a failed listing as its trigger, but the same refusal also fires when the run-writes audit or link resolution fails, and no test reaches that cause
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD`.
No `_lyx/loom/rework/` round exists, so rubric step 4 applies; `product.parent` is `main`.
`go build ./...`, `go vet ./...`, `go vet -tags integration ./...` and `go test ./...` pass on the branch head.
`go test -tags integration` passes for the websterengine, webstercli, batcher, configengine, configcli, loomshed, gitrepo, cmd and tools packages.

### [LOW] Standalone resets to the targets other than start list no uncommitted paths

**Location:** `internal/webstercli/reset.go:200`
**Issue:** Card 15 says "Every standalone reset lists the uncommitted paths it left (`UncommittedPaths` after the move) in the `uncommitted` envelope key: standalone keeps every change git carries across, the run's own included."
Card 14 words the pair half as "every reset to start", so the two cards differ on purpose.
`resetCmd` fills `uncommitted` (and `moved`, `warnings`) only inside `if target == websterengine.ResetToStart`.
A standalone `--to pre-fix`, `report-head`, `last-batch-head` or `batch-start` moves HEAD with git's keep form and leaves the run's own tracked changes in the tree, yet its envelope names none of them.
The long help and the package doc state the key for `--to start` only, so they agree with the code and not with the card.
**Fix:** Either list `UncommittedPaths` after a standalone move for every target, with a warning on a failed listing as the start path does, and pin it with a row in `TestResetCmd`'s standalone step; or narrow card 15's sentence in the plan's intent if only start was meant.

### [LOW] The recover-batch uncommitted-paths refusal has a second cause the spec row and the tests omit

**Location:** `contracts/specs/refusal-spec.md:151`
**Issue:** `uncommittedPathsBlock` returns an error from `UncommittedPaths`, from `loadRunWrites` (an audit of a recorded Master session) and from `writtenWorktreePaths` (link resolution).
`recoverSpawn` wraps all three into the one transient refusal naming `recover-batch NN`.
The spec row's trigger reads "cannot list the worktree's uncommitted paths for the recovery prompt", so it names only the first.
`TestRecoverSpawnOrAttach_StartFailures` reaches the refusal through a failing git status only (`fakeGit.dirtyPathsErr`); no row makes the run-writes audit fail.
**Fix:** Widen the row's trigger to the audit of the run's sessions, and add a row to `TestRecoverSpawnOrAttach_StartFailures` with a dirty path, a recorded `MasterSessionID` and an `AuditForksFn` that errors, reusing the test's existing assertions.

## Focus verdicts

1. `recoverSpawn`'s `uncommittedPathsBlock` refusal: a test makes the dirty-path read fail and asserts the `recover-batch 1` re-run suffix, `transient`, `PrepareCalls == 0`, no batch state, and a spawn on the retry.
   The run-writes half of the question is not reached; that is F2.
   The refusal comes after `archiveStaleReport` and `removeStrandIfLive`, so a retry meets an already archived report and an already removed strand; both steps are idempotent, so this is not a defect.
2. `Run` and `overviewFrameHash`: `Run` hashes the bytes its own parse accepted through `overviewFrameHashOf` and panics on the unreachable error, per the programmer-error rule.
   The remaining refusals on the path are the index-only read in `Rebaseline` and the restamp in `restampBaseline`, which `Run` also reaches through `restampAndSaveFingerprint`; `TestRebaseline_OverviewFrameUnreadable` reaches both.
   The refusal spec lists row "overview frame unreadable" for `rebaseline` and for a verb restamping the plan baseline, and names no `run`-only refusal that is gone.
   Not a defect.
3. `Rebaseline`'s index-only read refusal: the wording `way forward: transient, re-run \`lyx webster rebaseline\` with the same \`--card\` flags` is identical in `rebaseline.go`, the spec row and the test's expected suffix.
   Not a defect.
4. `MigrateConfig` and `TestMigrateConfig`: with `master_base` first and `startup_context` last, a last-drop index would put `orientation` after `custom`, and the row's expected output has it first; with `startup_context` first and `master_base` last, a first-drop index would run past the slice, and the other row catches that.
   Both rows distinguish the lowest drop index from either end's index.
   Not a defect.
