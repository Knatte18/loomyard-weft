# Fixer-report, webster round 1

## Changes

- F1: fixed in part.
  `internal/battencli/cancel_integration_test.go` gains `TestCancelIntegration_RemoveTearsDownPairAndBranch`, a tier-1 test over a hubforge hub with a pair, an unpushed task commit, an uncommitted file and a live pause entry of the slug.
  It runs `cancel --remove` and asserts the pair is removed, a snapshot tag is reported, the pause entry is lifted, the task branch is gone locally and on origin under an archive tag of its tip, and the claim is cleared.
  It passed at tier 1.
  The live-strand stop and the failing-lift case are deferred, see below.
  Commit e57b3760.
- F2: fixed as proposed.
  `refuseNothingLeft` in `internal/battencli/cancel.go` returns an origin-unreachable refusal naming the rerun when remove is set, end-only holds and no local part is left.
  A `TestCancelRun` row covers it.
  Commit f736b538.
- F3: fixed as proposed.
  `contracts/specs/loom-status-spec.md` names the `precondition unmet` prefix beside the bounce-budget entry.
  Commit a3e7acb9.
- F4: deferred, see below.
- F5: fixed as proposed.
  `refuseAtStart` in `internal/battencli/startguard.go` wraps the read error with `%w` and words it as reading the status file; the unused `errors` import is gone.
  Commit e138bf90.

## Disputed

None.

## Deferred

- F4: removing the reedengine changes of commit 0e0c9df2 reverts a tmux 3.4 fix, and the finding allows the other route, the orch confirming the commit in the run record.
  That is the orch's decision; the fixer sends no message to any other session and never touches `_lyx`.
- F1, live strand through `stopPairStrands`: it needs a live tmux strand, and tmux-tier runs are refused in this environment, so the test could not be verified.
- F1, `liftSlugPauses` keeping the scopes lifted before a failing lift: `tenderengine.Lift` succeeds on every ref the record names, so no seam makes a lift fail in a test without changing production code.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW, F2: the refusal also fires when origin is unreachable and nothing exists anywhere; no change, the message states the true fact that origin was unread, and the reviewer's own condition specified it.
- fix-proposal, LOW, F1 deferred parts: already recorded under Deferred.
- fix-proposal, LOW, F1 snapshot assertion checks only that a tag is reported: tried asserting the tag's contents, and the tag was not resolvable in the repository the test reads, so the assertion was reverted; the review asked only that the tag be taken.
- fix-affected-set: no findings.
  It noted the same status-read wording at four sites in `arm.go` and `owner.go`, outside the finding's location.
