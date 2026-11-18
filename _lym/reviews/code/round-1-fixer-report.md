---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- F1: `stopStepRunsWhere` (`internal/shuttleengine/stepstop.go`) re-reads each candidate's record with `loadRunState` before `stopRecorded`, and skips it, unlisted, when it no longer reads running or `selects` no longer accepts it.
  `stop_test.go` gained a `statusHookReed` wrapper and an `attached` row in `TestRunner_StopOrphanedStepRuns_StopsOnlyRunsWhoseWaiterIsGone`.
  The hook gives that row the live pid `os.Getpid()` after the scan, and the row stays running with no `RemoveStrand` call.
  The `stopStepRunsWhere` doc names the skip.
- F2: the empty-root check asserts `fake.CallLog` holds no `Status` entry.
- F3: `orphanedStepsNote` is a third row of the hooks table in `looparm_test.go`, behind a stop adapter.
- F4: `internal/shedverbs/doc.go` says "the next loop's first step".
- F5: `stepStopNote` says `could not stop the runs: `, and the failing-stop row's want string follows.
- F6: the `StopOrphanedStepRuns` doc says "or the record names no waiting process".
- F7: the `gone` row seeds `deadPID`, a pid above the kernel's `pid_max` that no process holds, so `runIsLive` reaches `isAlive`.
  A first version ran and reaped a child process; the gate's tier-purity and hermetic-git checks refuse `exec.Command` in a tier-0 test, so it was replaced.
  This also removes the pid-recycling risk the fix-proposal lens raised.

`lym gate test` over the three touched packages passed at tier 0.
The later doc-line addition to `stopStepRunsWhere` is comment-only.
A gate repair then re-broke three comment lines at sentence boundaries, in `shedverbs/doc.go` and `stepstop.go`:
the F4 paragraph, the `StopOrphanedStepRuns` doc line, and the F1 re-check comment.
The `StopOrphanedStepRuns` line now reads "gone or never recorded", which keeps the F6 meaning.
Fixes are committed in two commits, grouped by package, not one per finding.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings; one LOW observation, the `_lym/landing/summary.md` sentence, covered under deferred below.
- fix-proposal, LOW, F7: a reaped pid could be recycled before the check, which is unlikely on Linux; kept as proposed, since the `isAlive` stub needs a non-parallel test.
- fix-proposal, LOW, F1: a candidate that changed since the scan is now skipped silently for `StopStepRuns` too; fixed by documenting it on `stopStepRunsWhere`.
- fix-proposal, LOW, F1: `_lym/landing/summary.md` is stale; deferred below.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- F1, the `_lym/landing/summary.md` half: the sentence "At loop start no step of the run can have a run mid-start, so no live run can match" must be dropped or narrowed to the loop's own steps.
  The file lies under `_lym`, which the fixer never touches, so the loop that owns it needs to make that edit.

## Rulings

None.
