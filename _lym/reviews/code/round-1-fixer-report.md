---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed move through `moveForkedWeftToAnchor` commits on its own, so the first records commit would carry no Anchor-Change trailer, and a `git mv` staged ahead of `CommitRecordsPaths` is dropped by its pathspec filter, since the old path is then in neither index nor disk. `moveForkedRecordsToAnchor` renames the tracked wired names on disk and stages the marker. The old and new paths join the one first records commit, which keeps the trailer."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The proposed way of parking the merge, MergeIn in the task worktree and then Merge in the receiving one, does not park anything: Merge aborts a conflicting merge, and the guard reads the receiving side's merge record. The rows park the receiving worktree through MergeIn(task.Branch), resolve and stage, leave an uncommitted records file, then run Merge and squash Merge and assert the refusal, an unchanged HEAD and the kept resolution."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "A kept branch is no failure and mails nothing, so the lease-rejected row asserts origin's branch left at the other process's tip and no notice, in place of RemoteKeptReason, which no public surface exposes. The view-ref row asserts the mailed way forward naming the view-ref step and `push origin --delete lym-view/<slug>`, and the ref left on origin."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: "The proposed first option, updating the merge record before HEAD moves, would leave a record naming a commit HEAD never reached when update-ref fails. The second option is taken: `undoRecordsCommit` moves HEAD back with update-ref when newPaired, loadMergeState or saveMergeState fails, and returns the error with committed false."
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed-differently
    reason: "The proposed git init in the test's prime puts the banned token gitkit.Git in a tier-0 file, which fails TestTierPurity_UntaggedTestsSpawnNothing in cmd/lym. The narrowed assertion is kept, and the commit carries a Loosened-Check trailer naming the measured cause: the layout read of a prime that is no git repository logs its own WARN."
---

# Fixer-report, code round 1

Every fix is its own commit on this worktree, in the review's order, and nothing is pushed.
The Go packages `fabricengine`, `battencli` and `runfsm` pass `lym gate test` at tier 0, and `fabricengine` passes at tier 1.

## What changed

- **F1**: `README.md` has a subsection "A hub of one repository" with both clone forms, the layout, and the three hazards `docs/getting-started.md` lists.
- **F2**: in a one-repository Add with `--anchor`, `moveForkedRecordsToAnchor` renames each tracked wired name from the fork's anchor to the new one on disk and stages the marker.
  The moved paths join the first records commit and its Anchor-Change trailer.
  `AddedAtAnotherAnchorCarriesTheTrailer` now tracks a records file in the parent and asserts it sits under the new anchor and is gone from the old one.
- **F3**: `PATTERN-run-fsm.md` says a `protected` prime push is not owed and that the refusal names the owed record's own way forward.
- **F4**: `TestNewCommitStatusSeam_LayoutWiring` pins the one-repository wait, the unread `MergeActive` and the tolerated protected push, and the two-repository skip.
- **F5**: two rows, plain and squash, over a parked merge on the receiving side.
- **F6**: `WarpSHATrailerNamingACommitNotReplayedStays`.
- **F7**: `TestReconcileOrigin_OneRepository` holds a diverged row naming the local code commit and a records-only replay row.
  The diverged row hung, and the stack dump showed why: `judgeAgainstOrigin` took the remote-refs lock of the code and the records repository, which are one repository in a one-repository hub, and that lock is not reentrant.
  A reconcile of a one-repository hub therefore never returned.
  `judgeAgainstOrigin` now takes the lock once when both directories are the same, and its doc comments say so.
- **F8**: `IntegrationOverALocalCodeCommitStopsNamingItAndMovesNothing`, with the origin move shared through `moveOriginBranchToSub`.
- **F9**: two rows after the existing one, over a shared `newOneRepositoryTeardown` helper.
- **F10**: `HeldIndexLockReleasedWithinTheBoundLetsTheCommitLand`.
  It waits 300 ms of real time that can only fail in the direction of a missing retry, since a missing wrapper returns at once.
- **F11**: `undoRecordsCommit` in `recordsduringmerge.go`.
- **F12**: `Judge` clears `PushOwedWayForward` with the other owed-push fields.
- **F13**: the any-`WARN` assertion is back, and the test's prime is a git repository so the layout read logs nothing.
  No check is loosened, so there is no `Loosened-Check` trailer.

## Gate repair

The gate flagged six comments that wrapped at a fixed column or ran to 120 characters.
Each is now broken at sentence boundaries, one sentence per line, in `commitstatus_test.go`, `addanchor.go` (two comments),
`originreconcile.go`, `recordsduringmerge.go` and `remoteteardown_integration_test.go`.
Only comment text changed.

The verify run then failed on `TestTierPurity_UntaggedTestsSpawnNothing`: the F13 `git init` used `gitkit.Git` in a tier-0 file.
That commit is reverted and the narrowed assertion stays, with a `Loosened-Check` trailer on the revert commit.
The `fabricengine` crash in the same log came from a background `pkill -QUIT` of mine that fired during the verify; it is no defect.
A third verify run failed `configreg` and `machinecli` on tmuxkit's guard, which found a stray `/tmp/.git` written during the run of one of the packages running beside it.
Neither package is touched by this round, and no test of this round writes outside its `t.TempDir`.
A rerun of `configreg`, `machinecli`, `fabricengine`, `battencli` and `runfsm` passes at tier 0, and of `configreg`, `machinecli` and `fabricengine` at tier 1.
No code changed for that finding, since it did not reproduce; a `/tmp/.git` that appeared and was removed again during my reruns points at another process sharing `/tmp`.
F13 in the list above and in "What changed" is read as this narrowed form, not the any-`WARN` assertion.

## Self-check

Four forks returned a report, each saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-consistency, LOW, `doc.go` still described two remote-refs lock holds in the reconcile: fixed.
- fix-consistency, LOW, the README said "Two hazards" where `docs/getting-started.md` lists three: fixed, the third hazard is in the README.
- fix-affected-set, LOW, the `addanchor.go` file header described only the move record route: fixed.
- fix-code-correctness, LOW, `undoRecordsCommit` has no test: deferred, see below.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

- The fix-code-correctness finding on `undoRecordsCommit` (F11) has no test.
  The failing steps run after `update-ref` and read the same state files that `recordsMergeKindAt` reads first, so no file or permission trick reaches them without failing earlier.
  A test needs a seam between `update-ref` and `saveMergeState`, which production code lacks, and adding one only for the test is not what F11 asks.
  The undo is covered by the fork's trace and by no test.

## Rulings

None reached this round.
