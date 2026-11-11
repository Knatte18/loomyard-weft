# Fixer-report, round 1 (code)

## Changes

- **F1** fixed as proposed, then corrected after the self-check.
  - `droplocal_integration_test.go`: a `TestDropLocal` table where an ignored weft file origin tracks (git overwrites it, the drop records no removal) and an untracked weft file origin does not track (kept) survive; a subtest where an untracked code branch file origin's tip tracks is never removed and git refuses the move, with the file and the code HEAD unchanged.
  - `destroy_landedwork_integration_test.go`: `TestOwnedUntrackedTrackedAt_RefusesATargetTheIndexTracksOrTheCommitDoesNot` pins the owned case, the index-tracked refusal and the not-tracked-at-commit refusal.
  - Fixed differently: the review asked for the ignored file to "survive the drop", but git's `reset --keep` overwrites an ignored file, so the row asserts origin's content and no `KindPathRemoved`.
  - Fixed differently: the review asked for a code-branch file that "survives"; the first row (a file outside the touched paths) passed with the code-side guard removed, so the subtest makes origin track the file and asserts the refusal.
- **F2** fixed with my own change.
  - Proposed: a seam test over a hub with a briefly held push lock, or options exposed through `export_test.go`.
  - Made: `reconcileSeam(reconcile, prime, seams)` in `deps.go` builds the `Reconcile` seam, and `hubDepsAs` uses it with the engine's `ReconcileOrigin`.
    `TestReconcileSeamWaitsOnAPushLock` passes a stand-in reconcile and asserts it receives `PushLockWait` equal to `reconcilePushLockWait`.
    Dropping the option from the seam fails the test.
  - Why not the proposed hub test: a real held-lock wait needs the adopt integration hub and a pushed records branch; the stand-in pins the one value that regressed.
- **F3** fixed as proposed: `doc.go` Step done now says either re-run restarts the record at `reconcile` with its steps cleared.
- **F4** fixed as proposed: `docs/overview.md` adopt entry gains one sentence each for the pause exemption, the push-lock wait and the done-stop restart.
- **F5** fixed as proposed: a `TestReconcileOrigin_Moves` subtest holds the write lock with `PushLockWait` of 3 seconds and asserts `BranchBusy` on the weft write lock, zero sleeps and no move.
- **F6** fixed with my own change.
  - Proposed: the still-holding row asserts the `lym adopt --discard` way forward.
  - Made: the row `HeldBackRecordWhoseStopStillHoldsStopsAgainWithTheSameWayForward` asserts the same error, held-back set and way forward as the first run, after a second reconcile.
  - Why: the held-back stop's text names the keep-local and drop-local re-runs and carries no `--discard`; the only `--discard` strings in `internal/adoptcli` belong to the unfinished-record refusals (`worker.go:145`, `worker.go:209`, `follow.go:84`).
  - `HeldBackRecordRejudgedKeepsItsPausedSlugs` asserts the recorded paused slugs survive the clear and are not paused again.

Commits: one per finding, then two follow-ups from the self-check (`F2`; `F1, F5`).
Gate repair: the comment-format gate flagged two comments of this round, the wrapped header of `destroy_landedwork_integration_test.go` and an over-long `TestDropLocal` doc line in `droplocal_integration_test.go`; both now break at sentence boundaries, one sentence per line, with the wording unchanged.
Gate: `lym gate test` tier 0 and tier 1 over `./internal/adoptcli` and `./internal/fabricengine`, no failures.

## Self-check

- fix-code-correctness, MEDIUM, F2 test passes with the defect in place: fixed with my own change (`reconcileSeam`, above).
- fix-code-correctness, MEDIUM, `CodeBranchFile` row passes without the code-side guard: fixed, replaced by the subtest where origin tracks the file.
- fix-code-correctness, LOW, ignored-file row name says "not removed": fixed, renamed `IgnoredFileOnAPathOriginTracksIsOverwrittenByGit` with a comment, and the subtest title retitled.
- fix-consistency, LOW, `destroy_landedwork_integration_test.go` header omits the third kind: fixed.
- fix-consistency, LOW, `TestDropLocal` doc comment omits the new subtests: fixed.
- fix-consistency, LOW, `deps_test.go` header omits the reconcile bound: fixed.
- fix-proposal, MEDIUM, F2 pins a helper nobody is forced to call: merged with the code-correctness F2 finding, fixed.
- fix-proposal, LOW, ignored-file row name misleading: merged with the code-correctness LOW, fixed.
- fix-proposal, LOW, F5 does not assert the lock named: fixed, asserts `BusyLock == "weft write"`.
- fix-proposal, LOW, F6 lacks the `--discard` assertion: disputed, premise false (see Disputed).
- fix-affected-set: no findings.

## Disputed

- **Fix-proposal LOW on F6 (a `lym adopt --discard` assertion in the still-holds row)**, ground `premise false`, evidence: `grep "adopt --discard" internal/adoptcli/*.go` outside tests finds it only in `worker.go:145`, `worker.go:209`, `follow.go:84` and `doc.go`, the unfinished-record refusals; the held-back stop's way forward is the keep-local and drop-local re-run, which the row compares against the first run's.

## Deferred

None.

## Rulings

None.
