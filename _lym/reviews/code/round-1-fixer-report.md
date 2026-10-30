# Fixer-report, round 1 (code)

## Changes

All six findings fixed; the gate (`lym gate test ./internal/landingshed ./internal/pairteardown`) passes.

- **F1** — `wake` (`internal/landingshed/prreview.go`) reads the pull request status before it gates and skips the gate for a merged pull request, so `judge` reaches the merged-with-other-head halt.
  Fixed differently from the proposal's first form only in adding a forced poll: when a head is due for the gate, `pollStatus` reads the status whatever the minute says, so a merge that appeared between two scheduled polls is seen before the gate (a self-check finding).
  The test "TheMarkIsSetAtTheStartAndAfterAGateAndClearedOnDone" asserted Done for a merge at an ungated head, which is the defect; it now uses an approval at the gated head.
  New rows: merged at an ungated head, and a head moved after the entry poll that the forced poll reads merged.
- **F2** — `closeRecordedPullRequest` stops the recorded session before `RemoveRecord`.
  Fixed with an own change on the failure path: a failed stop returns the error and keeps the record, instead of the proposed log-and-continue, since dropping the record would orphan the session the stop just failed on (code-correctness and proposal lenses).
  New test `TestPRReview_NoStopsTheRecordedSessionBeforeDroppingTheRecord`.
- **F3** — `ensureSession` stops the new strand through `Sessions.StopStrand` when `updateRecord` fails after `StartSession`, logging a failed stop.
  New session-test row.
- **F4** — the gone-worktree Warn carries a `branch` attribute and reads "a pull request from its task branch, if the run opened one, may stay open".
  `closeOpenPullRequest` takes `taskBranch`; `closePairPullRequest` derives it as the fabric config's `BranchPrefix` plus the slug, and an unreadable config logs its own Warn and leaves the branch empty so the gone Warn still goes out.
  `doc.go` and the function docs agree.
- **F5** — the `holdWhileCovered` doc now says the pause error is logged on every read.
- **F6** — new row for an open pull request with the continue flag, a clean tree and an ungated head: it keeps waiting and never closes.
- Docs aligned with F1: the `wait` doc comment and `docs/overview.md`.

Commits: `726f4d7d` (F4), `e155fc29` (F1, F2, F3, F5, F6 — the findings share files, so one commit), and a follow-up for the self-check changes.

## Self-check

- fix-consistency, LOW, `pairteardown/doc.go` stale Warn wording — fixed as proposed.
- fix-consistency, LOW, `wait` doc and `docs/overview.md` keep the old gate-then-poll order — fixed as proposed.
- fix-consistency, LOW, `ClosedForContinue` re-entry removes the record without stopping a session — disputed, premise false (see Disputed).
- fix-consistency, LOW, new gone-path config load undocumented and untested — doc fixed; the fallback also removes the failure branch; no row added, since `closePairPullRequest` needs a full `Teardown` fixture and the remaining branch only sets a log attribute.
- fix-affected-set, LOW, `doc.go` stale — duplicate of the consistency finding, fixed.
- fix-affected-set, note, `finalize_pr.go` `dropRecordedPullRequest` removes the record without stopping a session — not a finding of this fix; see Deferred.
- fix-proposal, MEDIUM, merged-at-ungated-head halt bypassed on a wake with no poll due — fixed with the forced poll and a regression row.
- fix-proposal, LOW, failed stop then record dropped — fixed with own change (return the error, keep the record).
- fix-proposal, LOW, config failure suppresses the gone Warn — fixed with the empty-branch fallback.
- fix-code-correctness, LOW, failed stop then record dropped — duplicate of the proposal finding, fixed.

## Disputed

- **premise false** — fix-consistency's `ClosedForContinue` re-entry (`prreview.go`, `openPullRequest`): `resumeSession` returns without starting a session for a `ClosedForContinue` record, and `endSession` has already stopped the session on the verdict that set the flag, so a record in that state names no live session.

## Deferred

- `internal/landingshed/finalize_pr.go` `dropRecordedPullRequest` removes the record without stopping a session, the same shape as F2, but outside this review's findings and write surface.
  Finalize runs after PR-Review's `Call` has stopped the session on its verdict, so the exposure is only an interrupted earlier call; the parent may want a board note.

## Gate repair

The gate's fabric-vocabulary scan flagged a bare `warp` token in `internal/pairteardown/teardown_test.go`, in the task-branch value the F4 test row passes.
The row now uses `task/slug`, which the scan accepts; nothing else changed.
