# Fixer-report, round 1

## Changes

- F1 (BLOCKING), fixed as proposed: `internal/landingshed/fixerrun.go` derives the fixed end from `session.decided` holding a passed result.
  A run with no passed result and no `session.end` ends timeout when `Chair.Outcome` is `OutcomeTimeout`, and no-report otherwise.
  The `reached` field is removed.
  `internal/landingshed/fixerrun_test.go` gains two `TestFixerRun_Outcomes` rows, where turn 1 fails the gate and turn 2 never reaches the closure, ending timeout and no-report, backed by a `hangAfter`/`hangOutcome` option on `fakeFixerSeats`.
- F2 (MEDIUM), fixed as proposed: `internal/loomcli/doc.go` loses the PR-Gate alignment section, the Merge-Bouncer `merge` subject, the merge-subject sentence and the closing PR-Gate reject and `reject-review.md` lines.
  The bound sentence now names the landing gate at Merge-In and Finalize, and the line-140 mention drops PR-Gate.
  The same rewording is made at `internal/loomcli/subjects.go:139` and `internal/loomcli/decision.go:23`.
- F3 (LOW), fixed as proposed: `internal/loomcli/validate.go` help text and comments no longer name PR-Rework's gate and state the first_card check as `ValidateReworkPlan` runs it.
  `internal/battenshed/doc.go` and `internal/battenshed/notice.go` use `awaiting at Plan-Bouncer` as the example, and `internal/loomshed/doc.go` drops the PR-Gate rework clause.
  Fork finding fixed too: the PR-Rework rationale line in the `internal/loomcli/wiring.go` webster-record comment is removed.
- F4 (LOW), fixed as proposed: `internal/testkit/envkit/envkit.go` loses the `Merge-Review` segment and the `publish` gate list.
  Tests of `testkit`, `loomcli`, `battenshed`, `loomshed`, `landingshed` and every `envkit` importer pass.

Commits: one per finding, plus one for the `wiring.go` comment.

## Disputed

None.

## Deferred

- Remaining `Publish` mentions (`internal/loomcli/doc.go` lines 66, 123 and 175, `cli.go`, `roundgate.go`, `wiring.go`, `internal/loomshed/doc.go`, `verifygate.go`): the review's finding names none of them, and live code still carries Publish names (`internal/shedrecipe/recipe.go`, `verifytree.FailureKindPublish`), so whether each is stale is a wider sweep that belongs in its own task.
- The tests `internal/battenshed/classify_test.go` and `notice_test.go` use `PR-Gate` as a row name: renaming it broke five tests, because they pin the classifier's PR-Gate wake and reroute behavior, so those stay as they are.
  Evidence: the run after the rename failed `TestClassifyRecords` and `TestRenderNotice`; the rename was reverted.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `battenshed` tests still use the `awaiting at PR-Gate` example: not changed, see Deferred.
- fix-consistency, LOW, `reportengine` mentions PR-Gate's alignment agent: not changed, it reads old runs and is outside the change set.
- fix-affected-set, LOW, `wiring.go:717` PR-Rework comment: fixed.
- fix-affected-set, LOW, remaining Publish mentions: deferred, see Deferred.
- fix-proposal, LOW, the same Publish residue in `doc.go`: deferred, see Deferred.
- fix-proposal, LOW, F4 importers not tested: fixed by running their tests, all green.
